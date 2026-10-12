![n8n](https://img.shields.io/badge/n8n-FF6D5A?style=flat&logo=n8n&logoColor=white)
![Google Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=flat&logo=googledrive&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=googlesheets&logoColor=white)
![OCR.space](https://img.shields.io/badge/OCR.space-API-007BFF?style=flat)
![OpenRouter](https://img.shields.io/badge/OpenRouter-API-6B46C1?style=flat)
![Generative AI](https://img.shields.io/badge/Generative_AI-000000?style=flat&logo=openai&logoColor=white)

🌐 **[🇧🇷](README.md)** | **[🇬🇧](README_en.md)**

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

---

## 🔄 n8n Workflows

### Workflow 1: D&O Policy Processing & Structuring

- **File:** [`workflows/workflow01.json`](workflows/workflow01.json)
- **Role:** Monitors the Google Drive Entrada folder every 5 minutes, verifies MIME types, downloads files, dispatches them to OCR.space API, consolidates multi-page text via JavaScript ($input.all()), runs structured LLM extraction with a Structured Output Parser, and saves/updates the record in Google Sheets using the filename as primary key.

### Workflow 2: D&O Policy Query & Comparison Assistant

- **File:** [`workflows/workflow02.json`](workflows/workflow02.json)
- **Role:** Acts as the chat backend. Binds the When chat message received trigger to an AI Agent equipped with conversational memory (Simple Memory - 5 exchanges) and a query tool (Google Sheets Tool) linked to the Dados tab. The agent processes extracted policy parameters and generates concise comparative answers.

### Workflow 3: Policy Upload Pipeline

- **File:** [`workflows/workflow03.json`](workflows/workflow03.json)
- **Role:** Exposes a public n8n Form Trigger accepting multi-file submissions. A custom JavaScript node parses incoming binary streams and streams the files into Google Drive's Entrada folder, preserving original filenames for traceability.

## 🛠️ Tech Stack

| Layer	| Technology	| Description / Project Role |
| :--- | :--- | :--- |
| **Workflow Orchestration**	| **n8n** (Self-Hosted)	| Central automation engine deployed on Oracle Cloud Infrastructure (OCI) hosted under bot.rsa.ia.br. |
| **Computer Vision (OCR)**	| **OCR.space API**	| Conversion of digitized paperwork, scanned forms, and images into machine-readable text.|
| **Large Language Models (LLMs)**	| **OpenRouter API**	| Unified inference gateway. Models utilized: nvidia/nemotron-3-super-120b-a12b:free (structured extraction), cohere/north-mini-code:free (ultra-fast agent inference), and meta-llama/llama-3.1-8b-instruct:free (resilient fallback with native Tool Calling).|
| **Document Storage**	| **Google Drive**	| Storage repository for raw files (Entrada folder) equipped with scheduled polling triggers.|
| **Tabular Database**	| **Google Sheets**	| Base_Apolices_DO spreadsheet (Dados sheet) serving as structured persistent storage and fast lookup tool.|
| **Web Frontend**	| **HTML5, CSS3, JavaScript**	| Modern responsive interface embedding @n8n/chat, fully decoupled via config.js.|
| **Static Hosting**	| **GitHub Pages**	| Fast, secure, and maintenance-free static web hosting for user distribution.|

--- 

## 🧠 Architectural Decisions & Rationale

1. **Frontend/Backend Decoupling (config.js):** The static interface is completely decoupled from n8n backend logic. Connection URLs (chat webhook and file upload form) are centralized in config.js, allowing seamless infrastructure migrations without editing HTML/CSS.
2. **Batch Handling:** The JavaScript node in Workflow 1 iterates through $input.all(), handling array structures from multi-page OCR responses to guarantee concurrent uploads are processed without loss of document context.
3. **Extraction Resilience & Anti-Hallucination Guardrails:** The extractor prompt forces the model to return "Não encontrado" (Not Found) if a target field is not explicitly present in the OCR text. Furthermore, the AI Agent system prompt prohibits fabricating missing values, upholding legal compliance.
4. **Multi-Model Fallback Strategy:** To handle rate limits and availability fluctuations within free-tier APIs, the architecture accommodates seamless switching between LLM providers via OpenRouter.
5. **Contract Synchronization (Disabling Streaming):** The self-hosted n8n instance on OCI operates behind a reverse proxy that buffers Server-Sent Events (SSE) packets. Streaming generated fragmented chunks and frontend JSON parsing failures (SyntaxError). Disabling streaming on both the Chat Trigger and AI Agent nodes established stable, atomic payload delivery.
6. **Mitigating Timeouts & Keepalive Hallucinations:** n8n dispatches { "type": "keepalive" } heartbeats after 15 seconds of idle wait, triggering degraded frontend parser behavior. Adopting ultra-fast inference models (< 10 seconds), such as cohere/north-mini-code, eliminated keepalive timeouts.
7. **Strict Tool Calling Compatibility:** Certain lightweight models fail when formatting structured arguments for n8n tools (e.g., omitting required parameters). Selecting models with strict schema adherence (like Llama 3.1) ensured deterministic query executions.

## ⚙️ Installation & Setup Guide

### Prerequisites

- Running **n8n** instance (v1.x or later) reachable via public HTTPS endpoints.
- Configured API keys and accounts:
    - Google Cloud Console (OAuth2 credentials for Google Drive and Google Sheets).
    - **OCR.space** API Key.
    - **OpenRouter** API Key.

### 1. Backend Setup (n8n)

1.1. Open your n8n dashboard and navigate to **Workflows** > **Import from File**.

1.2. Import the three workflows from the `workflows/` folder:
    - workflow01.json
    - workflow02.json
    - workflow03.json

1.3. Configure credentials in n8n, linking your API keys and Google OAuth accounts.

1.4. In Google Drive, create a folder named `Entrada` and note its Folder ID.

1.5. In Google Sheets, create a spreadsheet named `Base_Apolices_DO` with a tab named `Dados` and the following headers:

| Arquivo | Seguradora | Segurado | Limite_Indenizacao | Vigencia |

1.6. Update the workflow nodes with your respective Folder ID and Spreadsheet ID.

1.7. Activate all three workflows (`Active = True`).

### 2. Frontend Setup

2.1. Clone this repository:
```bash
git clone https://github.com/RAguiarEng/n8n-policy-comparator.git
```

2.2. Create `config.js` based on `config_example.js`, populating your production n8n webhook and form URLs:
```js
const AppConfig = {
    googleDriveUrl: "https://YOUR_N8N_DOMAIN/form/YOUR_FORM_ID",
    n8nWebhookUrl: "https://YOUR_N8N_DOMAIN/webhook/YOUR_WEBHOOK_ID/chat"
};
```

2.3. Host the static files (`index.html`, `style.css`, `config.js`) on any web server or enable **GitHub Pages** in repository settings.

## 📁 Repository Structure

```text
n8n-policy-comparator/
├── .gitignore
├── config.js                     # Dynamic endpoint configuration (Webhook/Form)
├── favicon.ico / favicon.svg     # Application icons
├── index.html                    # Main UI (Landing Page and Chat Assistant)
├── style.css                     # Modern stylesheet and design system
├── README.md                     # Main documentation (Portuguese)
├── README_EN.md                  # Main documentation (English)
├── technical_report.md           # Full technical report for the capstone project
├── img/                          # Diagram assets and workflow screenshots
│   ├── analise-documentos.webp
│   ├── workflow01_pt.png
│   ├── workflow02_pt.png
│   └── workflow03_pt.png
├── workflows/                    # Exported n8n workflows in JSON format
│   ├── workflow01.json           # Workflow 1: Ingestion, OCR & Structuring
│   ├── workflow02.json           # Workflow 2: Query & Comparison Assistant
│   └── workflow03.json           # Workflow 3: Policy Upload Pipeline
└── artefatos/                    # Mandatory course completion deliverables
    ├── InsurMinds_Projeto_Final.pptx                    # Pitch Deck presentation
    ├── InsurMinds_Projeto_Final.mp4                     # Demonstration video (<= 5 min)
    └── GenAI_Seguros_InsurMinds_Projeto_Final.pdf       # Technical report in PDF format
```

## ⚠️ Known Limitations

- **Free-Tier OCR API:** OCR.space's free tier imposes rate limits and may experience lower extraction accuracy or page truncation on low-resolution scans or files exceeding 3 pages.
- **Google Drive Polling Interval:** The ingestion trigger runs on a 5-minute polling cycle, meaning ingestion is near-real-time rather than instantaneous.
- **Structured Schema Scope:** The MVP extracts the 4 primary pillars (Insurer, Policyholder, Limit of Liability, and Policy Period). Specific sub-limits, deductibles, and tailored endorsements remain roadmap items.
- **Data Normalization:** Policy monetary values and dates are persisted in their extracted textual format without automatic currency conversion or ISO date normalization.
- **Reverse Proxy Streaming Incompatibility:** Cloud hosting topologies that buffer Server-Sent Events necessitate disabling real-time streaming tokens on chat nodes.

## 🚀 Future Roadmap

- **RAG Architecture with Vector Databases:** Introduce vector storage (e.g., Pinecone or Qdrant) for deep semantic searches across extensive general terms and complex exclusion clauses.
- **High-Performance Document Intelligence:** Migration to specialized enterprise document parsers (e.g., Azure Document Intelligence or AWS Textract).
- **Automated Data Normalization & Typing:** Implement standardization pipelines for cross-currency conversion, numerical threshold calculations, and unified ISO date parsing.
- **Access Control & Privacy (LGPD/GDPR):** Incorporate user authentication, brokerage-level tenancy segregation, and rigorous data retention policies for sensitive records.

## 📄 License

This project is licensed under the [MIT License](LICENSE).