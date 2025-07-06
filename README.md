# AI-Powered SEC Filing Analysis Engine

This project is a sophisticated, full-stack application designed to automate the analysis of corporate SEC filings. It leverages Large Language Models (LLMs) to dissect dense financial documents (10-K and 10-Q reports) and transforms them into structured, actionable insights. The system provides a comprehensive overview of a company's business, operational performance, risk profile, and historical financial trends through an intuitive web interface.

As this repository is currently on [Gitea](https://about.gitea.com/) hosted on private server, this README serves as a public showcase of the project's architecture and technical capabilities.

---

## Key Features

*   **Comprehensive AI Analysis**: Extracts and analyzes three critical sections of a filing:
    *   **Business Overview**: Company operations, products, competitive advantages, and strategic initiatives.
    *   **Management Discussion & Analysis (MD&A)**: Key performance indicators, operational results, liquidity, and management outlook.
    *   **Risk Factors**: Identifies and categorizes the top 10 most significant risks, assessing their severity and likelihood.
*   **Interactive Financial History**: Visualizes historical financial data from multiple filings, allowing for trend analysis of key metrics from the Income Statement, Balance Sheet, and Cash Flow Statement.
*   **High-Performance Concurrent Backend**: Employs a parallel processing architecture to analyze multiple document sections simultaneously, significantly reducing end-to-end analysis time.
*   **Robust Model Evaluation Framework**: Includes a dedicated pipeline to benchmark the AI model's performance against a "golden dataset," measuring accuracy, completeness, latency, and cost-efficiency.
*   **Structured & Reliable Data Extraction**: Utilizes Pydantic models to enforce a strict, validated schema on the unstructured output from the LLM, ensuring data integrity.
*   **Containerized Deployment**: Fully containerized with Docker and Docker Compose for reproducible and scalable deployment.

---

## Technical Architecture

The system is designed with a modular, service-oriented architecture that separates concerns between data acquisition, AI-powered analysis, and user presentation.

```
  User Request (e.g., "Analyze AAPL")
        |
        v
+-----------------------+      +----------------------+      +--------------------+
|   FastAPI Web Server  |----->|  SEC EDGAR Service   |----->|  EDGAR API (sec.gov) |
| (main.py, routers)    |<-----| (edgar_cache.py)     |<-----|                    |
+-----------------------+      +----------------------+      +--------------------+
        |
        | (Filing Text)
        v
+-----------------------------------+      +-----------------------------+
|      Analysis Core Engine         |----->|    LLM Provider (OpenAI)    |
| (business.py, mda.py, risk.py)    |      | (via OpenRouter)            |
| - Concurrent Processing           |<-----|                             |
| - Pydantic Model Validation       |      +-----------------------------+
| - Cost Tracking                   |
+-----------------------------------+
        |
        | (Structured JSON Analysis)
        v
+-----------------------+      +--------------------+
|   FastAPI Web Server  |----->|      Database      |
| - Renders Templates   |      | (database.py)      |
+-----------------------+      +--------------------+
        |
        v
  User Interface
(HTML, Chart.js)
```

### Core Components

*   **Backend**: Built with **Python** and **FastAPI**, providing a high-performance, asynchronous API. **Uvicorn** serves as the ASGI server.
*   **AI Analysis Engine**:
    *   A generic `BaseAnalyzer` class provides a reusable foundation for interacting with the LLM.
    *   Specialized analyzers (`BusinessAnalyzer`, `MDAAnalyzer`, `RiskFactorAnalyzer`) inherit from the base and contain component-specific prompts and logic.
    *   **Concurrent execution** via `ThreadPoolExecutor` parallelizes LLM calls for different document sections, a key optimization that drastically reduces latency.
    *   **Pydantic models** are used extensively to define the expected JSON schema for each analysis type, providing automatic validation and error handling for the LLM's output.
*   **Data Layer**:
    *   A file-based cache (`edgar_cache.py`) stores downloaded SEC filings to minimize redundant calls to the EDGAR API.
    *   A database layer (interfaced via `database.py`) is in place for storing analysis results and other application data.
*   **Frontend**:
    *   Server-side rendered HTML using the **Jinja2** templating engine.
    *   Dynamic, interactive visualizations are powered by **Chart.js** and custom **JavaScript**, allowing users to filter data and change chart types.
    *   **Bootstrap 5** is used for a responsive and modern UI.
*   **Evaluation & Cost Management**:
    *   The `model_evaluation.py` script provides a robust pipeline for comparing model output against a manually verified "golden" dataset.
    *   It calculates key metrics like *parse rate*, *field fill rate*, *latency*, and *cost-efficiency* (cost-per-analysis).
    *   A custom `CostRegistry` and `CostTracker` monitor API costs for each component of the analysis, enabling fine-grained optimization.

---

## Challenges & Solutions

This project involved several interesting technical challenges:

1.  **Challenge**: Processing large, multi-section financial documents in a time-efficient manner. A sequential analysis would be too slow for a responsive user experience.
    *   **Solution**: I designed and implemented a concurrent processing architecture. The system breaks the document into logical sections based on SEC filing structure (Business, MD&A, Risks, etc) and dispatches analysis tasks to a thread pool. This parallel execution model dramatically reduces the total processing time from minutes to seconds.

2.  **Challenge**: Ensuring reliable and structured data from the non-deterministic, text-based output of Large Language Models.
    *   **Solution**: I implemented a multi-layered approach. First, I engineered highly specific prompts for each analysis component. Second, I defined strict **Pydantic models** that act as a validation layer, parsing the LLM output and coercing it into a predictable, typed data structure. The application can confidently handle the structured data downstream, and validation errors immediately flag issues with the LLM's output.

3.  **Challenge**: Managing and tracking the operational cost of using third-party AI APIs, which can be significant at scale.
    *   **Solution**: I built a lightweight cost-tracking system (`CostRegistry`, `CostTracker`). This utility intercepts API call information and logs the token usage and associated costs for every part of the analysis. This data is then used in the evaluation framework to calculate cost-efficiency metrics, enabling data-driven decisions when optimizing prompts or comparing different models.

---

This project demonstrates a powerful, practical application of what AI can achieve in the financial domain. 
It is still under development and one of most important features (I'd very much like to do it next) is to implemented Machine Learning pipeline to perform sentiment analysis towards a company use LLM-generated analysis.

---

Demo Video:

<p align="center">
  <video autoplay="autoplay" loop="loop" muted="muted" playsinline="playsinline" width="700">
    <source src="./assets/sec-filing-analysis-demo.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</p>
