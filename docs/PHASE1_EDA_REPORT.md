# Social Engine Data Intake Restoration & EDA Report
**Event:** Data Vortex — Aaruush '26 | Round 1 (Phase 1)  
**Author:** Kislay Jha  

---

## 1. Executive Summary
The Social Engine suffered a catastrophic intake breakdown resulting in record duplication, signed integer inversion, unparsed multi-dialect timestamps, and null categorical fields. Following the retrieval of Dataset 1 from surviving node_07, an end-to-end cleaning and exploratory analysis pipeline was deployed. The data was restored to 12,000 verified relational rows with zero integrity violations.

---

## 2. Ingestion Integrity Audit & Cleaning Methodology

| Anomaly Class | Raw State Defect | Resolution Methodology | Justification |
| :--- | :--- | :--- | :--- |
| **Duplicate Records** | 360 redundant rows sharing primary key `post_id`. | Deduplication preserving unique `post_id` instances. | Prevents artificial metric inflation in downstream analytical engines. |
| **Bit-Flip Sign Inversion** | 525 posts with negative likes (min: -4,987; median: -2,388). | Absolute value restoration (`abs(x)`). | Negative likes mirrored positive distribution curves while shares and comments remained strictly non-negative ($\ge 0$). |
| **Missing Likes** | 1,858 null entries. | Imputed using platform-specific median values. | Preserves metric distributions without skewing aggregate means. |
| **Platform Incompleteness** | 1,846 null platform labels. | Standardized to categorical label `'Unknown'`. | Cross-platform posting patterns proved users are multi-homed (up to 5 platforms), disallowing 1:1 user inference. |
| **Timestamp Desynchronization** | Three-way collision: Unix epoch integers, ISO-8601 strings, and DD-MM-YYYY dates. | Multi-dialect regex parser normalizing records to unified UTC ISO datetime (`YYYY-MM-DD HH:MM:SS`). | Ensures cross-temporal aggregation accuracy in relational databases. |
| **Relational Integrity** | 12,000 posts referencing 1,500 users. | Foreign key verification against `Social_Engine_Users.csv`. | 0 orphan records; 100% relational integrity maintained. |

---

## 3. Exploratory Data Analysis & System Diagnostics

### A. Platform Volume & Engagement Symmetry
The platform exhibits near-uniform generation volume and symmetrical engagement:
- **Facebook:** 2,074 posts | Mean Engagement: 4,026.76
- **Instagram:** 1,989 posts | Mean Engagement: 4,040.83
- **Reddit:** 2,031 posts | Mean Engagement: 4,002.36
- **Twitter:** 2,049 posts | Mean Engagement: 3,949.26
- **YouTube:** 2,073 posts | Mean Engagement: 4,044.55
- **Unknown Platform:** 1,784 posts | Mean Engagement: 3,973.59

### B. Temporal Intake Collapse (Anomaly Discovery)
Monthly analysis uncovers a critical event in the Social Engine lifecycle:
- **Normal Operations (Jan 2024 – Apr 2025):** Mean intake sustained at ~960 posts/month.
- **Intake Failure (May 2025 – Dec 2025):** Post ingestion plunged abruptly by **95.2%**, collapsing to an average of ~48 posts/month. This marks the exact timeline of node network severance.

### C. Geographic Activity Distribution
Ingestion activity is globally distributed, led by:
1. **Los Angeles, USA:** 459 posts
2. **Munich, Germany:** 452 posts
3. **Shanghai, China:** 451 posts
4. **Barcelona, Spain:** 439 posts
5. **Houston, USA:** 422 posts

### D. Audience Engagement Decoupling
Correlation analysis reveals a total decoupling between audience size and engagement metrics:
- $r(\text{follower\_count}, \text{total\_engagement}) = 0.0013$
- $r(\text{likes}, \text{shares}) = -0.0012$
- $r(\text{likes}, \text{comments}) = 0.0095$

Engagement behavior is uncorrelated with author scale, indicating an algorithmic recommendation architecture that distributes impressions independently of static follower counts.