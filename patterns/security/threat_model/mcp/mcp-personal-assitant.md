## Scenario: 
An end-user runs a personal AI assistant on their laptop, connected to various MCP servers for personal services – e.g., an “Email” MCP server for Gmail, a “Drive” server for cloud storage. The MCP servers are authorized via OAuth and store tokens to act on behalf of the user. The AI can retrieve emails, files, and perform actions such as sending messages or deleting files in response to a user’s natural language requests.

## Threat Landscape: 
This convenience creates a “keys to the kingdom” situation. If an attacker steals the OAuth tokens stored on the user’s MCP server, they can impersonate the user’s accounts without logging in (bypassing usual security alerts). Malicious prompts or data could trick the AI into leaking sensitive content (prompt injection). A rogue MCP server or malware on the device could completely hijack the assistant’s capabilities. The assistant’s broad access – emails, contacts, drive files – means any compromise can lead to extensive data exposure and unauthorized actions (like fraudulent emails or file deletions).
## Assets (A):
* A01: Personal data stores (emails, documents in cloud drive, contacts) accessible via MCP connectors – containing private and potentially sensitive information.
* A02: OAuth tokens/credentials for services (stored by MCP servers to access Gmail, Drive, etc.)

.
## Threat Actors (TA):
* TA01: Remote attacker (token thief) – steals saved tokens or config (e.g., via malware or device breach) and uses them to spin up their own MCP client, thus accessing the victim’s accounts silently

.
* TA02: Malicious content provider – sends the user a crafted email or file containing hidden instructions that the AI will interpret (prompt injection), causing unintended data exfiltration or actions.
## Security Controls (C):
* C01: Secure credential storage – encrypt and isolate OAuth tokens so that malware or untrusted processes cannot easily steal them (e.g., hardware-backed key storage).
* C02: Continuous authentication & token scope limits – use short-lived tokens and least privilege (e.g., read-only where possible) to minimize damage from token theft, and re-prompt for authorization on sensitive actions.
* C03: Content safety filters – the AI assistant should validate or sanitize inputs from emails/files, detecting hidden instructions or anomalies before executing them.
* C04: Activity monitoring – alert or require user confirmation if large-scale or unusual actions are initiated (e.g., mass email forwarding or bulk file deletion).
## Zones:
* User Device (personal computer running AI + MCP servers – semi-trusted environment)
* Internet Services (Google/Microsoft cloud for email, drive – external zone)
* (No separate LAN in this use case, everything occurs on the user device and the cloud)

```mermaid
flowchart LR
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph zone_user [User's Machine]
    style zone_user stroke:#ff0000,stroke-dasharray: 5 5;
    assistant[AI Assistant - MCP Client]:::system
    emailServer[MCP Server: Email Access]:::system
    driveServer[MCP Server: Drive Access]:::system
    tokensA02["A02: Stored OAuth Tokens"]:::asset
    C01[[C01 Secure Vault]]:::control
    C03[[C03 Content Filter]]:::control
  end
  subgraph zone_cloud [Internet Cloud Services]
    style zone_cloud stroke:#ff0000,stroke-dasharray: 5 5;
    gmail[Email Account]:::asset
    gdrive[Cloud Drive]:::asset
    TA02[TA02 Malicious Email/File Sender]:::threatActor
  end
  TA01[TA01 Attacker]:::threatActor

  gmail -- OAuth -- emailServer
  gdrive -- OAuth -- driveServer
  assistant -- "fetch/send data" --> emailServer
  assistant -- "fetch/upload files" --> driveServer
  emailServer -- API calls --> gmail
  driveServer -- API calls --> gdrive
  TA02 -. Sends poisoned content .-> gmail
  TA01 -. Steals A02 .- C01
  C01 -. protects .- tokensA02
  assistant -. checks .- C03
```

## References

1. https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp
2. https://techcommunity.microsoft.com/blog/microsoftdefendercloudblog/plug-play-and-prey-the-security-risks-of-the-model-context-protocol/4