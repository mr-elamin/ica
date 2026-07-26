# Istio Ambient Mode Lab Findings

This laboratory explores the "In-Pod Redirection" and "L7 Waypoint" mechanics of Istio Ambient mode within the `ambient-lab` namespace.

## 1. Lab Setup
- **Namespace:** `ambient-lab`
- **Label:** `istio.io/dataplane-mode=ambient`
- **Workloads:** 
    - `sleep-client`: A curl-based client pod.
    - `web-server`: An Nginx-based server pod.
- **Waypoint:** Deployed at the namespace level to handle L7 traffic.

## 2. Redirection Mechanics (The CNI & Ztunnel)
When a pod starts in an ambient-enabled namespace, the `istio-cni` node agent performs "In-Pod Redirection".

### Found: IPTables Rules
By inspecting the `istio-cni-node` logs, we identified the exact rules injected into the pod's network namespace:

| Table | Chain | Action | Purpose |
| :--- | :--- | :--- | :--- |
| **mangle** | `PREROUTING` | `MARK 0x539` | Marks packets coming from Ztunnel to prevent loops. |
| **nat** | `PREROUTING` | `REDIRECT --to-ports 15006` | Intercepts all incoming TCP and sends to Ztunnel inbound listener. |
| **nat** | `OUTPUT` | `REDIRECT --to-ports 15053` | Intercepts DNS queries. |
| **nat** | `ISTIO_OUTPUT` | `ACCEPT` | Allows traffic from Ztunnel specifically (via mark matching). |

### Found: Ztunnel Workload State
The Ztunnel pod on the node maintains a local database of all workloads in the mesh.
- **Command:** `istioctl ztunnel-config workload <ztunnel-pod>.istio-system`
- **Finding:** Both `sleep-client` and `web-server` are registered with the protocol `HBONE`.

## 3. L4 vs L7 Traffic Flow

### Simple L4 Flow (Ztunnel to Ztunnel)
- **Path:** `sleep-client` -> `ztunnel (source)` -> `ztunnel (target)` -> `web-server`.
- **Finding:** Ztunnel automatically provides mTLS and L4 Authorization without any Sidecars.

### L7 Flow (Waypoint involved)
By running `istioctl waypoint apply`, a waypoint (Envoy instance) was added.
- **Path:** `sleep-client` -> `ztunnel (source)` -> **`waypoint (ambient-lab)`** -> `ztunnel (target)` -> `web-server`.
- **Finding:** The Waypoint is required for any HTTP-level features (headers, paths, retries).

## 4. Useful Commands Performed

### Inspecting CNI Logs (Redirection Setup)
```bash
kubectl logs -n istio-system -l k8s-app=istio-cni-node --tail=1000 | grep "ambient-lab"
```

### Checking Ztunnel Workload Config
```bash
# Get local node ztunnel
ZTUNNEL=$(kubectl get pods -n istio-system -l app=ztunnel --field-selector spec.nodeName=$(kubectl get pod -n ambient-lab -l app=web-server -o jsonpath='{.items[0].spec.nodeName}') -o name)

# View config
istioctl ztunnel-config workload $ZTUNNEL
```

### Verifying Waypoint Status
```bash
kubectl get gtw,pods -n ambient-lab
```

### Testing Connectivity (The "Eye" Test)
```bash
kubectl exec -n ambient-lab deploy/sleep-client -- curl -v web-server
```

## 5. Summary of Roles
1. **Istio-CNI:** The "Installer". Injects iptables into your pods so they can't talk to the network without going through Ztunnel.
2. **Ztunnel:** The "Secure Tunnel". Handles mTLS, L4 policies, and zero-trust security. It is shared per node.
3. **Waypoint:** The "Policy Officer". Handles L7 (HTTP) logic. It lives as a deployment in your namespace.
