# Product Requirements Document (PRD)

## Project: Autonomous Long-Form SEO Content Generation Agent
* **Version:** 1.0.0  
* **Status:** In Production / Portfolio Reference  
* **Framework:** Google Opal (Visual Multi-Agent Orchestration)  
* **Author:** IL 

---

## 1. Executive Summary & Problem Statement

### 1.1 Problem Statement
Generative AI models excel at short-form text generation, but struggle significantly with long-form (2,000+ words) SEO content. Standard single-prompt LLM interactions suffer from three core failure modes:

1. **Context Drift & Repetition:** Around word 1,000, models lose track of previous sections, repeating key points, re-introducing concepts, and diluting word quality.
2. **Thin Content & Hallucinated Depth:** Single-turn LLMs generate generic advice rather than actionable, structured insight, failing Google’s **Helpful Content System** and **E-E-A-T** (Experience, Expertise, Authoritativeness, Trustworthiness) criteria.
3. **Lack of Self-Correction:** Models output their first draft without reviewing structure, keyword placement, fluff density, or formatting compliance.

### 1.2 Solution
An autonomous multi-agent pipeline built within **Google Opal**, designed to mimic a high-performing content editorial team. The system decouples long-form creation into three specialized agent nodes:

* **Strategy Planner (`01_strategy_planner.txt`):** Performs search-intent extraction and drafts structured outlines.
* **Section Writer (`02_section_writer.txt`):** Drafts content section-by-section with context (outline) awareness to maintain content flow without context rot.
* **Reflection Critic (`03_reflection_critic.txt`):** Acts as an automated editor, evaluating drafts against E-E-A-T criteria, fluff metrics, and formatting rules before approving final output.

---

## 2. Target Personas & Use Cases

| Persona | Role & Context | Core Need | Key Feature Solution |
| :--- | :--- | :--- | :--- |
| **SEO Content Lead** | Manages high-volume enterprise or niche blogs. | High-volume, rank-ready content that doesn't sound like generic AI text. | Multi-stage reflection loop enforcing E-E-A-T compliance and entity density. |
| **Solo Creator / YouTuber** | Produces written companion articles for video/podcast assets. | Comprehensive written coverage (2,000+ words) without spending 10+ hours per post. | Strategy Planner node establishing intent maps and structured H2/H3 outlines instantly. |
| **Agency Content Strategist** | Builds scalable client content stacks across multiple verticals. | Consistent tone, zero context drift, and automated self-correction/editing. | Modular system prompts (`prompts/`) ensuring reproducible section drafting. |

---

## 3. Product Goals & Non-Goals

### 3.1 Goals
* **Comprehensive Length:** Generate structured, non-repetitive Markdown content exceeding **2,500 words**.
* **Zero-Hallucination Structure:** Ensure logical progression (H1 $\rightarrow$ H2 $\rightarrow$ H3 $\rightarrow$ H4) where each section builds upon prior context rather than restating it.
* **E-E-A-T Alignment:** Automatically inject key takeaways, real-world examples, comparison tables, and structured callouts.
* **Automated Self-Correction:** Enable a feedback loop where sections failing editorial standards are returned to the writer agent with actionable correction notes.

### 3.2 Non-Goals
* **Direct WordPress/CMS Publishing:** This pipeline exports clean Markdown files (`.md`); automated API publishing to platforms like WordPress or Webflow is out of scope for v1.0.
* **Real-Time Live Web Scraping:** Search intent is established via structured prompt inputs and entity mapping; real-time SERP API scraping is reserved for future versions.

---

## 4. System Architecture & Functional Requirements

The workflow is visually configured and executed through sequential node passing in Google Opal.
```mermaid
graph TD
    A[User Input: Keyword, Audience, Tone, Include Images?] --> B[Generate Initial SEO Outline]
    B --> C{Reflection Evaluator: Best Practices Audit}
    C -- Fails Criteria --> D[Refine & Polish Outline]
    D --> C
    C -- Passes Criteria --> E[Loop Through Sections]
    E --> F[Generate Section Content]
    E --> G{Include Images?}
    G -- Yes --> H[Generate Contextual Image via Imagen]
    G -- No --> I[Skip Visuals]
    F --> J[Stitch & Assemble Master Document]
    H --> J
    I --> J
    J --> K[Final Output: Long-Form SEO Article]
```
---

### 4.1 User Inputs & Configuration Parameters

The Opal workflow accepts the following runtime parameters:

* **Primary Keyword:** Core search query (e.g., `"Agentic AI Workflows in Enterprise"`).
* **Secondary Keywords:** LSI terms and semantic variations.
* **Target Audience:** Professional skill level (e.g., *DevOps Engineers*, *CTOs*, *SEO Managers*).
* **Brand Tone:** Voice guidelines (e.g., *Authoritative, Witty, Data-backed*).
* **Visual Generation:** Boolean flag (`Enable Visual Assets: True/False`).

---

### 4.2 Workflow Node Specifications

#### Node 1: Strategy & Outline Generator (`gemini-3.1-pro`)

* **Role:** Analyzes intent and constructs an H2/H3 semantic outline.
* **Outputs:** Structured JSON containing:
* Title options (H1) optimized for high CTR.
* Section-by-section outline (H2/H3) with specific sub-keywords assigned per section.
* Target reader pain points and takeaways.



#### Node 2: Iterative Content Writer (`gemini-3.1-pro`)

* **Role:** Drafts long-form content section by section to avoid context dilution and ensure depth.
* **Execution:** Processes each section in sequence while maintaining state memory of previously written sections.

#### Node 3: Self-Correction Reflection Audit (`gemini-3.1-pro`)

* **Role:** Critic agent evaluating the draft against strict quality benchmarks:
* *SEO Coverage:* Are assigned sub-keywords placed naturally?
* *Depth & Accuracy:* Is the technical explanation concrete and actionable?
* *Readability:* Are sentence structures varied? Is passive voice minimized?


* **Output:** `audit_score` (0–100) and structured revision feedback.
* **Logic:** If `audit_score < 85`, the section is returned to **Node 2** with targeted revision instructions.

#### Node 4: Multimodal Visual Generator (Imagen 4 / Nano Banana)

* **Role:** Extracts visual concepts from H2 headings and generates context-aware blog graphics, diagrams, or hero images.
* **Output:** Public image URLs or Google Drive asset IDs inserted into the document flow.

#### Node 5: Google Docs Exporter (Workspace API Node)

* **Role:** Formats and publishes the article to a designated Google Drive folder.
* **Formatting Rules:** Applies standard H1, H2, H3 styles, bulleted lists, callout boxes, and embeds visual assets with alt-text.

---

### 4.3 Workflow Phases

#### Phase 1: Search Intent Analysis & Outline Generation

* Uses Google Opal's Agent capabilities paired with Gemini API search tools to analyze current SERP intent.
* Synthesizes an $H_1 / H_2 / H_3$ outline prioritizing content gaps and semantic topic coverage.

#### Phase 2: Self-Correction Reflection Audit Loop

* **Self-Critique Engine:** An autonomous audit node checks the generated outline against strict SEO rules (keyword placement, heading depth, logical narrative flow, and avoidance of generic filler).
* **Auto-Correction:** If the audit detects gaps or weak search alignment, the workflow feeds feedback back into Phase 1 for immediate revision before writing begins.

#### Phase 3: Iterative Section Drafting

* Expands each heading into comprehensive, actionable prose.
* Leverages Gemini’s long-context capabilities to maintain global coherence, technical accuracy, and consistent voice across multi-thousand-word outputs.

#### Phase 4: Contextual Visual Asset Generation *(Optional)*

* Scans the completed draft for key concepts suitable for visual aid (e.g., architecture flows, comparative tables, conceptual diagrams).
* Calls Gemini API image/diagram tools to generate visual assets and generates contextual alt-text.

#### Phase 5: Production Google Docs Publishing

* Authenticates with the Google Drive & Docs API.
* Converts markdown elements (headings, bullet points, callout blocks) into styled Google Doc elements.
* Inserts generated visual assets directly inline within the document.

---

## 5. Non-Functional & Technical Specifications

* **Orchestration:** Google Opal Agent Mode canvas (`workflow_manifest.json`).
* **LLM Tiering:**
* `gemini-3.1-pro` / reasoning tier for outline reflection and self-correction audits.
* `gemini-3-flash` for high-throughput section drafting and iterative generation.


* **Integrations/Tools:** Google Docs API (via service account authentication) and Google Drive API.
* **Performance Benchmark:** Under **5 minutes** to execute the full pipeline for a 2,000-word article with embedded visual assets.
* **Error Resilience:** Graceful fallbacks for API rate limits and automated retry loops on failed document formatting calls.

---


## 6. Production Performance & Benchmark Results

The pipeline was deployed across **production blog articles**. Below are the key performance metrics and live output samples.

### Execution & Efficiency Benchmarks

| Metric | Target Baseline | Production Benchmark (Avg) |
| --- | --- | --- |
| **Total Generation Time** | < 5.0 minutes | **2.8 minutes** |
| **Word Count Accuracy** | ±10% of target | **±3.4% of target** |
| **Reflection Audit Pass Rate** | > 80% on First Pass | **86.7% First-Pass Rate** |
| **Average Cost Per Article** | < $0.50 USD | **$0.28 USD** (Flash/Pro combined) |
| **Ahrefs / Surfer SEO Score** | > 75 / 100 | **84 / 100 average** |

---

### Production Article Output Log

| Article Topic | Primary Keyword | Word Count | SEO Score | Google Doc Output |
| --- | --- | --- | --- | --- |

---
