# Crypto Trader MCP Tool - AI Coding Instructions

## Project Overview
This is an **MCP (Model Context Protocol) server** that provides cryptocurrency market data using the CoinGecko API. It's designed to integrate with Claude Desktop and follows the FastMCP framework pattern.

## Architecture & Key Components

### Core Structure
- **`main.py`**: Single-file MCP server containing all tools and business logic
- **FastMCP Framework**: Uses `@mcp.tool()` decorators to expose functions as MCP tools
- **CoinGecko API Integration**: All tools interact with the free CoinGecko API via `pycoingecko` library
- **Async/Await Pattern**: All tool functions are async and return Dict[str, Any]

### Tool Design Pattern
Each tool follows this consistent structure:
```python
@mcp.tool("tool_name")
async def tool_function(param: str) -> Dict[str, Any]:
    try:
        # 1. Input validation and normalization
        symbol = symbol.lower().replace('-usd', '')
        
        # 2. Symbol-to-ID resolution (for most tools)
        coins_list = cg.get_coins_list()
        coin_id = next((coin['id'] for coin in coins_list if coin['symbol'].lower() == symbol), None)
        
        # 3. API call to CoinGecko
        data = cg.some_api_method(coin_id)
        
        # 4. Data transformation and return
        return {"formatted": "response"}
    except Exception as e:
        return {"error": f"Failed message: {str(e)}"}
```

## Critical Development Patterns

### Symbol Resolution Logic
Most tools require converting crypto symbols (BTC, ETH) to CoinGecko IDs (bitcoin, ethereum):
- Always lowercase symbols and strip `-usd` suffixes
- Use `cg.get_coins_list()` to find matching coin_id by symbol
- Return error if symbol not found before making API calls

### Error Handling Convention
- Always wrap tool logic in try/except
- Return `{"error": "message"}` for failures
- Log errors using the configured logger
- Never let exceptions bubble up to MCP framework

### Data Transformation Standards
- Uppercase symbols in responses for consistency
- Include timestamps using `datetime.now().isoformat()`
- Convert Unix timestamps to ISO format: `datetime.fromtimestamp(timestamp/1000)`
- Limit result sets (e.g., 25 search results) to prevent overwhelming responses

## Development Workflow

### Running & Testing
```bash
# Install dependencies
pip install -r requirements.txt

# Run the MCP server directly
py -3.13 main.py

# For Claude Desktop integration, configure MCP settings:
# Path format: "C:\\Full\\Path\\To\\main.py"
```

### Adding New Tools
1. Define async function with type hints
2. Add `@mcp.tool("tool_name")` decorator
3. Follow the error handling pattern
4. Test with sample queries via Claude Desktop
5. Update README.md with tool documentation

### API Rate Limiting
- CoinGecko free tier has rate limits
- No caching implemented - each tool call hits the API
- Consider adding delays or caching for production use

## Integration Points

### Claude Desktop Configuration
Tools are exposed to Claude Desktop via MCP protocol. Configuration requires:
- Absolute path to `main.py` in MCP settings
- Python executable that can import required packages
- Server must remain running during Claude Desktop usage

### External Dependencies
- **pycoingecko**: Primary API client, handles all CoinGecko interactions
- **pandas**: Currently imported but not actively used (consider removing)
- **FastMCP**: Core framework for MCP server functionality

## File-Specific Notes

### `main.py`
- Contains all 6 cryptocurrency tools in a single file
- Global `cg = CoinGeckoAPI()` client instance
- Each tool is self-contained with its own error handling
- Uses standard logging configuration

### `requirements.txt`
- Minimal dependencies for MCP server functionality
- pandas imported but not used in current implementation
- Specific version constraints ensure compatibility

### `README.md`
- Comprehensive tool documentation with JSON examples
- Claude Desktop integration instructions
- Sample queries that work with the implemented tools
