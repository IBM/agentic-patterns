## Scenario 
A software developer uses an IDE plugin powered by an MCP server for code suggestions and refactoring assistance. The MCP server runs on the developer’s machine, exposing local development assets (code files, repository access) to an AI assistant. The IDE (MCP client) communicates with either a local model or a cloud AI API for generating code completions.
Threat Landscape: In this setup, threats include malicious code injection via the AI assistant (if an attacker compromised the plugin or prompt context), leakage of proprietary source code to external parties, or unauthorized repository access. A compromised or trojanized MCP plugin could exfiltrate code or credentials. Network eavesdroppers could hijack traffic if communications (to AI APIs or git servers) are not encrypted. The developer’s machine becomes an attack surface through the MCP channel.
## Assets (A):
- A01: Proprietary source code (the project files/repository content in the IDE) – valuable intellectual property.
- A02: Developer’s credentials (SSH keys, access tokens for code repo or API) stored for tool access.
## Threat Actors (TA):
- TA01: Malicious plugin or extension author (attacker distributes a compromised MCP plugin that executes unauthorized commands
[techcommunity.microsoft.com](https://techcommunity.microsoft.com/blog/microsoftdefendercloudblog/plug-play-and-prey-the-security-risks-of-the-model-context-protocol/4410829#:~:text=MCP%E2%80%99s%20absence%20of%20robust%20authentication,credentials%2C%20leading%20to%20potential%20breaches)
).
- TA02: Network eavesdropper on the LAN/Internet (capable of sniffing or tampering with MCP traffic if not secured).
## Security Controls (C):
- C01: Trusted plugin sources & code signing – ensure the MCP plugin is vetted and signed to prevent rogue code.
- C02: TLS encryption on all MCP communications with external services (e.g. AI API calls) to thwart MITM eavesdropping.
- C03: Sandbox and monitoring – restrict the plugin’s file/system access and use static code scans or runtime monitoring to detect any malicious operations on code (e.g. unauthorized file exfiltration).
## Zones:
- Developer Machine (IDE environment – trusted internal zone)
- Corporate LAN (enterprise network with code repository server)
- Internet (external AI API or plugin source – untrusted zone)

```mermaid
flowchart LR
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph zone_dev [Developer’s Machine]
    style zone_dev stroke:#ff0000,stroke-dasharray: 5 5;
    ide[IDE + MCP Plugin]:::system
    codeA01[A01: Source Code Repository]:::asset
    credA02[A02: Repo Credentials]:::asset
    C03[[C03 Code Scanner]]:::control
  end
  subgraph zone_corp [Corporate LAN]
    style zone_corp stroke:#ff0000,stroke-dasharray: 5 5;
    gitServer[Git Repository Server]:::system
  end
  subgraph zone_net [Internet]
    style zone_net stroke:#ff0000,stroke-dasharray: 5 5;
    TA01[TA01 Malicious Plugin Author]:::threatActor
    TA02[TA02 MITM Eavesdropper]:::threatActor
    aiAPI[External AI Model API]:::system
  end

  ide -- code pushes/pulls --> gitServer
  ide -- "Code suggestions" --> aiAPI
  TA01 -. Distributes trojan plugin .-> ide
  TA02 -. Sniff/Inject traffic .- ide
  C03 -. monitors .- codeA01
  ide -- "TLS (C02)" --- aiAPI
```
