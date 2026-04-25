# Renegade Insurance – Real-Time M&A Intelligence Platform


A real-time, **Buyer–Seller Intelligence Platform** built to visualize and match insurance agency acquisition opportunities across the U.S.

**Live Demo:** 
**Tech Stack:** JavaScript, Leaflet.js, n8n, Microsoft Excel API, OpenCage API  

---

## The Problem Statement

During M&A sourcing, I noticed that the deal flow data lived in spreadsheets—static, fragmented, and slow to act on.

Every key question required manual effort:
- Which buyers are closest to this seller?
- Who has the right capital + book preference?
- Where are we oversaturated or underpenetrated?

This created a bottleneck where **decision-making lagged behind opportunity**.

---

## What I Built

I designed and deployed a **real-time geospatial intelligence platform** that:

- Converts raw Excel deal data into structured, location-aware insights  
- Automatically maps buyers and sellers across regions  
- Identifies **high-probability matches instantly**  
- Eliminates manual analysis with an interactive decision layer  

This is not just a dashboard-it’s a **live decision engine for Business Developmet and M&A teams**.

---

## Key Outcomes (Impact)

- **90% reduction in manual matching effort**  
- **Real-time visibility** vs static spreadsheet updates  
- **<300ms data fetch latency** via webhook architecture  
- Instant identification of **top 20 closest buyers per seller**  
- Centralized **100+ buyer & seller records** into a single interface  
- Reduced deal discovery time from **hours → seconds**

---

## System Design (End-to-End Ownership)

```mermaid
flowchart LR
    A[Excel - Source Data] --> B[n8n Workflow Engine]
    B --> C[Data Cleaning & Normalization]
    C --> D[Geocoding API]
    D --> E[Lat/Long Enrichment]
    E --> F[Webhook API Layer]
    F --> G[Frontend Map Engine]

