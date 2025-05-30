# Field Operations Support Agent

A MuleSoft application that provides an AI-powered conversational assistant for utility field technicians using Claude AI through MCP (Model Control Protocol) and MAC (MuleSoft AI Connector) connectors.

## Overview

This application enables field technicians to access critical operational information through natural language conversations with Claude AI, including:

- Standard Operating Procedures (SOPs) and Technical Manuals
- Asset Maintenance History
- Parts and Inventory Information  
- Work Order Details

## Features

### MVP Features (User Stories)

**F1: SOP & Technical Manual Access**
- Tool: `getSopManual`
- Access procedures for equipment replacement, maintenance, and safety guidelines
- Supports transformers, fuses, meters, and general equipment

**F2: Asset Maintenance History Lookup**
- Tool: `getAssetHistory`
- Query maintenance records and known issues by asset ID or location
- Understand past problems for better diagnosis

**F3: Parts & Inventory Check**
- Tool: `checkPartsInventory`
- Check parts availability across warehouses and mobile inventory
- Find compatible alternatives for specific parts

**F4: Work Order Information Retrieval**
- Tool: `getWorkOrderInfo`
- Access work order details and customer information
- Get technician assignments and job specifications

## Technical Stack

- **Runtime**: Mule 4.9.4
- **AI Integration**: Claude AI via MAC Connector
- **MCP Integration**: MuleSoft MCP Connector for tool orchestration
- **Backend Integration**: Database connectors for asset, inventory, and work order systems
- **Communication Protocol**: MCP over Server-Sent Events (SSE)

## Prerequisites

- MuleSoft Anypoint Studio or Anypoint Runtime 4.9.4+
- Claude AI API access
- Database access for backend systems (PostgreSQL recommended)
- MCP-compatible client for Claude integration

## Configuration

### Required Properties

Configure these properties in `src/main/resources/config.properties`:

```properties
# Claude AI Configuration
claude.api.key=${secure::claude.api.key}
claude.model=claude-3-sonnet-20240229

# Database Configurations
asset.db.url=jdbc:postgresql://localhost:5432/asset_management
asset.db.user=${secure::asset.db.user}
asset.db.password=${secure::asset.db.password}

inventory.db.url=jdbc:postgresql://localhost:5432/inventory_management
inventory.db.user=${secure::inventory.db.user}
inventory.db.password=${secure::inventory.db.password}
```

### Secure Properties

Create a `secure.properties` file (not included in repository) with:

```properties
claude.api.key=your-claude-api-key
asset.db.user=your-db-username
asset.db.password=your-db-password
inventory.db.user=your-inventory-db-username
inventory.db.password=your-inventory-db-password
```

## Installation and Deployment

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/field-operations-support-agent.git
   ```

2. **Import into Anypoint Studio**
   - File → Import → Anypoint Studio Project from File System
   - Select the project directory

3. **Configure properties**
   - Update `config.properties` with your environment settings
   - Create `secure.properties` with sensitive credentials

4. **Deploy**
   - Right-click project → Run As → Mule Application
   - Or deploy to CloudHub/On-premises runtime

## Usage

1. **Start the application**
   - The MCP server will be available on `http://localhost:8082`

2. **Connect Claude client**
   - Configure Claude client to connect to the MCP server endpoint
   - Initialize a new session

3. **Use natural language queries**
   ```
   "What's the procedure for replacing a transformer fuse?"
   "Show me maintenance history for asset TRANS-001"
   "Do we have part number XYZ-123 in stock?"
   "What are my assigned work orders for today?"
   ```

## Project Structure

```
field-operations-support-agent/
├── src/
│   ├── main/
│   │   ├── mule/
│   │   │   ├── field-operations-support-agent.xml  # Main flows
│   │   │   └── subflows.xml                        # Supporting logic
│   │   └── resources/
│   │       ├── config.properties                   # Configuration
│   │       ├── application-types.xml               # DataWeave types
│   │       └── log4j2.xml                         # Logging config
│   └── test/
├── exchange-docs/
│   └── home.md                                     # API documentation
├── pom.xml                                         # Maven configuration
└── mule-artifact.json                             # Mule application metadata
```

## API Tools

The application exposes the following MCP tools:

- `getSopManual` - Retrieve SOPs and technical manuals
- `getAssetHistory` - Get asset maintenance history
- `checkPartsInventory` - Check parts availability
- `getWorkOrderInfo` - Retrieve work order information

## Safety and Compliance

All procedures returned include:
- Safety precautions and warnings
- Required tools and equipment
- Estimated completion times
- Proper escalation procedures for complex situations

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

[Specify your license here]

## Support

For issues and questions:
- Check the documentation in `exchange-docs/`
- Review logs in the `logs/` directory
- Contact the development team

## Version History

- **1.0.0-SNAPSHOT** - Initial MVP implementation with core features