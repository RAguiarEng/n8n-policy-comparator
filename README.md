![n8n](https://img.shields.io/badge/n8n-FF6D5A?style=flat&logo=n8n&logoColor=white)
![Google Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=flat&logo=googledrive&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=googlesheets&logoColor=white)
![OCR.space](https://img.shields.io/badge/OCR.space-API-007BFF?style=flat)
![OpenRouter](https://img.shields.io/badge/OpenRouter-API-6B46C1?style=flat)
![Generative AI](https://img.shields.io/badge/Generative_AI-000000?style=flat&logo=openai&logoColor=white)

# GenAI Seguros - Plataforma Inteligente para Análise e Comparação de Apólices D&O

> **Projeto Final - I2A2 (Instituto de Inteligência Artificial Aplicada)**  
> 🔗 **[Acessar a Aplicação Web](https://raguiareng.github.io/n8n-policy-comparator/)**

---

## 👥 Integrantes do Grupo

| Nome	| E-mail |
| --- | --- |
| Bruno Corrêa	| correabruno321@gmail.com |
| [Jhiovana Silva Ribeiro](https://github.com/jhsribeiro)	| jhiovanasilva11@gmail.com |
| Luis R G Pereira	| luisrgpereira@gmail.com |
| [Rodrigo Medeiros Costa](https://github.com/rodrigomdc)	| eng.rodrigomdc@gmail.com |
| [Rodrigo Souza Aguiar](https://github.com/RAguiarEng)	| rodrigo_souza_aguiar@hotmail.com |

---

## 📖 Descrição do Projeto

O **GenAI Seguros** é um protótipo funcional (MVP) baseado em Inteligência Artificial Generativa, desenvolvido para automatizar a extração, organização e comparação de informações presentes em apólices de seguro D&O (*Directors and Officers*). 

O processo tradicional de análise de apólices exige horas de trabalho manual de especialistas para identificar cláusulas, limites e vigências. Nossa solução resolve esse problema através de uma arquitetura orientada a eventos e agentes inteligentes, permitindo a ingestão de documentos (PDF/Imagens), extração via OCR, estruturação via LLMs e uma interface de chat para comparação em linguagem natural.

---

## 🏗️ Arquitetura da Solução

A solução foi dividida em dois módulos independentes (Workflows no n8n) para garantir a separação de responsabilidades (*Separation of Concerns*):

1. **Módulo de Ingestão Assíncrona (Workflow 1):**
   - Monitora uma pasta no Google Drive (`Entrada`).
   - Ao detectar novos arquivos, realiza o download e envia para a API do OCR.space.
   - Um script JavaScript consolida o texto extraído (suportando processamento em lote/batch).
   - Um LLM (via OpenRouter) recebe o texto bruto e extrai os dados estruturados (Seguradora, Segurado, Limite, Vigência).
   - Os dados são salvos/atualizados em um banco de dados (Google Sheets).

2. **Módulo de Consulta e Comparação (Workflow 2):**
   - Um Agente de IA (*AI Agent*) conectado a um gatilho de chat web.
   - O Agente possui acesso a uma ferramenta (*Tool*) de leitura do Google Sheets.
   - Ao receber uma pergunta, o Agente consulta a base de dados, interpreta as informações e formula uma resposta comparativa, formatada e amigável.

---

## 🛠️ Tecnologias Utilizadas

- **Orquestração e Backend:** n8n (Self-Hosted na Oracle Cloud Infrastructure - OCI)
- **Visão Computacional (OCR):** OCR.space API
- **Modelos de Linguagem (LLMs):** OpenRouter (Modelos utilizados: `poolside/laguna-s-2.1:free`, `nvidia/nemotron-3-super-120b-a12b:free`, `qwen/qwen3.8-27b:free`)
- **Armazenamento e Banco de Dados:** Google Drive e Google Sheets
- **Frontend:** HTML5, CSS3, JavaScript (Vanilla)
- **Hospedagem Web:** GitHub Pages

---

## n8n Workflows

### Workflow 1 - Processamento de Apólices D&O

Recuperação dos arquivos `.pdf`, `.jpeg`, `.png` e `.tiff` do Google Drive, processamento pela [OCRSpace](https://ocr.space/ocrapi), plano gratuito, e registro das informações necessárias para comparação na planilha principal `Base_Apolices_DO`. 

![workflow1](img/workflow01_pt.png)

### Worflow 2 - Assistente de Comparação D&O

Recebimento das informações do workflow 1, análise e comparação dos dados via LLM, cujo gatilho é solicitação via chat. 
Memória definida em cinco contextos.

![workflow2](img/workflow02_pt.png)

## Workflow 3 - Upload de apólices

Carregamento de arquivos `.pdf`, `.jpeg`, `.png` e `.tiff` para a pasta `Entrada` o Google Drive.

![workflow3](img/workflow03_pt.png)

---

## 🧠 Justificativas Arquiteturais

- **Desacoplamento Frontend/Backend:** A interface web foi construída separando HTML, CSS e um arquivo `config.js`. Isso permite que as credenciais e URLs de Webhook sejam alteradas sem modificar a estrutura da página.
- **Processamento em Lote (Batch):** O nó de código JavaScript no Módulo 1 foi otimizado para iterar sobre `$input.all()`, garantindo que múltiplos uploads simultâneos no Drive não resultem em perda de dados.
- **Resiliência a Falhas de Extração:** Optou-se por instruir o LLM de extração a preencher campos com `"Não encontrado"` caso o OCR falhe (comum em imagens de baixa resolução). O Agente de Comparação foi instruído a ler esse dado e informar o usuário de forma transparente, evitando alucinações (*hallucinations*).
- **Fallback de Modelos:** Devido aos limites de taxa (*Rate Limits*) de APIs gratuitas, a arquitetura permite a rápida substituição de modelos no OpenRouter para garantir a continuidade do serviço.

---

## ⚙️ Instruções de Instalação e Execução

### 1. Configuração do Frontend
1. Clone este repositório: `git clone https://github.com/RAguiarEng/n8n-policy-comparator.git`
2. Abra o arquivo `config.js` e insira o link da sua pasta do Google Drive e a URL do Webhook do seu n8n.
3. Hospede os arquivos em qualquer servidor web ou utilize o GitHub Pages.

### 2. Configuração do Backend (n8n)
1. Importe os arquivos `.json` localizados na pasta `workflows/` para a sua instância do n8n.
2. Configure as seguintes credenciais no n8n:
   - Google Drive API (OAuth2)
   - Google Sheets API (OAuth2)
   - OCR.space API Key
   - OpenRouter API Key
3. No Workflow 1, atualize os IDs da pasta do Drive e da planilha do Sheets.
4. Ative os dois workflows.

---

## ⚠️ Limitações Conhecidas

- **Limitações do OCR Gratuito:** Documentos muito extensos ou imagens de baixa qualidade podem sofrer truncamento ou falha na extração de texto pela API do OCR.space.
- **Latência de LLMs:** O uso de modelos com muitos parâmetros (ex: Nemotron 120b) pode gerar um tempo de resposta superior a 1 minuto na extração.
- **Gatilho de Polling:** O Google Drive Trigger verifica a pasta a cada 5 minutos, o que significa que a ingestão não é estritamente em tempo real.

---

## 🚀 Evolução Futura

- Implementação de um banco de dados vetorial (ex: Pinecone) para permitir buscas semânticas (RAG) no texto completo das apólices, além dos dados estruturados.
- Substituição do OCR.space por soluções mais robustas (ex: AWS Textract ou Azure Document Intelligence).
- Adição de autenticação no frontend para garantir que apenas corretores autorizados acessem o chat.

---


## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.