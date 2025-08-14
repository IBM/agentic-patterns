

## Scenario:
In a microservices architecture, MCP is used as a mesh router between services, translating tool invocations from one microservice to another. Policies for access (e.g., namespace-level RBAC) are enforced at each MCP edge.

However, one MCP node is misconfigured to trust upstream requests and skips policy evaluation, assuming enforcement was done at the source. This opens the door to lateral movement, where a compromised service can issue tool calls across services without proper checks.

## Threat Landscape:
Trust between internal services is often implicit in mesh networks. A misconfigured node can allow cross-namespace access, effectively bypassing RBAC or ABAC policies due to trust downgrade. It’s a classic case of confused internal trust.

## Assets (A):
* A01: Microservice data in protected namespaces.
* A02: Policy configurations enforced at edge.
* A03: Service account identities.

## Threat Actors (TA):
* TA01: Compromised service instance making unauthorized tool calls.
* TA02: Internal attacker moving laterally.

## Security Controls (C):
* C01: Enforce policies on every MCP node, not just ingress.
* C02: Include identity headers and verify authenticity at each hop.
* C03: Use signed JWTs or mTLS to confirm tool call provenance.

## Zones:
* Internal Services (microservices)
* MCP Mesh (per-node policy enforcement)
* Data Stores (final target)

```mermaid
flowchart TD
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph svc1 [Compromised Service]
    TA01[TA01: Internal Compromised Node]:::threatActor
  end

  subgraph mcp1 [MCP Node A with Proper Policy]
    mcpNodeA[MCP A]:::system
    C01[[C01 Local Policy Enforcement]]:::control
  end

  subgraph mcp2 [Misconfigured MCP Node B]
    mcpNodeB[MCP B - Trusts Upstream]:::system
  end

  subgraph target [Protected Service]
    A01Data[A01: Service Data]:::asset
  end

  TA01 --> mcpNodeA --> mcpNodeB --> A01Data
  C01 -. ensures checks .-> mcpNodeA
```