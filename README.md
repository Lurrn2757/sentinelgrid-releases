<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&height=220&text=SentinelGrid&fontAlign=50&fontAlignY=40&color=0:0f172a,35:111827,70:1e293b,100:0ea5e9&fontColor=ffffff&desc=Infrastructure%20Monitoring%20%7C%20Alerting%20%7C%20Control%20Plane&descAlign=50&descAlignY=65&animation=fadeIn" alt="SentinelGrid banner" />
</p>

<p align="center">
  <a href="https://github.com/Lurrn2757/sentinelgrid/releases">
    <img src="https://img.shields.io/github/v/release/Lurrn2757/sentinelgrid-releases?style=for-the-badge&logo=github&label=latest%20release" />
  </a>
  <img src="https://img.shields.io/badge/status-active%20development-0ea5e9?style=for-the-badge" />
  <img src="https://img.shields.io/badge/platform-linux%20%7C%20docker%20%7C%20proxmox-111827?style=for-the-badge" />
  <img src="https://img.shields.io/badge/monitoring-self--hosted-22c55e?style=for-the-badge" />
  <img src="https://img.shields.io/badge/focus-reliability%20%26%20control-f97316?style=for-the-badge" />
</p>

<p align="center">
  <b>SentinelGrid</b> is a self-hosted infrastructure monitoring and alerting platform built for operators who want
  <b>visibility, actionability, and control</b> — without surrendering their stack to someone else’s cloud.
</p>

<p align="center">
  <i>Observe. Detect. Alert. Recover. Repeat.</i>
</p>

---

## ✨ What is SentinelGrid?

**SentinelGrid** is a modular monitoring platform for infrastructure, services, and operational workflows.

It combines:

- **agent-based telemetry collection**
- **central ingestion and storage**
- **alert routing and event lifecycle management**
- **Telegram-based operational control**
- **Grafana-compatible visualization**
- **Proxmox-aware infrastructure actions**
- a transition path toward the **native Sentinel Engine**

This repository contains the **source code and core platform logic**.

> For packaged public releases, installers, and deployment assets, visit:  
> **[sentinelgrid-releases](https://github.com/Lurrn2757/sentinelgrid-releases)**

---

## 🔥 Why SentinelGrid?

Because basic dashboards are cute.

SentinelGrid is being built to be a **real operator platform** — not just “graphs on a screen.”

It is meant to help you:

- **collect host and infrastructure metrics**
- **track failures and incident states**
- **receive actionable alerts**
- **perform controlled operational actions**
- **retain visibility during outages**
- **evolve toward an in-house alerting engine**

In short:

> **SentinelGrid is the bridge between monitoring, response, and control.**

---

## 🧠 Core philosophy

SentinelGrid is built around a few simple ideas:

- **Own your monitoring**
- **Keep the stack observable**
- **Don’t lose data during outages**
- **Treat alerts like stateful events, not noisy spam**
- **Give operators tools, not just notifications**
- **Design for the future native Sentinel Engine**

---

## ⚙️ Current architecture

```text
Agents / Proxmox / Hosts
          │
          ▼
   Sentinel Ingestion API
          │
          ▼
      PostgreSQL
          │
          ├── Grafana Dashboards
          ├── Telegram Bot / Alert Workflows
          ├── Operational Actions
          └── Alert Source Integration
                ├── Zabbix Adapter (current)
                └── Sentinel Engine (planned / evolving)
