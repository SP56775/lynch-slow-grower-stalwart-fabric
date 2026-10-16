# 🏗️ System Architecture & Data Engineering Design

## Overview
This platform implements an enterprise-grade **Medallion Architecture (Bronze → Silver → Gold)** deployed on **Microsoft Fabric**. It is engineered to ingest, clean, feature-engineer, screen, and simulate an autonomous equity portfolio over the **Nifty 500** universe without manual intervention.

---

## High-Level Architecture Flow

```mermaid
flowchart TD
    subgraph Data Sources
        A1[Yahoo Finance API via yfinance]
        A2[NSE Nifty 500 Index Constituents]
    end

    subgraph Fabric Lakehouse: Bronze Layer
        B1[(bronze_universe)]
        B2[(bronze_yahoo_prices)]
        B3[(bronze_yahoo_dividends)]
        B4[(bronze_yahoo_fundamentals_long)]
        B5[(bronze_yahoo_market_snapshot)]
        B6[(bronze_yahoo_ingestion_log)]
    end

    subgraph Fabric Lakehouse: Silver Layer
        C1[(silver_lynch_features)]
    end

    subgraph Fabric Lakehouse: Gold Layer
        D1[(gold_slow_grower_screen)]
        D2[(gold_slow_grower_candidates)]
        D3[(gold_stalwart_screen)]
        D4[(gold_stalwart_candidates)]
        D5[(gold_lynch_portfolio)]
        D6[(gold_lynch_transactions)]
    end

    subgraph Serving Layer
        E1[Power BI Direct Lake Semantic Model]
        E2[Executive Dashboard]
    end

    A2 -->|NB01: Deterministic Filter| B1
    A1 -->|NB02: Ingestion Engine| B2
    A1 -->|NB02: Ingestion Engine| B3
    A1 -->|NB02: Ingestion Engine| B4
    A1 -->|NB02: Ingestion Engine| B5
    A1 -->|NB02: Ingestion Engine| B6

    B1 & B2 & B3 & B4 & B5 -->|NB03: PySpark Feature Engineering| C1

    C1 -->|NB04a: Dividend Screening| D1
    C1 -->|NB04a: Dividend Screening| D2
    C1 -->|NB04b: Valuation & Compounding| D3
    C1 -->|NB04b: Valuation & Compounding| D4

    D2 & D4 -->|NB05: ₹10L Allocation Engine| D5
    D2 & D4 -->|NB05: ₹10L Allocation Engine| D6

    D2 & D4 & D5 & D6 & C1 -->|Direct Lake on OneLake| E1
    E1 --> E2