![n8n](https://img.shields.io/badge/n8n-FF6D5A?style=flat&logo=n8n&logoColor=white)
![Google Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=flat&logo=googledrive&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=googlesheets&logoColor=white)
![OCR.space](https://img.shields.io/badge/OCR.space-API-007BFF?style=flat)
![OpenRouter](https://img.shields.io/badge/OpenRouter-API-6B46C1?style=flat)
![Generative AI](https://img.shields.io/badge/Generative_AI-000000?style=flat&logo=openai&logoColor=white)

> 🌐 **[🇧🇷](README.md)

# GenAI Seguros - Intelligent Platform for D&O Insurance Policy Analysis and Comparison

> **Capstone Project - InsurMinds / I2A2 (Applied Artificial Intelligence Institute)**  
> 🔗 **[Access Production Web Application](https://raguiareng.github.io/n8n-policy-comparator/)**   
> 📄 **[Full Technical Report](technical_report.md)**   
> 🧑‍💻 **[Presentation Pitch Deck](artefatos/InsurMinds_Projeto_Final.pptx)**   
> 📹 **[Demonstration Video](artefatos/InsurMinds_Projeto_Final.mp4)** 

---

## 👥 Team Members

| Name | Email |
| --- | --- |
| Bruno Corrêa | correabruno321@gmail.com |
| [Jhiovana Silva Ribeiro](https://github.com/jhsribeiro) | jhiovanasilva11@gmail.com |
| Luis R G Pereira | luisrgpereira@gmail.com |
| [Rodrigo Medeiros Costa](https://github.com/rodrigomdc) | eng.rodrigomdc@gmail.com |
| [Rodrigo Souza Aguiar](https://github.com/RAguiarEng) | rodrigo_souza_aguiar@hotmail.com |

---

## 📖 Project Overview

**GenAI Seguros** is a functional prototype (MVP) powered by Generative Artificial Intelligence, designed to automate the extraction, structuring, and comparative analysis of insurance policies for D&O (*Directors and Officers*).

Insurance documents and D&O policies are characterized by extensive length, heterogeneous formatting, and intricate legal phrasing. The traditional comparative analysis workflow requires hours of meticulous, manual review by insurance brokers and underwriters. Our platform eliminates this bottleneck through an event-driven architecture and specialized intelligent components:

1. **Document Ingestion & Upload:** Streamlined ingestion of multiple document types (PDF, PNG, JPEG, TIFF) preserving file names for traceability.
2. **Extraction & Optical Character Recognition (OCR):** Conversion of physical/digital files into raw text via an OCR API.
3. **LLM-Based Structuring:** Deterministic key-field extraction (Insurer, Policyholder/Insured, Maximum Limit of Liability [LMI], and Policy Period) powered by JSON schema parsers and missing-field safeguards.
4. **Tabular Persistence:** Idempotent recording and updates directly into a Google Sheets spreadsheet (`Base_Apolices_DO`).
5. **Comparison Assistant:** Web-integrated AI Agent utilizing Tool Calling against the underlying database, enabling natural language inquiries and multi-policy comparisons.

---

## 📦 Final Project Deliverables

In accordance with the Capstone Project requirements, this delivery comprises:

1. **Technical Report:** Comprehensive document outlining architecture, rationale, technologies, limitations, and future enhancements ([technical_report.md](technical_report.md) and its generated PDF version).
2. **Source Code & Workflows:** Public GitHub repository containing the frontend web application, configuration scripts, and exported JSON workflows inside the `workflows/` directory.
3. **Presentation (Pitch Deck):** `InsurMinds_Projeto_Final.pptx` located in the artifacts directory.
4. **Demonstration Video:** Video presentation (up to 5 minutes) covering problem statement, architecture, live execution, and obtained results (`InsurMinds_Projeto_Final.mp4`).
5. **Artifacts Directory:** Dedicated `artefatos/` repository folder centralizing all delivery files.

---

## 📚 Data Sources and Test Policies

To validate and test the platform, test policy models and general terms aligned with **SUSEP (Superintendência de Seguros Privados)** regulatory guidelines and D&O market standards were used:
- **Policy Alfa (`Apolice_Alfa.pdf`):** Corporate test policy with standard indemnification limits and standard clauses.
- **Policy Beta (`Apolice_Beta.pdf` / `Apolice_Beta.png`):** Comparison policy with distinct validity periods, insurer, and LMI values, used to validate OCR capabilities across both image and PDF formats.

---

## 🏗️ Solution Architecture

The architecture adopts a multi-tiered approach enforcing Separation of Concerns, distributed across 3 modular **n8n** workflows:

```text
[ User ] 
    │
    ├─► 1. Document Upload (Frontend / Form Trigger)
    │        │
    │        ▼
    │   [ Workflow 3: Upload ] ──► Saves to Google Drive "Entrada" folder
    │
    ├─► 2. Asynchronous Ingestion & OCR (Google Drive Polling Trigger - 5 min)
    │        │
    │        ▼
    │   [ Workflow 1: Ingestion ]
    │        │
    │        ├─► File download (PDF / PNG / JPG / TIFF)
    │        ├─► OCR.space API (Raw text extraction)
    │        ├─► Code JS (Batch page text aggregation - $input.all())
    │        ├─► LLM Chain + Structured Output Parser (JSON extraction)
    │        └─► Google Sheets (Base_Apolices_DO / "Dados" tab - appendOrUpdate)
    │
    └─► 3. Interactive Query & Comparison
             │
             ▼
        [ Workflow 2: Chat Assistant ]
             │
             ├─► Chat Trigger (Frontend Web / @n8n/chat)
             ├─► AI Agent + Window Buffer Memory (5 interaction history)
             ├─► Tool Calling: Google Sheets Tool (Reads from "Dados" tab)
             └─► Natural Language comparative response formatted in Markdown
