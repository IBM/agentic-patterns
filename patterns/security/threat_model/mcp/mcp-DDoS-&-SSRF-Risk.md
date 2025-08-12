## Scenario:
An MCP server is deployed in a production environment using Docker and exposed to the internet on a public IP. The server is configured with several tools that allow AI agents to make HTTP requests (e.g., fetch data, crawl URLs, test APIs). Some of these tools accept unvalidated user input such as target URLs or payloads.

An attacker finds the endpoint and repeatedly invokes tools with malicious payloads – launching a denial-of-service attack, or using SSRF to probe internal metadata servers or cloud services (e.g., `http://169.254.169.254` on AWS).

## Threat Landscape:
MCP servers are attractive targets due to their high privilege and flexible tool invocation. Poorly validated inputs passed to internal request tools can lead to open SSRF vectors, and public exposure allows mass scanning. Lack of rate limiting or access control exacerbates the damage.

## Assets (A):
* A01: Internal metadata service (e.g., cloud tokens, instance info).
* A02: Availability of MCP server and tools.
* A03: Underlying infrastructure/network configuration.

## Threat Actors (TA):
* TA01: External attacker probing for SSRF.
* TA02: Botnet launching DDoS on open MCP endpoints.

## Security Controls (C):
* C01: Input validation + allowlisting of URLs.
* C02: Use reverse proxies with rate limiting and WAF rules.
* C03: Block internal IP ranges in HTTP tools.
* C04: Never expose internal MCP endpoints directly to the public internet.

## Zones:
* Public Internet (untrusted)
* MCP Server (containerized)
* Internal Cloud Network (with metadata endpoints)

```mermaid
flowchart TD
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph public [Internet]
    style public stroke:#ff0000,stroke-dasharray: 5 5;
    TA01[TA01: SSRF Prober]:::threatActor
    TA02[TA02: DDoS Botnet]:::threatActor
  end

  subgraph mcp_zone [MCP Server]
    style mcp_zone stroke:#ff0000,stroke-dasharray: 5 5;
    mcp[MCP Container]:::system
    C01[[C01 Input Validation]]:::control
    C02[[C02 Rate Limit + WAF]]:::control
  end

  subgraph cloud [Cloud Metadata Zone]
    style cloud stroke:#ff0000,stroke-dasharray: 5 5;
    A01[A01: Metadata Service]:::asset
  end

  TA01 --> mcp
  TA01 -. triggers SSRF .-> A01
  TA02 --> mcp
  C01 -. filters .-> mcp
  C02 -. protects .-> mcp
```

