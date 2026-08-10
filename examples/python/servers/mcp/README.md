# MCP Server with x402 Paid Tools

This example demonstrates how to create an MCP (Model Context Protocol) server with paid tools using the x402 payment protocol.

## Overview

The server exposes:

- **get_weather** (paid) - Returns weather data for a city, costs $0.001
- **ping** (free) - Returns "pong", no payment required

## Setup

1. Copy `.env-local` to `.env` and fill in your values:

```bash
cp .env-local .env
```

2. Configure environment variables:

   - `EVM_ADDRESS`: Your EVM wallet address to receive payments
   - `FACILITATOR_URL`: x402 facilitator URL (default: https://x402.org/facilitator)
   - `PORT`: Server port (default: 4022)

3. Install dependencies:

```bash
uv sync
```

## Running

All examples use the `create_payment_wrapper` API - a lean, functional approach to adding payments to MCP tools.

### Simple Mode (Recommended Starting Point)

Basic payment wrapper usage:

```bash
python main.py simple
```

Simple mode demonstrates:

- Creating a payment wrapper with shared configuration
- Wrapping tool handlers with payment logic
- Using native tool registration API
- Mixing paid and free tools

### Advanced Mode (Production Features)

Payment wrapper with hooks for monitoring and control:

```bash
python main.py advanced
```

Advanced mode demonstrates:

- Server-side hooks for production features
- `on_before_execution`: Rate limiting, access control (can abort)
- `on_after_execution`: Logging, metrics collection
- `on_after_settlement`: Receipts, notifications

### Existing Server Mode

Adding payment to an existing MCP server:

```bash
python main.py existing
```

Existing server mode demonstrates:

- Integrating x402 with an existing MCP server instance
- Minimal code changes to add payment
- Creating reusable payment wrappers

The server will start on `http://localhost:4022` with:

- SSE endpoint: `GET /sse`
- Messages endpoint: `POST /messages`
- Health check: `GET /health`

## Testing

Use the MCP client example (`examples/python/clients/mcp`) to connect and test:

```bash
cd ../../clients/mcp
python main.py simple
```

## Payment Flow

1. Client calls `get_weather` tool
2. Server returns 402 error with `PaymentRequired` data
3. Client creates payment payload and retries
4. Server verifies payment, executes tool, returns result
5. Payment is settled and receipt included in response

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   MCP Client    │────▶│    MCPServer     │────▶│  x402Facilitator │
│  (with wallet)  │     │ + PaymentWrapper │     │  (verification)  │
│                 │◀────│  (paid tools)    │◀────│  (settlement)    │
└─────────────────┘     └──────────────────┘     └──────────────────┘
```

The payment wrapper is a lightweight function that adds payment verification and settlement to individual tool handlers.


---

## Real-World Examples

### AgentBridge Atlas — Machine Commerce for China-Facing AI Agents

A production MCP server providing China-facing digital capabilities via x402 payments on Base chain. No API keys, no subscriptions — agents pay per call and get a result.

- **Repository**: [tianzizhiming-svg/agentbridge](https://github.com/tianzizhiming-svg/agentbridge)
- **Live API**: [api.060504.shop](https://api.060504.shop)
- **What it demonstrates**:
  - Two capability categories: `raw-content` (unverified, machine-fetched, $0.003–$0.008) and `verified-analysis` (human-authored, decision-grade, $90)
  - Bazaar discovery extension with dynamic route parameters
  - OpenAPI + MCP + x402 triple integration
  - Production deployment with real payment settlement on Base mainnet
- **MCP tools**: `discover_capabilities`, `get_capability_info`, `purchase_capability`
