# 🌐 VERIQTA Kubernetes Production Outages Master Handbook (Volume 2)
> **Networking & Service Failures: CoreDNS, Ingress, NetworkPolicies, CNI, Load Balancers, Service Mesh & Multi-Cluster Triage.**  
> *Engineered for Site Reliability Engineers (SREs), Network Architects, DevOps Engineers, and Kubernetes Incident Responders.*

---

## 📑 Table of Contents
1. [Executive Mental Model: The End-to-End Traffic Path](#1-executive-mental-model-the-end-to-end-traffic-path)
2. [How Kubernetes Networking Actually Works (CNI, Kube-Proxy & Services)](#2-how-kubernetes-networking-actually-works-cni-kube-proxy--services)
3. [Service Has No Endpoints: Diagnosis & Selector Discrepancies](#3-service-has-no-endpoints-diagnosis--selector-discrepancies)
4. [CoreDNS Failures & DNS Crash Loops in Production](#4-coredns-failures--dns-crash-loops-in-production)
5. [Pods Cannot Resolve DNS: `ndots:5`, Search Domains & Port 53 Traps](#5-pods-cannot-resolve-dns-ndots5-search-domains--port-53-traps)
6. [Ingress Not Routing Traffic: IngressClass, Paths & Controller Triage](#6-ingress-not-routing-traffic-ingressclass-paths--controller-triage)
7. [TLS Certificate Failures: Cert-Manager, Expired Keys & Browser Errors](#7-tls-certificate-failures-cert-manager-expired-keys--browser-errors)
8. [NetworkPolicy Blocks Production: Silent Drops & The DNS Egress Blocker](#8-networkpolicy-blocks-production-silent-drops--the-dns-egress-blocker)
9. [CNI Plugin Failures: IPAM Pool Exhaustion, VXLAN & MTU Disasters](#9-cni-plugin-failures-ipam-pool-exhaustion-vxlan--mtu-disasters)
10. [Pod-to-Pod Communication Failures & Cloud Security Group Locks](#10-pod-to-pod-communication-failures--cloud-security-group-locks)
11. [External Traffic Never Reaches the Cluster: Edge & Ingress Triaging](#11-external-traffic-never-reaches-the-cluster-edge--ingress-triaging)
12. [LoadBalancer Health Check Failures: The "All Targets Unhealthy" Blackhole](#12-loadbalancer-health-check-failures-the-all-targets-unhealthy-blackhole)
13. [Session Affinity & Sticky Session Breakdown](#13-session-affinity--sticky-session-breakdown)
14. [API Gateway & Ingress Error Codes (502, 503, 504 & Connection Resets)](#14-api-gateway--ingress-error-codes-502-503-504--connection-resets)
15. [Service Mesh Failures (Istio & Linkerd): Sidecars, mTLS & Subsets](#15-service-mesh-failures-istio--linkerd-sidecars-mtls--subsets)
16. [Cross-Namespace Communication Failures & FQDN Standards](#16-cross-namespace-communication-failures--fqdn-standards)
17. [Multi-Cluster Traffic Routing, Latency & Failover Failures](#17-multi-cluster-traffic-routing-latency--failover-failures)
18. [Enterprise Networking 9-Layer Investigation Workflow](#18-enterprise-networking-9-layer-investigation-workflow)
19. [High-Severity Production Networking Incident Playbook](#19-high-severity-production-networking-incident-playbook)
20. [Volume 2 Production Readiness Checklist & Scorecard](#20-volume-2-production-readiness-checklist--scorecard)

---

## 1. Executive Mental Model: The End-to-End Traffic Path

```
                           THE COMPLETE KUBERNETES TRAFFIC PATH
  ┌────────┐      ┌────────┐      ┌────────┐      ┌─────────┐      ┌───────────┐      ┌────────┐
  │ Client │ ───> │ DNS    │ ───> │ Cloud  │ ───> │ Ingress │ ───> │ Service   │ ───> │ Target │
  │ (User) │      │ Lookup │      │ LB/WAF │      │ Router  │      │ Endpoints │      │ Pod IP │
  └────────┘      └────────┘      └────────┘      └─────────┘      └───────────┘      └────────┘
                                                         │              │                  │
                                                         ▼              ▼                  ▼
                                                    [TLS Term]   [kube-proxy IPVS]   [Container]
```

> [!CAUTION]
> **Production Principle:** *"Networking failures almost always masquerade as application failures."* Senior SREs always validate the end-to-end network path step-by-step from client to container before inspecting application code.

---

## 2. How Kubernetes Networking Actually Works (CNI, Kube-Proxy & Services)

```
                            KUBERNETES NETWORKING FOUNDATIONS
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Flat CNI Model  : Every Pod gets a real, routable IP. No NAT between Pods.         │
 │ 2. Service VIPs    : Stable virtual IPs programmed by kube-proxy (iptables / IPVS).    │
 │ 3. CoreDNS Engine  : Resolves `<service>.<namespace>.svc.cluster.local` internally.   │
 │ 4. Ingress Gateway : Layer 7 reverse proxy routing external HTTP/HTTPS host/path rules.│
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3 Core Service Types
1. **`ClusterIP` (Default):** Virtual stable internal IP (`10.96.x.x`); only accessible from within the cluster.
2. **`NodePort`:** Exposes the service on each node's physical IP at a static port (`30000–32767`).
3. **`LoadBalancer`:** Automatically provisions a cloud provider Load Balancer (AWS NLB/ALB, GCP LB) routing to NodePorts/Pods.

---

## 3. Service Has No Endpoints: Diagnosis & Selector Discrepancies

```
                          SERVICE WITHOUT ENDPOINTS (OUTAGE)
   Service Manifest                    Label Query                    Backing Pods
┌──────────────────────┐         ┌──────────────────────┐         ┌──────────────────────┐
│ selector:            │         │ Looking for:         │         │ labels:              │
│   app: payment-api   │ ──────> │ app=payment-api      │ ──X──>  │   app: payment-svc   │
└──────────────────────┘         └──────────────────────┘         │ (Mismatch -> 0 EPs)  │
                                                                  └──────────────────────┘
```

### Top 6 Root Causes
1. **Label Selector Mismatch:** Service selector has `app: web` while Pods have `app: webapp` (Character-for-character mismatch).
2. **Failing Readiness Probes:** Pods exist and are running, but their readiness probes are failing (`READY: 0/1`). Kubelet drops unready pods from endpoints.
3. **Namespace Discrepancy:** Service deployed in `production` while target Pods are running in `staging`.
4. **Failed Rollout:** Deployment update failed; new pods never became ready while old pods were terminated.
5. **EndpointSlice Controller Lag/Corruption:** Stale `EndpointSlice` objects not updating due to API server overload.
6. **Port / TargetPort Mismatch:** Service `port: 80` targeting wrong `targetPort: 8080` where app listens on `3000`.

### Fast Triage Commands
```bash
# 1. Check Service VIP and ports
kubectl get svc -n production

# 2. Check backing endpoints (If <none>, you have an active outage!)
kubectl get endpoints -n production

# 3. Check modern EndpointSlice objects
kubectl get endpointslices -n production

# 4. Describe Service to inspect selector and active endpoints
kubectl describe svc <service-name> -n production
```

---

## 4. CoreDNS Failures & DNS Crash Loops in Production

```
                               COREDNS ARCHITECTURE
  [ Application Pod ] ──(UDP/TCP 53)──> [ CoreDNS ClusterIP: 10.96.0.10 ]
                                                   │
                         ┌─────────────────────────┴─────────────────────────┐
                         ▼                                                   ▼
             [ Kubernetes API Server ]                            [ Upstream Resolvers ]
             (Internal Cluster Discovery)                         (8.8.8.8 / VPC 169.254.169.253)
```

### Common Failure Modes
1. **CoreDNS CrashLoopBackOff:** CoreDNS pods running out of memory due to high query volume (Fix: increase CPU/RAM requests/limits or deploy `NodeLocal DNSCache`).
2. **DNS Forwarding Loop:** Upstream `/etc/resolv.conf` on worker nodes points back to CoreDNS, creating an infinite DNS query storm.
3. **NetworkPolicy Blocking DNS:** A default-deny egress NetworkPolicy applied to application namespaces blocking port 53 to `kube-system`.
4. **Corefile Syntax Errors:** Misconfigured upstream forwarders or plugin syntax errors breaking DNS parsing.

---

## 5. Pods Cannot Resolve DNS: `ndots:5`, Search Domains & Port 53 Traps

```
                           INSIDE THE POD: /etc/resolv.conf
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ nameserver 10.96.0.10                                                                  │
 │ search default.svc.cluster.local svc.cluster.local cluster.local                       │
 │ options ndots:5                                                                        │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

### The `ndots:5` Latency Trap
* **How it works:** If a query contains fewer than 5 dots (e.g., `api.stripe.com` has 2 dots), Kubernetes appends **every local search domain first** before attempting external resolution:
  1. `api.stripe.com.default.svc.cluster.local.` $\rightarrow$ NXDOMAIN (Fails)
  2. `api.stripe.com.svc.cluster.local.` $\rightarrow$ NXDOMAIN (Fails)
  3. `api.stripe.com.cluster.local.` $\rightarrow$ NXDOMAIN (Fails)
  4. `api.stripe.com.` $\rightarrow$ Resolves successfully on 4th attempt!
* **Impact:** Generates **$4\times$ DNS traffic**, causing CoreDNS CPU exhaustion and high latency.
* **Fix:** Append a trailing dot in external domains (`api.stripe.com.`) or customize `dnsConfig.options.ndots: 2` in the Pod spec.

---

## 6. Ingress Not Routing Traffic: IngressClass, Paths & Controller Triage

```
                             INGRESS ROUTING DECISION TREE
                          [ User Request: shop.example.com/orders ]
                                            │
                                            ▼
                           [ Ingress Controller (NGINX/ALB) ]
                                            │
           ┌────────────────────────────────┴────────────────────────────────┐
           ▼                                                                 ▼
   Host: shop.example.com?                                         Path: /orders matched?
   (If NO -> 404 Default Backend)                                  (If NO -> 404 Not Found)
           │                                                                 │
           └───────────────────────────────┬─────────────────────────────────┘
                                           ▼
                            [ Forward to orders-svc:8080 ]
```

### Common Ingress Failure Modes
* **Missing / Mismatched `ingressClassName`:** Ingress manifest lacks `ingressClassName: nginx`; controller ignores the resource.
* **Path Regex / Prefix Issues:** Using `pathType: Exact` instead of `pathType: Prefix` or missing `nginx.ingress.kubernetes.io/rewrite-target: /`.
* **Missing Backend Service:** Ingress points to a Service that does not exist in the same namespace.
* **Controller Unavailable:** Ingress Controller pods in `ingress-nginx` namespace are crashing or running out of memory.

---

## 7. TLS Certificate Failures: Cert-Manager, Expired Keys & Browser Errors

```
                            TLS TERMINATION IN KUBERNETES
  Client ──(HTTPS)──> [ Ingress Controller ] ──(TLS Secret Key/Cert)──> [ Service (HTTP) ] ──> [ Pod ]
```

### Common Browser Error Code Lookup
| Browser Error Code | Technical Root Cause | SRE Remediation Action |
| :--- | :--- | :--- |
| **`ERR_CERT_DATE_INVALID`** | TLS certificate has passed its expiration date. | Trigger cert-manager renewal or upload updated x509 secret. |
| **`ERR_CERT_COMMON_NAME_INVALID`** | Requested hostname does not match SANs in certificate. | Regenerate CSR with correct Subject Alternative Names (SAN). |
| **`ERR_CERT_AUTHORITY_INVALID`** | Self-signed certificate or missing intermediate CA cert. | Bundle intermediate certificate into `tls.crt` secret. |
| **`ERR_SSL_PROTOCOL_ERROR`** | HTTP traffic sent to HTTPS port or mismatched ciphers. | Verify TLS listener configuration on Ingress/LoadBalancer. |

---

## 8. NetworkPolicy Blocks Production: Silent Drops & The DNS Egress Blocker

```
                     THE DEADLY NETWORKPOLICY EGRESS TRAP
  [ Backend Pod ] ───(Default-Deny Egress Active)───X───> [ CoreDNS (10.96.0.10:53) ]
         │
         ▼
  DNS Query Blocked! ──> Pod cannot resolve Database hostname ──> 100% Application Crash!
```

### Production Rule: The Mandatory DNS Egress Block
Whenever creating an Egress NetworkPolicy, you **MUST explicitly allow UDP/TCP port 53**:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-egress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  # 1. ALLOW DNS RESOLUTION TO COREDNS
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  # 2. ALLOW DATABASE EGRESS
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

---

## 9. CNI Plugin Failures: IPAM Pool Exhaustion, VXLAN & MTU Disasters

```
                              CNI ARCHITECTURE
  Node 1 (Pod 10.244.1.5) ──> [ CNI Overlay (VXLAN/eBPF) ] ──> Node 2 (Pod 10.244.2.8)
```

### Top CNI Outage Scenarios
1. **IPAM Subnet Exhaustion:** Node runs out of IP addresses to allocate to new pods $\rightarrow$ Pods stay in `ContainerCreating`.
2. **VXLAN / Overlay MTU Mismatch:** Overlay packet header exceeds physical network MTU ($1500\text{ bytes} - 50\text{ bytes header} = 1450\text{ MTU}$). Causes silent packet fragmentation and dropped TCP connections.
3. **CNI DaemonSet Crash:** `aws-node`, `calico-node`, or `cilium` pods crash on a worker node $\rightarrow$ all local networking halts.

---

## 10. Pod-to-Pod Communication Failures & Cloud Security Group Locks

```
                     CROSS-NODE NETWORK VALIDATION PATH
  [ Pod A ] ──> [ Node 1 veth ] ──> [ AWS Security Group ] ──> [ Node 2 veth ] ──> [ Pod B ]
                                            │
                                    (Blocked Port?)
```

### 7-Step Diagnostic Checklist
```bash
# 1. Verify Pod IPs across nodes
kubectl get pods -o wide -n production

# 2. Test direct Pod IP connectivity
kubectl exec -it <pod-a> -n production -- ping <pod-b-ip>
kubectl exec -it <pod-a> -n production -- curl -I http://<pod-b-ip>:<port>

# 3. Check for blocking NetworkPolicies
kubectl get networkpolicies -A

# 4. Check CNI DaemonSet logs in kube-system
kubectl logs -n kube-system -l k8s-app=calico-node --tail=50

# 5. Check host routing tables
ip route show

# 6. Validate MTU consistency
ip link show | grep mtu
```

---

## 11. External Traffic Never Reaches the Cluster: Edge & Ingress Triaging

```
                 OUTSIDE-IN INVESTIGATION SEQUENCE
  [ 1. Public DNS ] ──> [ 2. Cloud LB / WAF ] ──> [ 3. Security Groups ] ──> [ 4. Ingress ]
```

### The "Cluster is Innocent Until Proven Guilty" Rule
When external users report outages:
1. **Test Public DNS:** `dig +short yourdomain.com` (Does it point to the Load Balancer CNAME/IP?).
2. **Check Cloud Load Balancer Status:** `aws elbv2 describe-load-balancers` (Is the LB active and provisioning?).
3. **Verify Security Groups:** Ensure port 80/443 inbound rules are open from `0.0.0.0/0`.
4. **Inspect Cloud Firewall / WAF:** Check if AWS WAF or Cloudflare is blocking user IP blocks with HTTP 403.

---

## 12. LoadBalancer Health Check Failures: The "All Targets Unhealthy" Blackhole

```
                        CLOUD LOAD BALANCER HEALTH CHECK
  Cloud LB ──(HTTP /healthz)──> Target Nodes/Pods ──(Failing Probe)──> Targets Marked Unhealthy
                                                                                 │
                                                                                 ▼
                                                                       100% Traffic Drops!
```

### Critical Health Check Parameters
* **Health Check Path:** Ensure Cloud LB checks `/healthz` or `/actuator/health` that returns HTTP 200 (not a protected route returning HTTP 401/403).
* **Port Mapping:** Verify LB health check port matches the Service `NodePort` or target container port.
* **Security Group Probes:** Cloud provider health-check CIDRs (e.g., AWS LB subnets) must be whitelisted in worker node Security Groups.

---

## 13. Session Affinity & Sticky Session Breakdown

```
                         SESSION AFFINITY FLOW
  User A (Cookie: SESSION_123) ──> Ingress/LB ──(Sticky Route)──> [ Pod 1 (Active Session) ]
```

### Why Sticky Sessions Break
1. **In-Memory State on Ephemeral Pods:** Pod restarts or scales down; local memory session is lost $\rightarrow$ User is logged out.
2. **Cookie Domain / SameSite Mismatch:** Browser rejects affinity cookie due to missing `Secure` or `SameSite=None` attributes.
3. **Cross-AZ LB Rebalancing:** Load balancer distributes requests across availability zones without maintaining cookie persistence.
4. **Architectural Solution:** Always decouple session state into a centralized distributed cache (**Redis / AWS ElastiCache**).

---

## 14. API Gateway & Ingress Error Codes (502, 503, 504 & Connection Resets)

```
                            HTTP GATEWAY ERROR MATRIX
 ┌───────────┬──────────────────────────┬────────────────────────────────────────────────────────┐
 │ Code      │ Standard Meaning         │ Kubernetes Root Cause & Investigation Target           │
 ├───────────┼──────────────────────────┼────────────────────────────────────────────────────────┤
 │ **502**   │ Bad Gateway              │ Backend pod closed TCP connection prematurely or died. │
 │ **503**   │ Service Unavailable      │ Service has **zero ready endpoints** or pods overloaded│
 │ **504**   │ Gateway Timeout          │ Backend took longer than proxy timeout (e.g., > 60s).  │
 │ **RST**   │ Connection Reset         │ Kernel rejected packet; host reachable, port closed.   │
 └───────────┴──────────────────────────┴────────────────────────────────────────────────────────┘
```

---

## 15. Service Mesh Failures (Istio & Linkerd): Sidecars, mTLS & Subsets

```
                        SERVICE MESH MTLS DATA PLANE
  [ Pod A ] ──> [ Envoy Proxy A ] ──(mTLS Encrypted)──> [ Envoy Proxy B ] ──> [ Pod B ]
```

### Common Mesh Failure Modes
1. **Sidecar Injection Missing:** Pod deployed without `istio-proxy` container; traffic bypasses mesh policies.
2. **mTLS Handshake Failures:** Expired certificates, clock skew between nodes, or trust domain mismatch (`503 UF / 495 SSL`).
3. **AuthorizationPolicy 403 Forbidden:** Strict zero-trust mesh policy blocking unauthenticated service accounts.
4. **DestinationRule Subset Mismatch:** Traffic routed to subset `version: v2` where 0 pods match the label.

```bash
# Debug Istio mesh proxy status
istioctl proxy-status
istioctl proxy-config endpoints <pod-name>
istioctl analyze
```

---

## 16. Cross-Namespace Communication Failures & FQDN Standards

```
                      CROSS-NAMESPACE DNS RESOLUTION
  Namespace: frontend                                  Namespace: backend
┌──────────────────────┐                             ┌──────────────────────┐
│ [ Web App Pod ]      │ ──(Resolves FQDN)─────────> │ [ Database Service ] │
└──────────────────────┘                             └──────────────────────┘
```

### The 4 DNS Name Formats
1. `db-service.backend.svc.cluster.local` $\rightarrow$ **Fully Qualified Domain Name (FQDN) [Best Practice]**
2. `db-service.backend.svc` $\rightarrow$ Short FQDN (Resolves reliably)
3. `db-service.backend` $\rightarrow$ Namespace-qualified (Works in cluster)
4. `db-service` $\rightarrow$ **Fails across namespaces!** (Only works within identical namespace)

---

## 17. Multi-Cluster Traffic Routing, Latency & Failover Failures

```
                     MULTI-CLUSTER ACTIVE-ACTIVE ARCHITECTURE
                               [ Global Anycast LB ]
                                         │
                     ┌───────────────────┴───────────────────┐
                     ▼                                       ▼
         [ Cluster 1 (Primary - US-East) ]       [ Cluster 2 (Secondary - US-West) ]
```

### Multi-Cluster Failure Modes
* **Stale Cross-Cluster Service Discovery:** Consul or AWS Cloud Map sync fails; primary cluster sends traffic to terminated IP addresses in secondary cluster.
* **Flapping Failover Alarms:** Health check thresholds too aggressive; minor internet latency causes continuous failover flapping.
* **Database Replication Lag:** Failover triggers write traffic to read-replica cluster before asynchronous replication catches up.

---

## 18. Enterprise Networking 9-Layer Investigation Workflow

```
                        9-LAYER NETWORK TRIAGE PATH
  1. User Impact ──> 2. DNS ──> 3. Cloud LB ──> 4. Ingress ──> 5. Service
                                                                   │
  9. Application <── 8. Pod <── 7. NetworkPolicy <── 6. Endpoints <┘
```

### Layer-by-Layer Verification Summary
| Layer | Diagnostic Target | Command / Tool | Success Condition |
| :--- | :--- | :--- | :--- |
| **1. User Impact** | Scope & error rate | Monitoring / Sentry / APM | Clear impact timeline |
| **2. DNS** | Public & internal DNS | `dig +short domain.com` | Resolves correct IP/CNAME |
| **3. Cloud LB** | Target group health | Cloud Console / AWS CLI | Targets show `Healthy` |
| **4. Ingress** | Routing rules & TLS | `kubectl describe ingress` | Host/Path rules matched |
| **5. Service** | Virtual IP & Port | `kubectl describe svc` | Port & TargetPort correct |
| **6. Endpoints** | Backing Pod IPs | `kubectl get endpoints` | Lists $\ge 1$ healthy Pod IP |
| **7. NetPol** | Firewall rules | `kubectl get netpol -A` | Port & DNS whitelisted |
| **8. Pod** | Container & Probes | `kubectl get pods -o wide` | `Status: Running`, `1/1 Ready` |
| **9. App** | Application code | `kubectl logs <pod>` | HTTP 200, no 5xx exceptions |

---

## 19. High-Severity Production Networking Incident Playbook

```
                         INCIDENT RESPONSE LIFECYCLE
  [ 01. Alert ] ──> [ 02. Verify Impact ] ──> [ 03. Freeze Changes ] ──> [ 04. Validate DNS ]
                                                                               │
  [ 07. Check Endpoints ] <── [ 06. Validate Service ] <── [ 05. Inspect LB ] <┘
        │
        └──> [ 08. Test NetPol/CNI ] ──> [ 09. Recover Traffic ] ──> [ 10. Post-Mortem ]
```

### Severity Escalation Matrix
* **SEV-1 (Critical Outage):** Core revenue/customer path down $\rightarrow$ Page on-call lead, open war room, restore traffic first, diagnose later.
* **SEV-2 (Degraded Performance):** High latency or partial region failure $\rightarrow$ Notify team, engage secondary responder, scale capacity.
* **SEV-3 (Internal Service Impact):** Non-critical internal tool offline $\rightarrow$ Standard issue tracker triage.

---

## 20. Volume 2 Production Readiness Checklist & Scorecard

| Category | Verification Item | Production Status |
| :--- | :--- | :---: |
| **Service** | Every Service has verified non-empty Endpoints (`kubectl get ep`) | [ ] |
| **DNS** | CoreDNS replicas $\ge 2$, resources scaled, `ndots` tuned | [ ] |
| **DNS** | All external domains in code append trailing dot or use FQDN | [ ] |
| **Ingress** | `ingressClassName` explicitly defined; TLS secrets verified and unexpired | [ ] |
| **NetPol** | Every egress NetworkPolicy explicitly allows UDP/TCP 53 to CoreDNS | [ ] |
| **CNI** | IPAM subnet utilization monitored; MTU consistent across all nodes | [ ] |
| **Edge LB** | Cloud Load Balancer health checks point to dedicated lightweight `/healthz` | [ ] |
| **State** | Session affinity uses external Redis store instead of in-memory state | [ ] |
| **Mesh** | Service mesh mTLS certificate expiration alerts configured | [ ] |
| **Multi-AZ**| Endpoints and Pods evenly balanced across $\ge 3$ Availability Zones | [ ] |
