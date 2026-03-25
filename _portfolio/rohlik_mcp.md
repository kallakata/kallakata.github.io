---
title: "Rohlik MCP"
excerpt: "A Python-based terminal application that connects to a Rohlik MCP (Model Context Protocol) server and enables natural language"
layout: page
collection: portfolio
---

[Rohlik MCP](https://github.com/kallakata/rohlik_mcp)

**Forked from original [Rohlík MCP project](https://github.com/tomaspavlin/rohlik-mcp) by Tomas Pavlin**

---

# Rohlik MCP Terminal Client

A Python-based terminal application that connects to a Rohlik MCP (Model Context Protocol) server and enables natural language

**Example:**

````
You: Analyze my orders from the last 3 months and show me the top 5 productsteractions with Rohlik.cz grocery shopping services through Claude AI.

## Overview

This terminal client demonstrates how to:
- Connect to an MCP server via stdio
- Integrate MCP tools with Claude AI for conversational interactions
- Manage shopping cart, search products, view orders, and analyze purchase history
- Use the Model Context Protocol for tool-based AI interactions

## Features

- **Shopping Management**: Search products, manage cart, view shopping lists
- **Order Analytics**: Analyze purchase history, track spending patterns, find most-ordered products
- **Delivery Information**: Check delivery slots, upcoming orders, and delivery status
- **Natural Language Interface**: Interact with Rohlik services using conversational AI
- **MCP Integration**: Seamless connection to local or published MCP servers

## Prerequisites

- Python 3.10 or higher
- Anthropic API key (Claude)
- Rohlik.cz account credentials
- Node.js 18+ (for running the MCP server)

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/kallakata/rohlik-mcp.git
   cd rohlik_mcp
````

2. **Create a virtual environment (recommended):**

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies:**

   ```bash
   pip install anthropic mcp python-dotenv
   ```

4. **Create a `.env` file:**

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and add your credentials:

   ```properties
   ANTHROPIC_API_KEY=your-anthropic-api-key-here
   ROHLIK_USERNAME=your-rohlik-email@example.com
   ROHLIK_PASSWORD=your-rohlik-password
   ROHLIK_BASE_URL=https://www.rohlik.cz
   ```

## Usage

### Running with Local MCP Server

If you have the Rohlik MCP server running locally:

```bash
python3 rohlik_terminal.py
```

The client will automatically connect to your local MCP server at `/Users/p.varenka/Repos/rohlik-mcp-integ/dist/index.js`.

### Running with Published MCP Server

To use the published npm package instead, edit `rohlik_terminal.py` and change the connection parameters:

```python
await terminal.connect_mcp_server(
    command="npx",
    args=["-y", "@tomaspavlin/rohlik-mcp"],
    env={...}
)
```

### Example Interactions

Once the terminal is running, you can ask questions like:

- **Product Search:**

  - "Can you find me some milk products?"
  - "Search for organic vegetables"

- **Cart Management:**

  - "What's in my cart?"
  - "Add product ID 12345 to my cart"
  - "Remove item from my cart"

- **Order Analytics:**

  - "Show me my orders from the last 2 months"
  - "What are my most frequently ordered products?"
  - "How much did I spend on groceries last month?"
  - "Analyze my purchase patterns over the last 6 months"

- **Order Details:**

  - "Show me details of order 1110449889"
  - "What did I order last time?"

- **Delivery Information:**

  - "When is my next delivery?"
  - "What delivery slots are available?"
  - "Show me my upcoming orders"

- **Account Information:**
  - "Show me my account summary"
  - "What's my premium status?"
  - "How many reusable bags do I have?"

## Available MCP Tools

The Rohlik MCP server provides the following tools:

### Shopping & Cart

- `search_products` - Search for products on Rohlik.cz by name
- `add_to_cart` - Add products to the shopping cart
- `get_cart_content` - Get current shopping cart contents
- `remove_from_cart` - Remove items from the cart
- `get_shopping_list` - Get a shopping list by ID

### Orders & Analytics

- `get_order_history` - Get past delivered orders
- `get_order_detail` - Get detailed information about a specific order
- `analyze_orders_by_months` - **NEW!** Comprehensive order analytics including:
  - Most ordered products
  - Top categories and brands by spending
  - Monthly spending breakdown
  - Total spending and average order value
- `get_upcoming_orders` - Get scheduled upcoming orders

### Delivery

- `get_delivery_info` - Get current delivery information
- `get_delivery_slots` - Get available delivery time slots

### Account

- `get_account_data` - Get comprehensive account information
- `get_premium_info` - Get Rohlik Premium subscription details
- `get_announcements` - Get current announcements
- `get_reusable_bags_info` - Get reusable bags information

## Order Analytics Tool

The `analyze_orders_by_months` tool provides powerful insights into your shopping habits:

**Parameters:**

- `months` (1-24, default: 6) - Number of months to analyze
- `maxOrders` (10-500, default: 100) - Maximum orders to fetch
- `topCount` (5-50, default: 10) - Number of top items to show

**Example:**

```
You: Analyze my orders from the last 3 months and show me the top 5 products
```

