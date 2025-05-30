# Field Operations Support Agent

## Overview

The Field Operations Support Agent is an AI-powered conversational assistant designed to support utility field technicians with instant access to critical operational data and procedures. The agent leverages Claude AI through MuleSoft's MCP (Model Control Protocol) connector and MAC (MuleSoft AI Connector) to provide hands-free access to:

- Standard Operating Procedures (SOPs) and Technical Manuals
- Asset Maintenance History
- Parts and Inventory Information
- Work Order Details

## Features

### F1: SOP & Technical Manual Access
- **Tool Name**: `getSopManual`
- **Description**: Retrieves standard operating procedures and technical manuals for specific equipment types and procedures
- **Parameters**: 
  - `equipmentType` (required): Type of equipment (transformer, fuse, meter, pole)
  - `procedureType` (required): Type of procedure (replacement, installation, maintenance, safety)
  - `modelNumber` (optional): Specific model number or equipment identifier

### F2: Asset Maintenance History Lookup
- **Tool Name**: `getAssetHistory`
- **Description**: Retrieves maintenance history and known issues for specific assets
- **Parameters**:
  - `assetId` OR `location` (required): Asset identifier or location
  - `equipmentType` (optional): Filter by equipment type

### F3: Parts & Inventory Check
- **Tool Name**: `checkPartsInventory`
- **Description**: Checks parts availability and finds compatible alternatives
- **Parameters**:
  - `partNumber` OR `partDescription` (required): Part to search for
  - `location` (optional): Warehouse location or 'truck' for mobile inventory
  - `findAlternatives` (optional): Whether to search for compatible alternatives

### F4: Work Order Information Retrieval
- **Tool Name**: `getWorkOrderInfo`
- **Description**: Retrieves work order details and customer information
- **Parameters**:
  - `workOrderNumber` OR `technicianId` (required): Specific work order or technician assignments
  - `status` (optional): Filter by work order status

## Technical Architecture

- **Runtime**: Mule 4.9.4
- **Integration Layer**: MCP Server with SSE connection
- **AI Integration**: MAC Connector with Claude AI
- **Backend Integration**: Database connectors for asset, inventory, and work order systems
- **Communication**: HTTP listener for MCP protocol

## Configuration

### Required Properties
- `claude.api.key`: API key for Claude AI integration
- `asset.db.*`: Asset management database connection settings
- `inventory.db.*`: Inventory database connection settings
- Database connection details for backend systems

### HTTP Configuration
- Default port: 8082
- MCP server endpoint available for Claude client connection

## Usage

1. Connect Claude client to the MCP server endpoint
2. Initialize conversation session
3. Use natural language to invoke tools:
   - "What's the procedure for replacing a transformer fuse?"
   - "Show me maintenance history for asset TRANS-001"
   - "Do we have part number XYZ-123 in stock?"
   - "What are my assigned work orders for today?"

## Safety and Compliance

This agent is designed to support field technicians with information access while maintaining all safety protocols. All procedures returned include safety precautions and proper escalation paths for complex situations.

## Version

Version: 1.0.0-SNAPSHOT