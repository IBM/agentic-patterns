
## Scenario:
An MCP plugin integrates with a local LLM like Ollama or LMStudio to generate responses from private tools. The tool passes raw user inputs to the model, and the model is not sandboxed or validated. A prompt injection modifies the tool’s instructions to bypass restrictions, invoke hidden tools, or exfiltrate data via response tokens.

This attack allows privilege escalation or leaking internal tool config even when the AI runs locally.

## Threat Landscape:
Local models offer speed and control, but they lack hardened isolation or content filtering. If user input is passed without sandboxing or output filtering, LLMs can hallucinate tool names, synthesize requests to fake endpoints, or leak sensitive instructions.

## Assets (A):
* A01: Internal tool instructions or config.
* A02: LLM-generated tool calls or text.
* A03: Response logs stored for auditing.

## Threat Actors (TA):
* TA01: Prompt attacker chaining local model with crafted input.
* TA02: Rogue user testing jailbreaks on local model.

## Security Controls (C):
* C01: Output filtering before executing tool calls.
* C02: Context chunking and isolation.
* C03: Do not serialize LLM outputs directly into tool requests.

## Zones:
* Local LLM Runtime
* MCP Server
* Tools or File System

```mermaid
flowchart LR
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph user [Prompt Input]
    style user stroke:#ff0000,stroke-dasharray: 5 5;
    TA01[TA01: Prompt Attacker]:::threatActor
  end

  subgraph mcp [MCP Plugin]
    style mcp stroke:#ff0000,stroke-dasharray: 5 5;
    localLLM[Local LLM - Ollama]:::system
    A01Config[A01: Tool Config]:::asset
    A02Text[A02: Response Text with Call]:::asset
    C01[[C01 Output Filtering]]:::control
  end

  TA01 --> localLLM
  localLLM --> A02Text
  A02Text --> ToolCall[Actual Tool Invocation]
  localLLM --> A01Config
  C01 -. filters .-> A02Text
```