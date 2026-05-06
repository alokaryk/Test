To ensure a perfect score and address the specific feedback you received, I have refined your text to be more precise regarding **HSM data handling** and the **User Distribution shift** in the bonus section. 

You can use the following version for your final submission.

---

## Scenario 1 – Medium-size Organization (2,500 users)

### 1. Total IOPS Required
*   **User IOPS (25% Concurrency):**
    *   Active Light: $500 \times 1.2 = 600$
    *   Active Medium: $94 \times 3 = 282$
    *   Active Power: $31 \times 6 = 186$
    *   **Subtotal:** 1,068 IOPS
*   **System Overhead (+30%):** $1,068 \times 1.3 = 1,388$ IOPS
*   **Headroom (1.25):** $1,388 \times 1.25 = \mathbf{1,735}$ **IOPS**

### 2. Daily Throughput
*   **Total Messages:** 51,250 msgs/day
*   **Raw Data:** $51,250 \times 150 \text{ KB} = 7.7 \text{ GB/day}$
*   **Net Throughput (35% Deletion/Crunch):** $7.7 \times 0.65 = \mathbf{5}$ **GB/day**

### 3. Total Hot Data Footprint (HSM 90 Days)
*   **Base Hot Data:** $5 \text{ GB/day} \times 90 \text{ days} = 450 \text{ GB}$
*   **Total Hot Footprint (+30% Overhead):** $450 \times 1.3 = \mathbf{585}$ **GB (~0.6 TB)**
*   *Note: While total user quotas sum to ~9 TB, the HSM policy ensures only 0.6 TB resides on Primary (Hot) storage.*

### 4. Mailstores Required
*   **IOPS-based:** $1,735 / 600 \text{ (Small Tier Cap)} = 2.9 \rightarrow 3$ nodes
*   **Storage-based:** $0.6 \text{ TB} / 4 \text{ TB} = 0.15 \rightarrow 1$ node
*   **Final Requirement (N+1):** $3 + 1 = \mathbf{4}$ **Small Mailstores**

### 5. Dominant Constraint
**IOPS (Performance).** Storage utilization is less than 15% of capacity, but 3 active nodes are required to sustain the I/O load within the safe CPU envelope.

---

## Scenario 1B – Alternative User Mix (2,500 users)

### 1. Total IOPS Required
*   **User IOPS (25% Concurrency):**
    *   Active Light (250), Medium (313), Power (63)
    *   **Subtotal:** 1,617 IOPS
*   **Total (with System + Headroom):** $\mathbf{2,628}$ **IOPS**

### 2. Daily Throughput & Hot Data
*   **Daily Throughput:** **9.5 GB/day**
*   **Hot Data (90 Days + 30% Overhead):** $(9.5 \times 90) \times 1.3 = \mathbf{1.11}$ **TB**

### 3. Mailstores Required
*   **IOPS-based:** $2,628 / 600 = 4.38 \rightarrow 5$ nodes
*   **Final Requirement (N+1):** $5 + 1 = \mathbf{6}$ **Small Mailstores**

### 4. Comparison Analysis
Scenario 1B is **35% more demanding** in IOPS and **nearly double** the storage throughput compared to Scenario 1. The primary change is the shift from a "Light" majority to a "Medium" majority, which drastically increases active I/O operations per user.

---

## Scenario 2 – Multi-domain Service Provider

### Part A: Baseline (6,000 Users)
1.  **Total IOPS:** $4,830 \text{ (Base)} \times 1.25 \times 1.2 = \mathbf{7,245}$ **IOPS**
2.  **Daily Throughput:** **83 GB/day**
3.  **Hot Data (15 Days HSM + 30%):** $(83 \times 15) \times 1.3 = \mathbf{1.62}$ **TB**
4.  **Nodes (Medium Tier):**
    *   IOPS: $7,245 / 1,100 = 6.58 \rightarrow 7$ nodes
    *   Final: $7 + 1 = \mathbf{8}$ **Medium Nodes**
5.  **Threshold Check:** CPU load per node is ~59%, which is safe but leaves very little margin for peak bursts.

### Part B & C: 12-24 Month Projections

| Timeline | Users | Total IOPS | Hot Data | Nodes (Large Tier) |
| :--- | :--- | :--- | :--- | :--- |
| **Baseline** | 6,000 | 7,245 | 1.62 TB | **5 (4+1)** |
| **12 Months** | 8,000 | 9,660 | 2.16 TB | **6 (5+1)** |
| **24 Months** | 12,000 | 14,490 | 3.24 TB | **9 (8+1)** |

*   **Scalability Discussion:** Migration to the **Large Tier** is recommended by Month 12. While "Adding Nodes" (Option A) works, 15 Medium nodes create excessive operational overhead compared to 9 Large nodes. 

### Bonus: Power User Growth (5% → 8%)
*   **User Mix at 12,000 Users:** Light (6,240), Medium (4,800), Power (960).
*   **New Total IOPS:** **13,608 IOPS** (Calculated with 35% concurrency and headroom).
*   **New Hot Data:** **3.30 TB** (Insignificant growth due to 15-day HSM).
*   **Limiting Factor:** **IOPS/CPU Saturation.** 
    *   With 9 nodes (8+1), IOPS per node = 1,701. 
    *   CPU: $(1,701 / 2,000) \times 60\% + 10\% = \mathbf{61\%}$.
*   **Correction:** Because the 60% CPU threshold is breached, the Power User shift forces an increase to **10 Large Nodes (9+1)** to maintain sustainable performance.

### Final Conclusion
The architecture is dominated by **IOPS and CPU constraints**. The HSM policy effectively detaches storage growth from the primary mailstore requirements, allowing the service provider to scale based on user activity levels rather than total disk volume.

---
**Tip:** Make sure the formatting (tables and bold numbers) is preserved when you upload! Good luck with the final result!
