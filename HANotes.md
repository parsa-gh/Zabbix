## Solid reasons to avoid (or be very cautious with) two-node clustering

### 1) **Split-brain risk (tie = disaster)**
With only **two nodes**, if there’s a **network partition** (they can’t see each other), you can get:

- Node A thinks Node B is down → promotes itself to **primary**
- Node B thinks Node A is down → also promotes itself to **primary**

✅ Result: **split-brain**, where **both nodes believe they’re the leader**, potentially causing **data divergence**.

---

### 2) **No quorum → failover becomes unpredictable**
Quorum-based systems need a **majority** to decide who should lead.

With **two nodes**, a majority is hard to achieve during failures, so systems may behave differently depending on implementation:

- both might refuse to fail over (system “freezes”)
- or both might attempt takeover
- or leadership might **flap**

✅ This makes failover behavior less deterministic than with **3+ nodes**.

---

### 3) **Maintenance is painful**
With **two nodes**:

- taking one node down for updates/reboots often reduces redundancy immediately
- failover may trigger unexpectedly
- failback later can be riskier (depending on how state recovery is handled)

✅ With 3+ nodes, you can usually do **rolling maintenance** more safely.

---

### 4) **Higher chance of “flapping”**
If the network is unstable, two-node clusters may repeatedly:

- lose connectivity
- re-elect leaders
- switch roles

✅ This creates instability, extra failover events, and application disruptions.

---

### 5) **Less tolerance to “non-node” failures**
Many 2-node designs assume the only failure is “a node goes down”.

But real failures include:

- network partitions
- storage latency spikes
- DNS / load balancer issues
- misconfigured fencing (“who is allowed to become leader?”)

✅ With just two nodes, these issues more easily push the system into **ambiguous states**.

---

## When 2-node clustering can still work (but needs safeguards)
If you must use **2 nodes**, you typically need a **tie-breaker / quorum witness**, such as:

- a **third vote** (quorum device / witness / voting node)
- an **odd number** of participants (e.g., **3 nodes**)
- strong **fencing** / STONITH so only one node can become primary safely

🚫 Without that, the architecture is inherently fragile.

---

### Quick recommendation
- **Best practice:** use **3 nodes** (or **5/7** for larger systems).
- **If 2 nodes:** require a **witness/tie-breaker** + **fencing** strategy.

---

## How failover works

- **Status reporting:** All nodes report their status every **5 seconds**.

- **When you shut down a node:**
  - The node enters a **shutdown** state.
  - Within **5 seconds**, another node will take over.

- **When a node fails unexpectedly:**
  - The workflow accounts for a **failover delay**.
  - By default, the failover delay is **1 minute**.

- **Failover delay behavior:**
  - The standby node waits for **1 minute** to see whether the failed active node comes back and updates its status.
  - If after **1 minute** the active node is still **not visible**, the standby node will **take over**.
