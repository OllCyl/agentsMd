# AI infra lager

| Lager | Ansvar |
| --- | --- |
| **Data layer** | Få in, rengöra, transformera och kvalitetssäkra rådata |
| **Knowledge layer** | Göra data till sökbar/användbar kunskap |
| **Integration layer** | Ge AI-systemet tillgång till externa system och funktioner |
| **AI application layer** | Bygga själva AI-beteendet och applikationslogiken |
| **Model layer** | Abstrahera och tillhandahålla modeller |
| **Platform layer** | Säkerhet, drift, observability, governance och deployment |

```mermaid
flowchart TB

    APP["AI APPLICATION LAYER<br/><br/>
    RAG · Prompt management<br/>
    Tool calling · Agent runtime<br/>
    Agent loop · Guardrails"]

    DATA["DATA LAYER<br/><br/>
    Samla in · Förbereda<br/>
    Kvalitetssäkra"]

    KNOW["KNOWLEDGE LAYER<br/><br/>
    Dela upp · Göra sökbart<br/>
    Söka · Rangordna"]

    INT["INTEGRATION LAYER<br/><br/>
    System · Tjänster<br/>
    Verktyg · Processer"]

    MODEL["MODEL LAYER<br/><br/>
    Språkmodell · Sökmodell<br/>
    Relevansmodell"]

    PLATFORM["PLATFORM LAYER<br/><br/>
    Behörighet · Säkerhet<br/>
    Övervakning · Utvärdering<br/>
    Loggning · Drift"]

    DATA --> KNOW
    KNOW --> APP
    INT --> APP

    MODEL -. stödjer .-> APP
    MODEL -. stödjer .-> KNOW

    PLATFORM -. stödjer .-> DATA
    PLATFORM -. stödjer .-> KNOW
    PLATFORM -. stödjer .-> INT
    PLATFORM -. stödjer .-> APP
    PLATFORM -. stödjer .-> MODEL
```

## LLM flöde enkel

```mermaid
flowchart LR
    Input --> LLM --> Output
```

## RAG enkel

```mermaid
flowchart TD
    Question --> RI[Retrieve information] --> LLM --> Answer
```

## Tool enkel

```mermaid
flowchart TD
    Question --> L1[LLM] --> Tool --> Result --> L2[LLM] --> Answer
```

## Workflow enkel

```mermaid
flowchart TD
    Input --> L1[LLM] --> Tool --> L2[LLM] --> RAG --> L3[LLM] --> Output
```

## Agent enkel

```mermaid
flowchart TD
    Goal --> L1[LLM]
    L1 --> Q1["Vad behöver jag göra?"]
    Q1 --> RAG
    Q1 --> Tool
    RAG --> Resultat
    Tool --> Resultat
    Resultat --> L2[LLM]
    L2 --> Q2["Vad gör jag nu?"]
    Q2 -->|Ny iteration| Q1
    Q2 -->|Klar| Done["Goal nått"]
```
