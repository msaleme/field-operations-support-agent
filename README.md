# Utility Field Operations Support Agent

A MuleSoft demonstration application for giving utility field technicians conversational access to approved procedures, asset history, parts availability, and work-order context.

> **Project status:** Demonstration implementation. It is not a production safety system and must not replace approved procedures, dispatch authority, switching orders, or qualified human judgment.

## Use case

Field personnel frequently move between work management, asset, inventory, document, and customer systems. This project demonstrates how an AI assistant can retrieve governed context through explicit tools while keeping source systems behind MuleSoft APIs.

## Supported tools

| Tool | Purpose |
|---|---|
| `getSopManual` | Retrieve approved procedure or technical-manual context |
| `getAssetHistory` | Review asset maintenance and known-issue history |
| `checkPartsInventory` | Check availability and approved alternatives |
| `getWorkOrderInfo` | Retrieve assignment and job context |

## Architecture

- Mule runtime provides the integration and policy boundary.
- MCP exposes narrowly scoped tools to the assistant.
- Source adapters connect to document, asset, inventory, and work-management systems.
- Correlation identifiers and audit events should connect the request, retrieved evidence, assistant response, and resulting human action.

## Safety and governance boundaries

Production implementations should:

- retrieve only approved and current procedures;
- display source, version, owner and effective date;
- require human confirmation for consequential actions;
- prevent the assistant from issuing switching or safety authority;
- protect customer, workforce and critical-infrastructure information;
- log tool use without exposing credentials or unnecessary personal data;
- fail safely when sources are unavailable or contradictory.

## Technology

- Mule 4
- MuleSoft AI and MCP connectors
- database or API adapters for enterprise systems
- MCP-compatible assistant client

## What's actually included

This repository is a Mule 4 application, not just documentation:

```text
src/main/mule/field-operations-support-agent.xml   Main flow: MCP server listener and tool routing
src/main/mule/subflows.xml                         Supporting sub-flows for the four tools
src/main/mule/global-config.xml                    Connector and global configuration
src/main/mule/error-handlers.xml                   Shared error-handling logic
src/main/resources/application-types.xml           DataWeave type definitions
src/main/resources/config.properties               Externalized configuration
src/main/resources/log4j2.xml                      Logging configuration
src/test/munit/field-operations-support-agent-test.xml   MUnit test suite containing 8 test cases
exchange-docs/home.md                              Anypoint Exchange asset documentation
architecture-diagram.md                            Architecture overview
pom.xml / mule-artifact.json                       Maven build and Mule application metadata
```

The application exposes four MCP tools to the assistant: `getSopManual`, `getAssetHistory`, `checkPartsInventory`, and `getWorkOrderInfo`.

## Getting started

Review the Mule configuration and replace example endpoints and credentials with environment-managed configuration. Never commit real credentials. Connect only non-production data until security and operational reviews are complete.

## Portfolio context

This repository is part of a broader [utility grid-modernization portfolio](https://github.com/msaleme/utility-ai-mulesoft-api/blob/master/docs/portfolio-guide.md) covering grid intelligence, field operations, smart meters, customer programs, compliance, and governed AI-assisted operations.


## Related projects

- [Utility AI Semantic Layer](https://github.com/msaleme/utility-ai-mulesoft-api)
- [Utility Customer Service Process API](https://github.com/msaleme/utility-customer-service-process-api)
- [Vegetation Management Process API](https://github.com/msaleme/vegetation-management-process-api)

## License

Add an explicit license before redistribution or production reuse.
