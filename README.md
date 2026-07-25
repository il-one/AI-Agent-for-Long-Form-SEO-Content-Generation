# 🚀 AI Long-Form Content & SEO Agent (Gemini + Opal Framework)

An agentic workflow built with **Google Opal** and the **Gemini API**. The agent accepts target keywords and parameters, generates an SEO-optimized outline, runs a self-correction reflection audit, and iteratively builds a comprehensive long-form article with optional AI-generated visuals.

---

## 🔄 Agentic Workflow Diagram

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

