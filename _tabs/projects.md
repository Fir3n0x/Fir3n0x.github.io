---
title: Projects
icon: fas fa-code
order: 1
---
A selection of personal projects spanning Android development, offensive security tooling, and AI. All source code is available on [GitHub](https://github.com/Fir3n0x).

---

## 📡 ESPion32 - OffSec Wi-Fi Tool
**Language:** Kotlin (Android) + C (ESP32 firmware) · **Status:** Active (v2.5.1)

An Android companion app that communicates with an ESP32 microcontroller over BLE to perform offensive Wi-Fi operations, including network sniffing and deauthentication attacks. The ESP32 firmware (written in C) is provided below, aswell as the UI repository. Built for educational and authorized use only.

**Stack:** Kotlin · BLE · ESP32 · C firmware

![ESPion32](assets/img/projects/espion32-app.jpg){: width="800"}

[GitHub](https://github.com/Fir3n0x/ESPion32){: .btn .btn-outline-primary }
[GitHub](https://githu.com/Fir3n0x/ESPion32-firmware){: .btn .btn-outline-primary }

---

## 🗺️ MiniMap - Wi-Fi Security Analyzer
**Language:** Kotlin · **Platform:** Android · **Status:** Active (v2.22.1)

An Android application that passively scans nearby Wi-Fi networks and classifies each one as **Safe**, **Medium**, or **Dangerous** using an on-device TensorFlow Lite model. Features a radar-like canvas UI, geo-tagging, JSON/CSV export, and an optional background worker that alerts when a new dangerous network is detected.

**Stack:** Kotlin · Jetpack Compose · Material 3 · TFLite · Coroutines · DataStore

![MiniMap](assets/img/projects/minimap.jpg){: width="400"}

[GitHub](https://github.com/Fir3n0x/MiniMap){: .btn .btn-outline-primary }

---

## 🎯 L4Go0n - C2 Framework
**Language:** Go (server) + C (agent) · **Platform:** Windows · **Status:** Active

A custom Command & Control framework built from scratch. The Go server handles agent connections and task dispatching, while two types of C agents can be deployed on Windows targets, a **basic** agent for standard operations and a **stealthy** agent designed for lower detection footprint. Encryption support is planned. Built for research and authorized red team use only.

**Stack:** Go · C · Windows API

![L4Go0n](assets/img/projects/L4Go0n.jpg)

[GitHub](https://github.com/Fir3n0x/L4Go0n){: .btn .btn-outline-primary }

---

## 🐀 RATz - Remote Access Trojan
**Language:** C++ · **Status:** In progress

A C++ RAT with remote control capabilities over a target Windows machine. Current features include **PPID spoofing**, **backdoor deployment**, and a **keylogger**. Persistence mechanisms are implemented. The project was started in December 2024 and is actively being developed.

**Stack:** C++ · Windows API

[GitHub](https://github.com/Fir3n0x/RATz){: .btn .btn-outline-primary }

---

## ♟️ OthelloAI - Reversi AI Engine
**Language:** Java · **Status:** Complete

A Java implementation of an Othello/Reversi AI using the **Minimax algorithm** with **alpha-beta pruning** and a custom heuristic evaluation function (corners, mobility, positional weight). Supports Human vs Human, Human vs AI, and AI vs AI modes via a console interface.

**Stack:** Java · Minimax · Alpha-Beta Pruning

[GitHub](https://github.com/Fir3n0x/OthelloAI){: .btn .btn-outline-primary }

---

## 🧩 DSL-Project - Othello Domain-Specific Language
**Language:** TypeScript · JavaScript · Python · **Status:** Complete

A full Domain-Specific Language built with **Langium** for specifying and generating Othello-like games and their AI players. Supports multiple board topologies (square, circular), several gameplay variants, and multiple play modes including Human vs AI, AI vs AI, and LLM-driven matches via **GPT-4o**. A notable finding from the project: GPT-4o consistently outperformed depth-limited Minimax AIs, suggesting LLMs capture long-term strategic patterns that depth-capped search misses.

**Stack:** Langium · TypeScript · Python · GPT-4o · Minimax

[GitHub](https://github.com/Fir3n0x/DSL-Project){: .btn .btn-outline-primary }

---

## 🚀 DevOps-INSA - Taquin Web App
**Language:** Various · **Status:** Complete

A university DevOps project built on top of a sliding puzzle (Taquin) web application developed during a web course at INSA Rennes. The DevOps layer covers a full CI/CD pipeline, Docker images, Kubernetes pod deployment, Prometheus dashboards for monitoring, and frontend/backend test suites.

**Stack:** Docker · Kubernetes · Prometheus · CI/CD

[GitHub](https://github.com/Fir3n0x/DevOps-INSA){: .btn .btn-outline-primary }

---

## 🎰 VoltorbFlipSolver - Pokémon Mini-Game Solver
**Language:** Java · **Status:** Complete

A solver for the **Voltorb Flip** casino mini-game from Pokémon HeartGold/SoulSilver. Because grinding coins manually to get Dragonite is a crime against humanity.

[GitHub](https://github.com/Fir3n0x/VoltorbFlipSolver){: .btn .btn-outline-primary }
