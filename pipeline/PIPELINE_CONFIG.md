# ⚡ Data Pipeline Configuration & Orchestration Guide

## Pipeline Identifier
* **Name:** `PL_Lynch_Monthly_Screen`
* **Target Workspace:** `ws_lynch_value`
* **Default Lakehouse:** `lh_lynch_value`
* **Execution Environment:** `env_lynch`

---

## Directed Acyclic Graph (DAG) Layout

```mermaid
graph LR
    A[Act_01_Universe] --> B[Act_02_Yahoo_Bronze]
    B --> C[Act_03_Silver_Features]
    C --> D[Act_04a_Slow_Growers]
    C --> E[Act_04b_Stalwarts]
    D --> F[Act_05_Paper_Portfolio]
    E --> F