# AI infra byggstenar

| Lager | Ansvar |
| --- | --- |
| **Data layer** | Få in, rengöra, transformera och kvalitetssäkra rådata |
| **Knowledge layer** | Göra data till sökbar/användbar kunskap |
| **Integration layer** | Ge AI-systemet tillgång till externa system och funktioner |
| **AI application layer** | Bygga själva AI-beteendet och applikationslogiken |
| **Model layer** | Abstrahera och tillhandahålla modeller |
| **Platform layer** | Säkerhet, drift, observability, governance och deployment |

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
