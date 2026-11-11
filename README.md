# Ravanan

### Ten Minds. One Intelligence.

A personal project to build and experiment with a self-hosted AI automation using multiple specialised agents.

The idea is to have different AI agents handle different tasks, such as **managing emails**, finding **job opportunities**, **organising personal tasks**, and **conducting research**. Instead of building each agent as a completely separate application, I want them to work together using a **shared AI infrastructure**.

I'm using this project to explore local LLMs, AI agents, automation, containerisation, and how different services can be connected to build a useful personal assistant system.

## Why the name Ravanan?

The name is inspired by Ravanan from **Indian mythology**.

Ravanan was known for his **knowledge, abilities, determination, and ten heads**.


His story reminds us that:
> Knowledge without wisdom, and power without self-control, can lead to destruction.

---

## What am I building?

The plan is to create a modular platform with:

- **AI Core:** A central service that handles LLM requests and provides shared intelligence.
- **AI Agents:** Independent agents designed for specific tasks.
- **Automation:** Scheduled tasks, background jobs, and automated workflows.
- **Integrations:** Connections to services such as Gmail, Notion, calendars, and external APIs.
- **Infrastructure:** Containerised services that can be developed, deployed, and maintained independently.
- **Monitoring:** Basic logging, health checks, and resource monitoring.

---

## Planned Agents

| Agent | Purpose |
|---|---|
| Email Agent | Email classification and management |
| Job search Agent | Job discovery and matching |

> These are initial ideas. More agents and capabilities may be added as the project evolves.

---

## Technology Stack

The initial technology choices include:

- Python
- FastAPI
- Ollama
- Docker
- Docker Compose
- SQLite
- PostgreSQL (planned)
- Redis (planned)

The intention is to run most of the platform locally and use a shared LLM service rather than running a separate model for every agent.


---


# License and Usage
### Copyright (c) 2026. All rights reserved.

This repository is publicly available for viewing and exploration.

You are welcome to browse the source code, study the implementation, and fork the repository on GitHub.

However, no permission is granted to reuse, reproduce, modify, redistribute, or incorporate any part of this project's source code into other projects without prior written permission from the author.

If you are interested in using any part of this project, please contact me to discuss permission.

email : saikrishna.freelances@gmail.com

Public availability does not mean the source code is released under an open-source licence.


