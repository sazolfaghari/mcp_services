# Open-Meteo Weather MCP Server

A Python-based Model Context Protocol (MCP) server built with FastMCP that provides weather data tools using the Open-Meteo API. This server enables AI agents and language models to access current and forecasted weather information through standardized MCP tools.

## Overview

This MCP server provides tools to:
- Get current weather conditions for any location using coordinates
- Retrieve detailed weather forecasts with daily resolution
- Access weather data using the free Open-Meteo API with proper error handling

## Features

- **Current Weather**: Get real-time weather conditions including temperature, precipitation, wind speed, and weather codes
- **Weather Forecasts**: Access detailed daily weather forecasts with max/min temperatures, precipitation, and weather codes
- **Coordinate-Based Queries**: Query weather using precise latitude and longitude coordinates
- **FastMCP Framework**: Built using the modern FastMCP framework for streamlined MCP server development
- **Robust Error Handling**: Comprehensive error handling with proper HTTP timeouts and fallback responses
- **No API Key Required**: Uses the free Open-Meteo API service
- **MCP Compatible**: Fully compliant with Model Context Protocol standards

## Prerequisites

- Python 3.8 or higher
- pip package manager
- Git (for cloning the repository)

## Installation

1. **Clone the Repository**
   ```bash
   git clone https://github.com/sazolfaghari/mcp_services.git/
   cd mcp_services/mcp-server-weather
   ```

2. **Create Virtual Environment** (Recommended)
   ```bash
   python -m venv .venv
   ```

3. **Install Dependencies**
   ```bash
   pip install uv
   pip install fastmcp httpx 
   # Download and install nvm:
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
   
   # in lieu of restarting the shell
   \. "$HOME/.nvm/nvm.sh"
   
   # Download and install Node.js:
   nvm install 22
   
   # Verify the Node.js version:
   node -v # Should print "v22.18.0".
   nvm current # Should print "v22.18.0".
   
   # Verify npm version:
   npm -v # Should print "10.9.3".

   npx @modelcontextprotocol/inspector node build/index.js
   ```
## Usage

### Development Mode with MCP Inspector

To test the server during development:

1. Start the virtual environment.
```bash
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv init
uv add "mcp[cli]" httpx
```
NOTE: To stop the virtual environment:
```bash
 deactivate
```
2. Run MCP server in dev mode with the [MCP Inspector](https://github.com/modelcontextprotocol/inspector):
```bash
mcp dev server.py
```

### Running the Server Directly

You can also run the server directly:

```bash
python server.py
```

The server will start and listen for MCP protocol messages over stdio transport.

### Production Usage with Claude Desktop

1. **Configure Claude Desktop**
   
   Add the server configuration to your Claude Desktop settings file:
   
   **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
   
   **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

2. **Find the full path to `uv`**
   
   MacOS / Linux:
   ```
   which uv
   ```

   Windows:
   ```
   where uv
   ```
3. **In `claude_desktop_config.js`**

   MacOS/Linux:
   
   ```json
     {
     "mcpServers": {
       "weather": {
         "command": "uv",
         "args": [
           "--directory",
           "/ABSOLUTE/PATH/TO/PARENT/FOLDER/",
           "run",
           "server.py"
         ]
       }
     }
   }
```

Windows:

   ```json
   {
     "mcpServers": {
       "weather": {
         "command": "uv",
         "args": [
           "--directory",
           "C:\\ABSOLUTE\\PATH\\TO\\PARENT\\FOLDER\\",
           "run",
           "server.py"
         ]
       }
     }
   }
```

4. **Reboot Claude Desktop and use a prompt that will trigger your MCP.**


### Usage with Cursor

Configure the server in Cursor's MCP settings following similar steps as Claude Desktop.

## Available Tools

The server provides the following MCP tools:

### `get_current_weather`
Get current weather conditions for a specified location.

**Parameters:**
- `latitude` (float): Latitude coordinate of the location
- `longitude` (float): Longitude coordinate of the location

**Returns:**
- Current temperature in Celsius
- Precipitation amount in mm
- Weather code (numerical representation)
- Wind speed at 10m height
- Timestamp of the data

**Example:**
```
What's the current weather at coordinates 40.7128, -74.0060? (New York City)
```

### `get_forecast`
Get weather forecast for a specified location.

**Parameters:**
- `latitude` (float): Latitude coordinate of the location
- `longitude` (float): Longitude coordinate of the location

**Returns:**
- Daily forecast data including:
  - Date
  - Maximum temperature (°C)
  - Minimum temperature (°C)
  - Precipitation sum (mm)
  - Weather code
- Covers multiple days (typically 7 days)

**Example:**
```
Give me a weather forecast for coordinates 51.5074, -0.1278 (London).
```

## API Integration

This server uses the [Open-Meteo API](https://open-meteo.com/), a free weather API that provides:

- **No API Key Required**: Free access without registration
- **Global Coverage**: Weather data for locations worldwide using coordinates
- **High Accuracy**: Based on multiple weather models
- **Real-time Data**: Updated weather information
- **Multiple Parameters**: Temperature, precipitation, wind speed, weather codes

### API Endpoints Used

- **Current Weather**: `/v1/forecast` with current weather parameters
- **Forecast**: `/v1/forecast` with hourly and daily forecast parameters

### Weather Codes

The API returns numerical weather codes that represent different weather conditions:
- 0: Clear sky
- 1-3: Partly cloudy variations
- 45-48: Fog variations
- 51-57: Drizzle variations
- 61-67: Rain variations
- 71-77: Snow variations
- 80-82: Rain showers
- 85-86: Snow showers
- 95-99: Thunderstorm variations

## Project Structure

```
mcp-server-weather/
├── server.py          # Main MCP server implementation
└── README.md                 # This documentation file
```

## Code Structure

The server is implemented in a single Python file (`server.py`) with the following key components:

### Core Components

1. **FastMCP Server Initialization**
   ```python
   from mcp.server.fastmcp import FastMCP
   mcp = FastMCP("weather")
   ```

2. **HTTP Client with Error Handling**
   ```python
   async def make_openmeteo_request(url: str) -> dict[str, Any] | None:
   ```

3. **MCP Tool Decorators**
   ```python
   @mcp.tool()
   async def get_current_weather(latitude: float, longitude: float) -> str:
   
   @mcp.tool()
   async def get_forecast(latitude: float, longitude: float) -> str:
   ```

4. **Stdio Transport**
   ```python
   mcp.run(transport='stdio')
   ```

## Error Handling

The server includes robust error handling for:
- **HTTP Request Failures**: Uses try-catch blocks with proper timeout (30 seconds)
- **Invalid Coordinates**: Handles malformed latitude/longitude values
- **Network Connectivity Issues**: Returns user-friendly error messages when API is unreachable
- **API Response Errors**: Validates API responses and handles missing data gracefully
- **JSON Parsing Errors**: Safely handles malformed API responses

Error responses are returned as descriptive strings that can be presented to users.

## Related Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)
- [FastMCP Framework](https://github.com/modelcontextprotocol/python-sdk)
- [Open-Meteo API Documentation](https://open-meteo.com/en/docs)
- [HTTPX Documentation](https://www.python-httpx.org/)
- [MCP Server Examples](https://github.com/modelcontextprotocol/servers)








