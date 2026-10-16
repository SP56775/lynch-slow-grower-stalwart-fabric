
---

# 3. `docs/SETUP_GUIDE.md`

```markdown
# 🛠️ Microsoft Fabric Deployment & Setup Guide

This guide details how to reproduce this quantitative research engine within a Microsoft Fabric workspace from scratch.

---

## Prerequisites
* An active Microsoft Fabric capacity (F2+ SKU or 60-Day Trial).
* Workspace created (e.g., `ws_lynch_value`).
* Basic familiarity with Notebooks and Lakehouse artifacts in Fabric.

---

## Step 1: Create Lakehouse & Fabric Environment

1. Open your workspace and click **+ New item** → **Lakehouse**. Name it `lh_lynch_value`.
2. Create a custom Environment to eliminate inline `%pip` dependency issues:
   * Click **+ New item** → **Environment**. Name it `env_lynch`.
   * Under **Public Libraries**, add:
     * `yfinance`
     * `pandas`
     * `numpy`
   * Click **Save** and then **Publish**. Wait for publication to complete.

---

## Step 2: Establish Lakehouse Folder Hierarchy

Navigate to **Files** inside `lh_lynch_value` and create the following directory structure:

```text
Files/
├── config/
└── reference/