# ICA Exam: CRD Capability Map

Use this to memorize **WHERE** a feature belongs. In the exam, they will try to trick you by asking if a "Retry" is configured in a `DestinationRule` (it's not, it's in `VirtualService`).

---

## 1. VirtualService (The "Router")
**Mental Model:** The "Flow" controller. Matches traffic and decides its path.

| Feature | Key Path |
| :--- | :--- |
| **Header Match** | `spec.http.match.headers` |
| **URI Rewrite** | `spec.http.rewrite.uri` |
| **Fault Injection** | `spec.http.fault.delay` / `abort` |
| **Mirroring** | `spec.http.mirror` |
| **Retries** | `spec.http.retries` |
| **Timeouts** | `spec.http.timeout` |
| **CORS** | `spec.http.corsPolicy` |
| **Traffic Shifting** | `spec.http.route.weight` |

---

## 2. DestinationRule (The "Policy")
**Mental Model:** Post-routing rules. What happens after the destination is chosen?

| Feature | Key Path |
| :--- | :--- |
| **Subsets (Versions)** | `spec.subsets.name` + `labels` |
| **Load Balancing** | `spec.trafficPolicy.loadBalancer` (LEAST_CONN, RANDOM, etc.) |
| **Circuit Breaking** | `spec.trafficPolicy.outlierDetection` |
| **Connection Pooling** | `spec.trafficPolicy.connectionPool` |
| **TLS (Client Side)** | `spec.trafficPolicy.tls.mode` (SIMPLE, MUTUAL, ISTIO_MUTUAL) |

---

## 3. Gateway (The "Entry")
**Mental Model:** The edge of the mesh.

| Feature | Key Path |
| :--- | :--- |
| **TLS Termination** | `spec.servers.tls.mode: SIMPLE` |
| **SNI Passthrough** | `spec.servers.tls.mode: PASSTHROUGH` |
| **mTLS at Edge** | `spec.servers.tls.mode: MUTUAL` |
| **Selector** | `spec.selector: istio: ingressgateway` |

---

## 4. Exam Scenarios to Memorize

### Scenario A: Canary Release
1.  **DestinationRule:** Define subsets `v1` and `v2` based on pod labels.
2.  **VirtualService:** Use `route.weight` to split traffic (e.g., 90/10).

### Scenario B: Circuit Breaker
1.  **DestinationRule:** Configure `outlierDetection` (consecutiveErrors, interval, baseEjectionTime).

### Scenario C: External Traffic
1.  **ServiceEntry:** Define the external host (e.g., `google.com`).
2.  **VirtualService:** (Optional) Add a timeout for that external host.

### Scenario D: Security Lockdown
1.  **PeerAuthentication:** Set to `STRICT` (forces mTLS).
2.  **AuthorizationPolicy:** Set an `ALLOW` rule for only specific service accounts.
