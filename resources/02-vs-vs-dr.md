To never mix up **VirtualService (VS)** and **DestinationRule (DR)** again, use the **"GPS vs. Car Manual"** logic.

### 1. The Mental Model: Routing vs. Policy

| Feature | **VirtualService (VS)** | **DestinationRule (DR)** |
| :--- | :--- | :--- |
| **Concept** | **The GPS (Routing)** | **The Car Manual (Policy)** |
| **Question** | *Where* should the traffic go? | *How* should we talk to the destination? |
| **Logic** | Decides the **Target**. | Defines the **Behavior**. |
| **Side** | Client-Side (Source Proxy) | Server-Side (Destination Identity) |

---

### 2. How to categorize the "Confusion Points"

#### **VirtualService (Routing Decision)**
Think of these as **Instructions** you give to a driver:
*   **Retries / Timeouts:** "If you hit a dead end, try again 3 times."
*   **Match Rules (Headers/URI):** "If the passenger has a 'Gold' badge, take them to the VIP building."
*   **Weight / Traffic Splitting:** "Send 80% of cars to the old road and 20% to the new one."
*   **Fault Injection:** "Pretend the bridge is out (503) to see how the client reacts."
*   **Rewrites / Redirects:** "If they ask for `/shop`, change it to `/v2/catalog`."

#### **DestinationRule (Connection Rules)**
Think of these as the **Rules of the Building** you are visiting:
*   **Subsets:** Defining what the "A wing" and "B wing" of the building are based on labels.
*   **LoadBalancer:** "Pick a door randomly" vs. "Always use the same door for the same person" (Consistent Hash).
*   **Circuit Breaker:** "If the building is too full (max connections), tell people to wait outside."
*   **Outlier Detection:** "If a specific door is broken (returning 5xx), lock it for 5 minutes."
*   **TLS / mTLS:** "You must show your ID (Certificate) to enter this building."

---

### 3. The "Cheat" Mnemonic: **WHERE** vs **HOW**

1.  **WHERE** is the request going? → **VirtualService**
    *   *Example:* To `v1` or `v2`? To a file? To a different URL? Should I retry if `v1` is slow?
    *   *Result:* A list of destination **hosts** or **subsets**.

2.  **HOW** do we treat that destination? → **DestinationRule**
    *   *Example:* Should we use Round Robin? Is the connection encrypted? Should we kick out failing pods?
    *   *Result:* A definition of **connection health** and **load selection**.

### 4. Why is "Subsets" in `DestinationRule`?
This is the #1 point of confusion. 
*   **DestinationRule** *defines* what a subset is: `subset v1 = version:v1`. (It "creates" the name).
*   **VirtualService** *uses* that name: `destination: subset v1`.

**Think of it like this:**
You can't tell a driver to "Go to the VIP lounge" (**VS**) until the building manager has defined which room is actually the "VIP lounge" (**DR**).

### Summary Table for the Exam
| Field | CRD | Shortcut Logic |
| :--- | :--- | :--- |
| **Retry / Timeout** | **VS** | It's a "Try again" routing instruction. |
| **Fault Injection** | **VS** | It's a "Lie to the client" routing trick. |
| **Weight (%)** | **VS** | It's a "Split the flow" decision. |
| **Circuit Breaker** | **DR** | It's a "Protection/Safety" policy. |
| **Load Balancer** | **DR** | It's a "How to pick a pod" rule. |
| **mTLS Settings** | **DR** | It's a "Security Handshake" rule. |
| **Outlier Detection** | **DR** | It's a "Health Check" policy. |