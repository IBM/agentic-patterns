## Scenario:
An MCP server includes a GraphQL-based tool (e.g., schema explorer, dashboard query, reporting) that communicates with an internal GraphQL API. For convenience, introspection is enabled to allow agents to understand schema types and query structures. However, this introspection endpoint is also exposed to unauthenticated users or indirectly accessible from prompt-engineered LLM calls.

An attacker could use the exposed introspection to enumerate all types, fields, and mutations – then craft malicious queries to extract sensitive fields not meant for public or AI use.

## Threat Landscape:
GraphQL introspection endpoints, if exposed, allow full visibility into backend data models. Prompt injection or unvalidated tool invocations can give LLMs or attackers access to internal data logic, field names, and even admin mutations.

## Assets (A):
* A01: GraphQL schema (includes hidden mutations or internal fields).
* A02: Data exposed via GraphQL (e.g., user secrets, logs).

## Threat Actors (TA):
* TA01: Prompt injector or user bypassing normal UI.
* TA02: Scripted attacker scanning for introspection endpoints.

## Security Controls (C):
* C01: Disable introspection in production.
* C02: Use GraphQL query depth limiting and rate limiting.
* C03: Apply field-level auth controls even on introspected types.

## Zones:
* User (Agent prompt or API)
* MCP Server
* Internal GraphQL API

```mermaid
flowchart LR
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph user_zone [User Agent or Prompt]
    style user_zone stroke:#ff0000,stroke-dasharray: 5 5;
    TA01[TA01: Prompt Injector]:::threatActor
  end

  subgraph mcp_zone [MCP Server]
    style mcp_zone stroke:#ff0000,stroke-dasharray: 5 5;
    MCP[MCP Server with GraphQL Tool]:::system
    C01[[C01 Disable Introspection]]:::control
  end

  subgraph gql [GraphQL API]
    style gql stroke:#ff0000,stroke-dasharray: 5 5;
    A01[A01: Schema Exposure]:::asset
    A02[A02: Internal Data Fields]:::asset
  end

  TA01 --> MCP
  MCP --> A01
  MCP --> A02
  C01 -. disables .-> A01
```