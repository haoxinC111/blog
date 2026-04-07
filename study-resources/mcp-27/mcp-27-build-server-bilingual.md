# Build an MCP Server

**Source URL:** https://modelcontextprotocol.io/docs/develop/build-server  
**整理日期:** 2026-04-07

---

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-python)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-python)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* Python
* LLMs like Claude

- Python
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never write to stdout. Writing to stdout will corrupt the JSON-RPC messages and break your server. The `print()` function writes to stdout by default, but can be used safely with `file=sys.stderr`.

**对于基于 STDIO 的服务器：** 永远不要写入 stdout。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。`print()` 函数默认写入 stdout，但可以搭配 `file=sys.stderr` 安全使用。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use a logging library that writes to stderr or files.

- 使用写入 stderr 或文件的日志库。

### Quick Examples

### 快速示例

```python
import sys
import logging

# Bad (STDIO)
print("Processing request")

# Good (STDIO)
print("Processing request", file=sys.stderr)

# Good (STDIO)
logging.info("Processing request")
```

### System requirements

### 系统要求

* Python 3.10 or higher installed.
* You must use the Python MCP SDK 1.2.0 or higher.

- 已安装 Python 3.10 或更高版本。
- 必须使用 Python MCP SDK 1.2.0 或更高版本。

### Set up your environment

### 设置环境

First, let's install `uv` and set up our Python project and environment:

首先，让我们安装 `uv` 并设置 Python 项目及环境：

Make sure to restart your terminal afterwards to ensure that the `uv` command gets picked up.

安装后请务必重启终端，以确保 `uv` 命令可用。

Now, let's create and set up our project:

现在，让我们创建并设置项目：

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Importing packages and setting up the instance

### 导入包并设置实例

Add these to the top of your `weather.py`:

将以下内容添加到你的 `weather.py` 文件顶部：

```python
from typing import Any

import httpx
from mcp.server.fastmcp import FastMCP

# Initialize FastMCP server
mcp = FastMCP("weather")

# Constants
NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"
```

The FastMCP class uses Python type hints and docstrings to automatically generate tool definitions, making it easy to create and maintain MCP tools.

FastMCP 类利用 Python 的类型提示和文档字符串自动生成工具定义，从而让创建和维护 MCP 工具变得非常简单。

### Helper functions

### 辅助函数

Next, let's add our helper functions for querying and formatting the data from the National Weather Service API:

接下来，添加辅助函数，用于向美国国家气象局（NWS）API 查询数据并格式化结果：

```python
async def make_nws_request(url: str) -> dict[str, Any] | None:
    """Make a request to the NWS API with proper error handling."""
    headers = {"User-Agent": USER_AGENT, "Accept": "application/geo+json"}
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers=headers, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except Exception:
            return None


def format_alert(feature: dict) -> str:
    """Format an alert feature into a readable string."""
    props = feature["properties"]
    return f"""
Event: {props.get("event", "Unknown")}
Area: {props.get("areaDesc", "Unknown")}
Severity: {props.get("severity", "Unknown")}
Description: {props.get("description", "No description available")}
Instructions: {props.get("instruction", "No specific instructions provided")}
"""
```

### Implementing tool execution

### 实现工具执行

The tool execution handler is responsible for actually executing the logic of each tool. Let's add it:

工具执行处理器负责实际执行每个工具的逻辑。让我们添加它：

```python
@mcp.tool()
async def get_alerts(state: str) -> str:
    """Get weather alerts for a US state.

    Args:
        state: Two-letter US state code (e.g. CA, NY)
    """
    url = f"{NWS_API_BASE}/alerts/active/area/{state}"
    data = await make_nws_request(url)

    if not data or "features" not in data:
        return "Unable to fetch alerts or no alerts found."

    if not data["features"]:
        return "No active alerts for this state."

    alerts = [format_alert(feature) for feature in data["features"]]
    return "\n---\n".join(alerts)


@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """Get weather forecast for a location.

    Args:
        latitude: Latitude of the location
        longitude: Longitude of the location
    """
    # First get the forecast grid endpoint
    points_url = f"{NWS_API_BASE}/points/{latitude},{longitude}"
    points_data = await make_nws_request(points_url)

    if not points_data:
        return "Unable to fetch forecast data for this location."

    # Get the forecast URL from the points response
    forecast_url = points_data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    if not forecast_data:
        return "Unable to fetch detailed forecast."

    # Format the periods into a readable forecast
    periods = forecast_data["properties"]["periods"]
    forecasts = []
    for period in periods[:5]:  # Only show next 5 periods
        forecast = f"""
{period["name"]}:
Temperature: {period["temperature"]}°{period["temperatureUnit"]}
Wind: {period["windSpeed"]} {period["windDirection"]}
Forecast: {period["detailedForecast"]}
"""
        forecasts.append(forecast)

    return "\n---\n".join(forecasts)
```

### Running the server

### 运行服务器

Finally, let's initialize and run the server:

最后，初始化并运行服务器：

```python
def main():
    # Initialize and run the server
    mcp.run(transport="stdio")


if __name__ == "__main__":
    main()
```

Your server is complete! Run `uv run weather.py` to start the MCP server, which will listen for messages from MCP hosts.

你的服务器已经完成！运行 `uv run weather.py` 即可启动 MCP 服务器，它将监听来自 MCP 宿主（host）的消息。

Let's now test your server from an existing MCP host, Claude for Desktop.

现在让我们在现有的 MCP 宿主——Claude for Desktop 中测试你的服务器。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**

We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.

我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。如果文件不存在，请创建它。

For example, if you have [VS Code](https://code.visualstudio.com/) installed:

例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key. The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。

In this case, we'll add our single weather server like so:

在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. To launch it by running `uv --directory /ABSOLUTE/PATH/TO/PARENT/FOLDER/weather run weather.py`

1. 有一个名为 "weather" 的 MCP 服务器
2. 通过运行 `uv --directory /ABSOLUTE/PATH/TO/PARENT/FOLDER/weather run weather.py` 来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-typescript)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-typescript)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* TypeScript
* LLMs like Claude

- TypeScript
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `console.log()`, as it writes to standard output (stdout) by default. Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `console.log()`，因为它默认写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use `console.error()` which writes to stderr, or use a logging library that writes to stderr or files.

- 使用 `console.error()`（写入 stderr），或使用写入 stderr 或文件的日志库。

### Quick Examples

### 快速示例

```javascript
// Bad (STDIO)
console.log("Server started");

// Good (STDIO)
console.error("Server started"); // stderr is safe
```

### System requirements

### 系统要求

For TypeScript, make sure you have the latest version of Node installed.

对于 TypeScript，请确保你已安装最新版本的 Node。

### Set up your environment

### 设置环境

First, let's install Node.js and npm if you haven't already. You can download them from [nodejs.org](https://nodejs.org/).
Verify your Node.js installation:

首先，如果你还没有安装 Node.js 和 npm，请前往 [nodejs.org](https://nodejs.org/) 下载。
验证你的 Node.js 安装：

```
node --version
npm --version
```

For this tutorial, you'll need Node.js version 16 or higher.

本教程需要 Node.js 16 或更高版本。

Now, let's create and set up our project:

现在，让我们创建并设置项目：

Update your package.json to add type: "module" and a build script:

更新你的 package.json，添加 `type: "module"` 和一个构建脚本：

package.json

```json
{
  "type": "module",
  "bin": {
    "weather": "./build/index.js"
  },
  "scripts": {
    "build": "tsc && chmod 755 build/index.js"
  },
  "files": ["build"]
}
```

Create a `tsconfig.json` in the root of your project:

在项目的根目录创建 `tsconfig.json`：

tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./build",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Importing packages and setting up the instance

### 导入包并设置实例

Add these to the top of your `src/index.ts`:

将以下内容添加到你的 `src/index.ts` 文件顶部：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const NWS_API_BASE = "https://api.weather.gov";
const USER_AGENT = "weather-app/1.0";

// Create server instance
const server = new McpServer({
  name: "weather",
  version: "1.0.0",
});
```

### Helper functions

### 辅助函数

Next, let's add our helper functions for querying and formatting the data from the National Weather Service API:

接下来，添加辅助函数，用于向美国国家气象局 API 查询数据并格式化结果：

```typescript
// Helper function for making NWS API requests
async function makeNWSRequest<T>(url: string): Promise<T | null> {
  const headers = {
    "User-Agent": USER_AGENT,
    Accept: "application/geo+json",
  };

  try {
    const response = await fetch(url, { headers });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return (await response.json()) as T;
  } catch (error) {
    console.error("Error making NWS request:", error);
    return null;
  }
}

interface AlertFeature {
  properties: {
    event?: string;
    areaDesc?: string;
    severity?: string;
    status?: string;
    headline?: string;
  };
}

// Format alert data
function formatAlert(feature: AlertFeature): string {
  const props = feature.properties;
  return [
    `Event: ${props.event || "Unknown"}`,
    `Area: ${props.areaDesc || "Unknown"}`,
    `Severity: ${props.severity || "Unknown"}`,
    `Status: ${props.status || "Unknown"}`,
    `Headline: ${props.headline || "No headline"}`,
    "---",
  ].join("\n");
}

interface ForecastPeriod {
  name?: string;
  temperature?: number;
  temperatureUnit?: string;
  windSpeed?: string;
  windDirection?: string;
  shortForecast?: string;
}

interface AlertsResponse {
  features: AlertFeature[];
}

interface PointsResponse {
  properties: {
    forecast?: string;
  };
}

interface ForecastResponse {
  properties: {
    periods: ForecastPeriod[];
  };
}
```

### Implementing tool execution

### 实现工具执行

The tool execution handler is responsible for actually executing the logic of each tool. Let's add it:

工具执行处理器负责实际执行每个工具的逻辑。让我们添加它：

```typescript
// Register weather tools

server.registerTool(
  "get_alerts",
  {
    description: "Get weather alerts for a state",
    inputSchema: {
      state: z
        .string()
        .length(2)
        .describe("Two-letter state code (e.g. CA, NY)"),
    },
  },
  async ({ state }) => {
    const stateCode = state.toUpperCase();
    const alertsUrl = `${NWS_API_BASE}/alerts?area=${stateCode}`;
    const alertsData = await makeNWSRequest<AlertsResponse>(alertsUrl);

    if (!alertsData) {
      return {
        content: [
          {
            type: "text",
            text: "Failed to retrieve alerts data",
          },
        ],
      };
    }

    const features = alertsData.features || [];
    if (features.length === 0) {
      return {
        content: [
          {
            type: "text",
            text: `No active alerts for ${stateCode}`,
          },
        ],
      };
    }

    const formattedAlerts = features.map(formatAlert);
    const alertsText = `Active alerts for ${stateCode}:\n\n${formattedAlerts.join("\n")}`;

    return {
      content: [
        {
          type: "text",
          text: alertsText,
        },
      ],
    };
  },
);

server.registerTool(
  "get_forecast",
  {
    description: "Get weather forecast for a location",
    inputSchema: {
      latitude: z
        .number()
        .min(-90)
        .max(90)
        .describe("Latitude of the location"),
      longitude: z
        .number()
        .min(-180)
        .max(180)
        .describe("Longitude of the location"),
    },
  },
  async ({ latitude, longitude }) => {
    // Get grid point data
    const pointsUrl = `${NWS_API_BASE}/points/${latitude.toFixed(4)},${longitude.toFixed(4)}`;
    const pointsData = await makeNWSRequest<PointsResponse>(pointsUrl);

    if (!pointsData) {
      return {
        content: [
          {
            type: "text",
            text: `Failed to retrieve grid point data for coordinates: ${latitude}, ${longitude}. This location may not be supported by the NWS API (only US locations are supported).`,
          },
        ],
      };
    }

    const forecastUrl = pointsData.properties?.forecast;
    if (!forecastUrl) {
      return {
        content: [
          {
            type: "text",
            text: "Failed to get forecast URL from grid point data",
          },
        ],
      };
    }

    // Get forecast data
    const forecastData = await makeNWSRequest<ForecastResponse>(forecastUrl);
    if (!forecastData) {
      return {
        content: [
          {
            type: "text",
            text: "Failed to retrieve forecast data",
          },
        ],
      };
    }

    const periods = forecastData.properties?.periods || [];
    if (periods.length === 0) {
      return {
        content: [
          {
            type: "text",
            text: "No forecast periods available",
          },
        ],
      };
    }

    // Format forecast periods
    const formattedForecast = periods.map((period: ForecastPeriod) =>
      [
        `${period.name || "Unknown"}:`,
        `Temperature: ${period.temperature || "Unknown"}°${period.temperatureUnit || "F"}`,
        `Wind: ${period.windSpeed || "Unknown"} ${period.windDirection || ""}`,
        `${period.shortForecast || "No forecast available"}`,
        "---",
      ].join("\n"),
    );

    const forecastText = `Forecast for ${latitude}, ${longitude}:\n\n${formattedForecast.join("\n")}`;

    return {
      content: [
        {
          type: "text",
          text: forecastText,
        },
      ],
    };
  },
);
```

### Running the server

### 运行服务器

Finally, implement the main function to run the server:

最后，实现运行服务器的主函数：

```typescript
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Weather MCP Server running on stdio");
}

main().catch((error) => {
  console.error("Fatal error in main():", error);
  process.exit(1);
});
```

Make sure to run `npm run build` to build your server! This is a very important step in getting your server to connect.

请务必运行 `npm run build` 来构建你的服务器！这是让服务器成功连接的一个非常重要的步骤。

Let's now test your server from an existing MCP host, Claude for Desktop.

现在让我们在现有的 MCP 宿主——Claude for Desktop 中测试你的服务器。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**

We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.

我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。如果文件不存在，请创建它。

For example, if you have [VS Code](https://code.visualstudio.com/) installed:

例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key. The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。

In this case, we'll add our single weather server like so:

在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. Launch it by running `node /ABSOLUTE/PATH/TO/PARENT/FOLDER/weather/build/index.js`

1. 有一个名为 "weather" 的 MCP 服务器
2. 通过运行 `node /ABSOLUTE/PATH/TO/PARENT/FOLDER/weather/build/index.js` 来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/spring-projects/spring-ai-examples/tree/main/model-context-protocol/weather/starter-stdio-server)For more information, see the [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) reference documentation.
For manual MCP Server implementation, refer to the [MCP Server Java SDK documentation](https://java.sdk.modelcontextprotocol.io/).

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/spring-projects/spring-ai-examples/tree/main/model-context-protocol/weather/starter-stdio-server)更多信息请参阅 [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) 参考文档。
如需手动实现 MCP 服务器，请参阅 [MCP Server Java SDK 文档](https://java.sdk.modelcontextprotocol.io/)。

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* Java
* LLMs like Claude

- Java
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `System.out.println()` or `System.out.print()`, as they write to standard output (stdout). Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `System.out.println()` 或 `System.out.print()`，因为它们写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use a logging library that writes to stderr or files.
* Ensure any configured logging library will not write to stdout.

- 使用写入 stderr 或文件的日志库。
- 确保任何已配置的日志库不会写入 stdout。

### System requirements

### 系统要求

* Java 17 or higher installed.
* [Spring Boot 3.3.x](https://docs.spring.io/spring-boot/installing.html) or higher

- 已安装 Java 17 或更高版本。
- [Spring Boot 3.3.x](https://docs.spring.io/spring-boot/installing.html) 或更高版本。

### Set up your environment

### 设置环境

Use the [Spring Initializer](https://start.spring.io/) to bootstrap the project.You will need to add the following dependencies:

使用 [Spring Initializer](https://start.spring.io/) 来引导项目。你需要添加以下依赖：

Then configure your application by setting the application properties:

然后通过设置 application properties 来配置你的应用：

The [Server Configuration Properties](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html#_configuration_properties) documents all available properties.Now let's dive into building your server.

[服务器配置属性](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html#_configuration_properties)文档记录了所有可用属性。现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Weather Service

### Weather Service

Let's implement a [WeatherService.java](https://github.com/spring-projects/spring-ai-examples/blob/main/model-context-protocol/weather/starter-stdio-server/src/main/java/org/springframework/ai/mcp/sample/server/WeatherService.java) that uses a REST client to query the data from the National Weather Service API:

让我们实现一个 [WeatherService.java](https://github.com/spring-projects/spring-ai-examples/blob/main/model-context-protocol/weather/starter-stdio-server/src/main/java/org/springframework/ai/mcp/sample/server/WeatherService.java)，它使用 REST 客户端查询美国国家气象局 API 的数据：

```java
@Service
public class WeatherService {

	private final RestClient restClient;

	public WeatherService() {
		this.restClient = RestClient.builder()
			.baseUrl("https://api.weather.gov")
			.defaultHeader("Accept", "application/geo+json")
			.defaultHeader("User-Agent", "WeatherApiClient/1.0 (your@email.com)")
			.build();
	}

  @Tool(description = "Get weather forecast for a specific latitude/longitude")
  public String getWeatherForecastByLocation(
      double latitude,   // Latitude coordinate
      double longitude   // Longitude coordinate
  ) {
      // Returns detailed forecast including:
      // - Temperature and unit
      // - Wind speed and direction
      // - Detailed forecast description
  }

  @Tool(description = "Get weather alerts for a US state")
  public String getAlerts(
      @ToolParam(description = "Two-letter US state code (e.g. CA, NY)") String state
  ) {
      // Returns active alerts including:
      // - Event type
      // - Affected area
      // - Severity
      // - Description
      // - Safety instructions
  }

  // ......
}
```

The `@Service` annotation will auto-register the service in your application context.
The Spring AI `@Tool` annotation makes it easy to create and maintain MCP tools.The auto-configuration will automatically register these tools with the MCP server.

`@Service` 注解会自动将服务注册到你的应用上下文中。
Spring AI 的 `@Tool` 注解让创建和维护 MCP 工具变得非常简单。自动配置会自动将这些工具注册到 MCP 服务器。

### Create your Boot Application

### 创建你的 Boot 应用

```java
@SpringBootApplication
public class McpServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(McpServerApplication.class, args);
	}

	@Bean
	public ToolCallbackProvider weatherTools(WeatherService weatherService) {
		return  MethodToolCallbackProvider.builder().toolObjects(weatherService).build();
	}
}
```

Uses the `MethodToolCallbackProvider` utils to convert the `@Tools` into actionable callbacks used by the MCP server.

使用 `MethodToolCallbackProvider` 工具将 `@Tools` 转换为 MCP 服务器使用的可操作回调。

### Running the server

### 运行服务器

Finally, let's build the server:

最后，构建服务器：

```
./mvnw clean install
```

This will generate an `mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar` file within the `target` folder.Let's now test your server from an existing MCP host, Claude for Desktop.

这会在 `target` 文件夹中生成一个 `mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar` 文件。现在让我们在现有的 MCP 宿主——Claude for Desktop 中测试你的服务器。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed.
[You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**We'll need to configure Claude for Desktop for whichever MCP servers you want to use.
To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor.
Make sure to create the file if it doesn't exist.For example, if you have [VS Code](https://code.visualstudio.com/) installed:

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。
为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。
如果文件不存在，请创建它。例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key.
The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.In this case, we'll add our single weather server like so:

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "my-weather-server"
2. To launch it by running `java -jar /ABSOLUTE/PATH/TO/PARENT/FOLDER/mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar`

1. 有一个名为 "my-weather-server" 的 MCP 服务器
2. 通过运行 `java -jar /ABSOLUTE/PATH/TO/PARENT/FOLDER/mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar` 来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

## Testing your server with Java client

## 使用 Java 客户端测试服务器

### Create an MCP Client manually

### 手动创建 MCP 客户端

Use the `McpClient` to connect to the server:

使用 `McpClient` 连接到服务器：

```java
var stdioParams = ServerParameters.builder("java")
  .args("-jar", "/ABSOLUTE/PATH/TO/PARENT/FOLDER/mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar")
  .build();

var stdioTransport = new StdioClientTransport(stdioParams);

var mcpClient = McpClient.sync(stdioTransport).build();

mcpClient.initialize();

ListToolsResult toolsList = mcpClient.listTools();

CallToolResult weather = mcpClient.callTool(
  new CallToolRequest("getWeatherForecastByLocation",
      Map.of("latitude", "47.6062", "longitude", "-122.3321")));

CallToolResult alert = mcpClient.callTool(
  new CallToolRequest("getAlerts", Map.of("state", "NY")));

mcpClient.closeGracefully();
```

### Use MCP Client Boot Starter

### 使用 MCP Client Boot Starter

Create a new boot starter application using the `spring-ai-starter-mcp-client` dependency:

使用 `spring-ai-starter-mcp-client` 依赖创建一个新的 boot starter 应用：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client</artifactId>
</dependency>
```

and set the `spring.ai.mcp.client.stdio.servers-configuration` property to point to your `claude_desktop_config.json`.
You can reuse the existing Anthropic Desktop configuration:

并将 `spring.ai.mcp.client.stdio.servers-configuration` 属性设置为指向你的 `claude_desktop_config.json`。
你可以复用现有的 Anthropic Desktop 配置：

```
spring.ai.mcp.client.stdio.servers-configuration=file:PATH/TO/claude_desktop_config.json
```

When you start your client application, the auto-configuration will automatically create MCP clients from the claude_desktop_config.json.For more information, see the [MCP Client Boot Starters](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-client-docs.html) reference documentation.

启动客户端应用时，自动配置会根据 claude_desktop_config.json 自动创建 MCP 客户端。更多信息请参阅 [MCP Client Boot Starters](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-client-docs.html) 参考文档。

## More Java MCP Server examples

## 更多 Java MCP 服务器示例

The [starter-webflux-server](https://github.com/spring-projects/spring-ai-examples/tree/main/model-context-protocol/weather/starter-webflux-server) demonstrates how to create an MCP server using SSE transport.
It showcases how to define and register MCP Tools, Resources, and Prompts, using the Spring Boot's auto-configuration capabilities.

[starter-webflux-server](https://github.com/spring-projects/spring-ai-examples/tree/main/model-context-protocol/weather/starter-webflux-server) 演示了如何使用 SSE 传输创建 MCP 服务器。
它展示了如何利用 Spring Boot 的自动配置能力来定义和注册 MCP 工具、资源和提示词。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/kotlin-sdk/tree/main/samples/weather-stdio-server)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/kotlin-sdk/tree/main/samples/weather-stdio-server)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* Kotlin
* LLMs like Claude

- Kotlin
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `println()`, as it writes to standard output (stdout) by default. Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `println()`，因为它默认写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use a logging library that writes to stderr or files.

- 使用写入 stderr 或文件的日志库。

### System requirements

### 系统要求

* JDK 11 or higher installed.

- 已安装 JDK 11 或更高版本。

### Set up your environment

### 设置环境

First, let's install `java` and `gradle` if you haven't already.
You can download `java` from [official Oracle JDK website](https://www.oracle.com/java/technologies/downloads/).
Verify your `java` installation:

首先，如果你还没有安装 `java` 和 `gradle`，请前往 [Oracle 官方 JDK 网站](https://www.oracle.com/java/technologies/downloads/) 下载 `java`。
验证你的 `java` 安装：

```
java --version
```

Now, let's create and set up your project:

现在，让我们创建并设置你的项目：

After running `gradle init`, select **Application** as the project type, **Kotlin** as the programming language.Alternatively, you can create a Kotlin application using the [IntelliJ IDEA project wizard](https://kotlinlang.org/docs/jvm-get-started.html).After creating the project, replace the contents of your `build.gradle.kts` with:

运行 `gradle init` 后，选择 **Application** 作为项目类型，**Kotlin** 作为编程语言。或者，你也可以使用 [IntelliJ IDEA 项目向导](https://kotlinlang.org/docs/jvm-get-started.html) 创建 Kotlin 应用。创建项目后，将你的 `build.gradle.kts` 内容替换为：

build.gradle.kts

```kotlin
// Check latest versions at https://github.com/modelcontextprotocol/kotlin-sdk/releases
val mcpVersion = "0.9.0"
val ktorVersion = "3.2.3"
val slf4jVersion = "2.0.17"

plugins {
    kotlin("jvm") version "2.3.20"
    kotlin("plugin.serialization") version "2.3.20"
    id("com.gradleup.shadow") version "8.3.9"
    application
}

application {
    mainClass.set("MainKt")
}

dependencies {
    implementation("io.modelcontextprotocol:kotlin-sdk:$mcpVersion")
    implementation("io.ktor:ktor-client-content-negotiation:$ktorVersion")
    implementation("io.ktor:ktor-serialization-kotlinx-json:$ktorVersion")
    implementation("io.ktor:ktor-client-cio:$ktorVersion")
    implementation("org.slf4j:slf4j-simple:$slf4jVersion")
}
```

Verify that everything is set up correctly:

验证所有设置是否正确：

```
./gradlew build
```

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Setting up the instance

### 设置实例

Add a server initialization function:

添加一个服务器初始化函数：

```kotlin
fun runMcpServer() {
    val server = Server(
        Implementation(
            name = "weather",
            version = "1.0.0",
        ),
        ServerOptions(
            capabilities = ServerCapabilities(tools = ServerCapabilities.Tools(listChanged = true)),
        ),
    )

    // register tools on server here

    val transport = StdioServerTransport(
        System.`in`.asInput(),
        System.out.asSink().buffered(),
    )

    runBlocking {
        val session = server.createSession(transport)
        val done = Job()
        session.onClose {
            done.complete()
        }
        done.join()
    }
}
```

### Weather API helper functions

### 天气 API 辅助函数

Next, let's add functions and data classes for querying and converting responses from the National Weather Service API:

接下来，让我们添加用于查询美国国家气象局 API 并将其响应进行转换的函数和数据类：

```kotlin
val httpClient = HttpClient(CIO) {
    defaultRequest {
        url("https://api.weather.gov")
        headers {
            append("Accept", "application/geo+json")
            append("User-Agent", "WeatherApiClient/1.0")
        }
        contentType(ContentType.Application.Json)
    }
    install(ContentNegotiation) {
        json(Json { ignoreUnknownKeys = true })
    }
}

// Extension function to fetch weather alerts for a given state
suspend fun HttpClient.getAlerts(state: String): List<String> {
    val alerts = this.get("/alerts/active/area/$state").body<AlertsResponse>()
    return alerts.features.map { feature ->
        """
            Event: ${feature.properties.event}
            Area: ${feature.properties.areaDesc}
            Severity: ${feature.properties.severity}
            Status: ${feature.properties.status}
            Headline: ${feature.properties.headline}
        """.trimIndent()
    }
}

// Extension function to fetch forecast information for given latitude and longitude
suspend fun HttpClient.getForecast(latitude: Double, longitude: Double): List<String> {
    val points = this.get("/points/$latitude,$longitude").body<PointsResponse>()
    val forecastUrl = points.properties.forecast ?: error("No forecast URL available")
    val forecast = this.get(forecastUrl).body<ForecastResponse>()
    return forecast.properties.periods.map { period ->
        """
            ${period.name}:
            Temperature: ${period.temperature}°${period.temperatureUnit}
            Wind: ${period.windSpeed} ${period.windDirection}
            ${period.shortForecast}
        """.trimIndent()
    }
}

@Serializable
data class PointsResponse(val properties: PointsProperties)

@Serializable
data class PointsProperties(val forecast: String? = null)

@Serializable
data class ForecastResponse(val properties: ForecastProperties)

@Serializable
data class ForecastProperties(val periods: List<ForecastPeriod> = emptyList())

@Serializable
data class ForecastPeriod(
    val name: String? = null,
    val temperature: Int? = null,
    val temperatureUnit: String? = null,
    val windSpeed: String? = null,
    val windDirection: String? = null,
    val shortForecast: String? = null,
)

@Serializable
data class AlertsResponse(val features: List<AlertFeature> = emptyList())

@Serializable
data class AlertFeature(val properties: AlertProperties)

@Serializable
data class AlertProperties(
    val event: String? = null,
    val areaDesc: String? = null,
    val severity: String? = null,
    val status: String? = null,
    val headline: String? = null,
)
```

### Implementing tool execution

### 实现工具执行

The tool execution handler is responsible for actually executing the logic of each tool. Let's add it:

工具执行处理器负责实际执行每个工具的逻辑。让我们添加它：

```kotlin
// Register weather tools

server.addTool(
    name = "get_alerts",
    description = "Get weather alerts for a US state. Input is a two-letter US state code (e.g. CA, NY)",
    inputSchema = ToolSchema(
        properties = buildJsonObject {
            putJsonObject("state") {
                put("type", "string")
                put("description", "Two-letter US state code (e.g. CA, NY)")
            }
        },
        required = listOf("state"),
    ),
) { request ->
    val state = request.arguments?.get("state")?.jsonPrimitive?.content
        ?: return@addTool CallToolResult(
            content = listOf(TextContent("The 'state' parameter is required.")),
        )

    val alerts = httpClient.getAlerts(state)
    CallToolResult(content = alerts.map { TextContent(it) })
}

server.addTool(
    name = "get_forecast",
    description = "Get weather forecast for a location. Note: only US locations are supported by the NWS API.",
    inputSchema = ToolSchema(
        properties = buildJsonObject {
            putJsonObject("latitude") {
                put("type", "number")
                put("description", "Latitude of the location")
            }
            putJsonObject("longitude") {
                put("type", "number")
                put("description", "Longitude of the location")
            }
        },
        required = listOf("latitude", "longitude"),
    ),
) { request ->
    val latitude = request.arguments?.get("latitude")?.jsonPrimitive?.doubleOrNull
    val longitude = request.arguments?.get("longitude")?.jsonPrimitive?.doubleOrNull
    if (latitude == null || longitude == null) {
        return@addTool CallToolResult(
            content = listOf(TextContent("The 'latitude' and 'longitude' parameters are required.")),
        )
    }

    val forecast = httpClient.getForecast(latitude, longitude)
    CallToolResult(content = forecast.map { TextContent(it) })
}
```

### Running the server

### 运行服务器

Finally, implement the main function to run the server:

最后，实现运行服务器的主函数：

```kotlin
fun main() = runMcpServer()
```

You can run the server directly during development:

你可以在开发期间直接运行服务器：

```
./gradlew run
```

For production use, build the shadow JAR:

对于生产用途，请构建 shadow JAR：

```
./gradlew build
java -jar build/libs/weather-0.1.0-all.jar
```

Let's now test your server from an existing MCP host, Claude for Desktop.

现在让我们在现有的 MCP 宿主——Claude for Desktop 中测试你的服务器。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**We'll need to configure Claude for Desktop for whichever MCP servers you want to use.
To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor.
Make sure to create the file if it doesn't exist.For example, if you have [VS Code](https://code.visualstudio.com/) installed:

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。
为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。
如果文件不存在，请创建它。例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key.
The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.In this case, we'll add our single weather server like so:

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. Launch it by running `java -jar /ABSOLUTE/PATH/TO/PARENT/FOLDER/weather/build/libs/weather-0.1.0-all.jar`

1. 有一个名为 "weather" 的 MCP 服务器
2. 通过运行 `java -jar /ABSOLUTE/PATH/TO/PARENT/FOLDER/weather/build/libs/weather-0.1.0-all.jar` 来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/csharp-sdk/tree/main/samples/QuickstartWeatherServer)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/csharp-sdk/tree/main/samples/QuickstartWeatherServer)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* C#
* LLMs like Claude
* .NET 8 or higher

- C#
- 像 Claude 这样的大语言模型
- .NET 8 或更高版本

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `Console.WriteLine()` or `Console.Write()`, as they write to standard output (stdout). Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `Console.WriteLine()` 或 `Console.Write()`，因为它们写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use a logging library that writes to stderr or files.

- 使用写入 stderr 或文件的日志库。

### System requirements

### 系统要求

* [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) or higher installed.

- 已安装 [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) 或更高版本。

### Set up your environment

### 设置环境

First, let's install `dotnet` if you haven't already. You can download `dotnet` from [official Microsoft .NET website](https://dotnet.microsoft.com/download/). Verify your `dotnet` installation:

首先，如果你还没有安装 `dotnet`，请从 [Microsoft .NET 官方网站](https://dotnet.microsoft.com/download/) 下载。验证你的 `dotnet` 安装：

```
dotnet --version
```

Now, let's create and set up your project:

现在，让我们创建并设置你的项目：

After running `dotnet new console`, you will be presented with a new C# project.
You can open the project in your favorite IDE, such as [Visual Studio](https://visualstudio.microsoft.com/) or [Rider](https://www.jetbrains.com/rider/).
Alternatively, you can create a C# application using the [Visual Studio project wizard](https://learn.microsoft.com/en-us/visualstudio/get-started/csharp/tutorial-console?view=vs-2022).
After creating the project, add NuGet package for the Model Context Protocol SDK and hosting:

运行 `dotnet new console` 后，你将获得一个新的 C# 项目。
你可以在你喜欢的 IDE（如 [Visual Studio](https://visualstudio.microsoft.com/) 或 [Rider](https://www.jetbrains.com/rider/)）中打开该项目。
或者，你也可以使用 [Visual Studio 项目向导](https://learn.microsoft.com/en-us/visualstudio/get-started/csharp/tutorial-console?view=vs-2022) 创建 C# 应用。
创建项目后，添加 Model Context Protocol SDK 和托管的 NuGet 包：

```
# Add the Model Context Protocol SDK NuGet package
dotnet add package ModelContextProtocol --prerelease
# Add the .NET Hosting NuGet package
dotnet add package Microsoft.Extensions.Hosting
```

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

Open the `Program.cs` file in your project and replace its contents with the following code:

打开项目中的 `Program.cs` 文件，并将其内容替换为以下代码：

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using ModelContextProtocol;
using System.Net.Http.Headers;

var builder = Host.CreateEmptyApplicationBuilder(settings: null);

builder.Services.AddMcpServer()
    .WithStdioServerTransport()
    .WithToolsFromAssembly();

builder.Services.AddSingleton(_ =>
{
    var client = new HttpClient() { BaseAddress = new Uri("https://api.weather.gov") };
    client.DefaultRequestHeaders.UserAgent.Add(new ProductInfoHeaderValue("weather-tool", "1.0"));
    return client;
});

var app = builder.Build();

await app.RunAsync();
```

This code sets up a basic console application that uses the Model Context Protocol SDK to create an MCP server with standard I/O transport.

这段代码设置了一个基本的控制台应用，使用 Model Context Protocol SDK 创建一个基于标准 I/O 传输的 MCP 服务器。

### Weather API helper functions

### 天气 API 辅助函数

Create an extension class for `HttpClient` which helps simplify JSON request handling:

为 `HttpClient` 创建一个扩展类，以简化 JSON 请求处理：

```csharp
using System.Text.Json;

internal static class HttpClientExt
{
    public static async Task<JsonDocument> ReadJsonDocumentAsync(this HttpClient client, string requestUri)
    {
        using var response = await client.GetAsync(requestUri);
        response.EnsureSuccessStatusCode();
        return await JsonDocument.ParseAsync(await response.Content.ReadAsStreamAsync());
    }
}
```

Next, define a class with the tool execution handlers for querying and converting responses from the National Weather Service API:

接下来，定义一个类，用于处理查询美国国家气象局 API 并将其响应进行转换的工具执行：

```csharp
using ModelContextProtocol.Server;
using System.ComponentModel;
using System.Globalization;
using System.Text.Json;

namespace QuickstartWeatherServer.Tools;

[McpServerToolType]
public static class WeatherTools
{
    [McpServerTool, Description("Get weather alerts for a US state code.")]
    public static async Task<string> GetAlerts(
        HttpClient client,
        [Description("The US state code to get alerts for.")] string state)
    {
        using var jsonDocument = await client.ReadJsonDocumentAsync($"/alerts/active/area/{state}");
        var jsonElement = jsonDocument.RootElement;
        var alerts = jsonElement.GetProperty("features").EnumerateArray();

        if (!alerts.Any())
        {
            return "No active alerts for this state.";
        }

        return string.Join("\n--\n", alerts.Select(alert =>
        {
            JsonElement properties = alert.GetProperty("properties");
            return $"""
                    Event: {properties.GetProperty("event").GetString()}
                    Area: {properties.GetProperty("areaDesc").GetString()}
                    Severity: {properties.GetProperty("severity").GetString()}
                    Description: {properties.GetProperty("description").GetString()}
                    Instruction: {properties.GetProperty("instruction").GetString()}
                    """;
        }));
    }

    [McpServerTool, Description("Get weather forecast for a location.")]
    public static async Task<string> GetForecast(
        HttpClient client,
        [Description("Latitude of the location.")] double latitude,
        [Description("Longitude of the location.")] double longitude)
    {
        var pointUrl = string.Create(CultureInfo.InvariantCulture, $"/points/{latitude},{longitude}");
        using var jsonDocument = await client.ReadJsonDocumentAsync(pointUrl);
        var forecastUrl = jsonDocument.RootElement.GetProperty("properties").GetProperty("forecast").GetString()
            ?? throw new Exception($"No forecast URL provided by {client.BaseAddress}points/{latitude},{longitude}");

        using var forecastDocument = await client.ReadJsonDocumentAsync(forecastUrl);
        var periods = forecastDocument.RootElement.GetProperty("properties").GetProperty("periods").EnumerateArray();

        return string.Join("\n---\n", periods.Select(period => $"""
                {period.GetProperty("name").GetString()}
                Temperature: {period.GetProperty("temperature").GetInt32()}°F
                Wind: {period.GetProperty("windSpeed").GetString()} {period.GetProperty("windDirection").GetString()}
                Forecast: {period.GetProperty("detailedForecast").GetString()}
                """));
    }
}
```

### Running the server

### 运行服务器

Finally, run the server using the following command:

最后，使用以下命令运行服务器：

```
dotnet run
```

This will start the server and listen for incoming requests on standard input/output.

这将启动服务器并通过标准输入/输出监听传入的请求。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**
We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.
For example, if you have [VS Code](https://code.visualstudio.com/) installed:

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**
我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。如果文件不存在，请创建它。
例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key.
The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.In this case, we'll add our single weather server like so:

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. Launch it by running `dotnet run /ABSOLUTE/PATH/TO/PROJECT`

1. 有一个名为 "weather" 的 MCP 服务器
2. 通过运行 `dotnet run /ABSOLUTE/PATH/TO/PROJECT` 来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-ruby)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-ruby)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* Ruby
* LLMs like Claude

- Ruby
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `puts` or `print`, as they write to standard output (stdout) by default. Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `puts` 或 `print`，因为它们默认写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use a logging library that writes to stderr or files.

- 使用写入 stderr 或文件的日志库。

### Quick Examples

### 快速示例

```ruby
# Bad (STDIO)
puts "Processing request"

# Good (STDIO)
require "logger"
logger = Logger.new($stderr)
logger.info("Processing request")
```

### System requirements

### 系统要求

* Ruby 2.7 or higher installed.

- 已安装 Ruby 2.7 或更高版本。

### Set up your environment

### 设置环境

First, let's make sure you have Ruby installed. You can check by running:

首先，确保你已安装 Ruby。你可以通过运行以下命令检查：

```
ruby --version
```

Now, let's create and set up our project:

现在，让我们创建并设置项目：

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Importing packages and setting up constants

### 导入包并设置常量

Open `weather.rb` and add these requires and constants at the top:

打开 `weather.rb`，在顶部添加以下 require 和常量：

```ruby
require "json"
require "mcp"
require "net/http"
require "uri"

NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"
```

The `mcp` gem provides the Model Context Protocol SDK for Ruby, with classes for server implementation and stdio transport.

`mcp` gem 提供了 Ruby 的 Model Context Protocol SDK，包含服务器实现和 stdio 传输的类。

### Helper methods

### 辅助方法

Next, let's add helper methods for querying and formatting data from the National Weather Service API:

接下来，添加辅助方法，用于向美国国家气象局 API 查询数据并格式化结果：

```ruby
module HelperMethods
  def make_nws_request(url)
    uri = URI(url)
    request = Net::HTTP::Get.new(uri)
    request["User-Agent"] = USER_AGENT
    request["Accept"] = "application/geo+json"

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    raise "HTTP #{response.code}: #{response.message}" unless response.is_a?(Net::HTTPSuccess)

    JSON.parse(response.body)
  end

  def format_alert(feature)
    properties = feature["properties"]

    <<~ALERT
      Event: #{properties["event"] || "Unknown"}
      Area: #{properties["areaDesc"] || "Unknown"}
      Severity: #{properties["severity"] || "Unknown"}
      Description: #{properties["description"] || "No description available"}
      Instructions: #{properties["instruction"] || "No specific instructions provided"}
    ALERT
  end
end
```

### Implementing tool execution

### 实现工具执行

Now let's define our tool classes. Each tool subclasses `MCP::Tool` and implements the tool logic:

现在让我们定义工具类。每个工具继承 `MCP::Tool` 并实现工具逻辑：

```ruby
class GetAlerts < MCP::Tool
  extend HelperMethods

  tool_name "get_alerts"
  description "Get weather alerts for a US state"
  input_schema(
    properties: {
      state: {
        type: "string",
        description: "Two-letter US state code (e.g. CA, NY)"
      }
    },
    required: ["state"]
  )

  def self.call(state:)
    url = "#{NWS_API_BASE}/alerts/active/area/#{state.upcase}"
    data = make_nws_request(url)

    if data["features"].empty?
      return MCP::Tool::Response.new([{
        type: "text",
        text: "No active alerts for this state."
      }])
    end

    alerts = data["features"].map { |feature| format_alert(feature) }
    MCP::Tool::Response.new([{
      type: "text",
      text: alerts.join("\n---\n")
    }])
  end
end

class GetForecast < MCP::Tool
  extend HelperMethods

  tool_name "get_forecast"
  description "Get weather forecast for a location"
  input_schema(
    properties: {
      latitude: {
        type: "number",
        description: "Latitude of the location"
      },
      longitude: {
        type: "number",
        description: "Longitude of the location"
      }
    },
    required: ["latitude", "longitude"]
  )

  def self.call(latitude:, longitude:)
    # First get the forecast grid endpoint.
    points_url = "#{NWS_API_BASE}/points/#{latitude},#{longitude}"
    points_data = make_nws_request(points_url)

    # Get the forecast URL from the points response.
    forecast_url = points_data["properties"]["forecast"]
    forecast_data = make_nws_request(forecast_url)

    # Format the periods into a readable forecast.
    periods = forecast_data["properties"]["periods"]
    forecasts = periods.first(5).map do |period|
      <<~FORECAST
        #{period["name"]}:
        Temperature: #{period["temperature"]}°#{period["temperatureUnit"]}
        Wind: #{period["windSpeed"]} #{period["windDirection"]}
        Forecast: #{period["detailedForecast"]}
      FORECAST
    end

    MCP::Tool::Response.new([{
      type: "text",
      text: forecasts.join("\n---\n")
    }])
  end
end
```

### Running the server

### 运行服务器

Finally, initialize and run the server:

最后，初始化并运行服务器：

```ruby
server = MCP::Server.new(
  name: "weather",
  version: "1.0.0",
  tools: [GetAlerts, GetForecast]
)

transport = MCP::Server::Transports::StdioTransport.new(server)
transport.open
```

Your server is complete! Run `bundle exec ruby weather.rb` to start the MCP server, which will listen for messages from MCP hosts.Let's now test your server from an existing MCP host, Claude for Desktop.

你的服务器已经完成！运行 `bundle exec ruby weather.rb` 即可启动 MCP 服务器，它将监听来自 MCP 宿主的消息。现在让我们在现有的 MCP 宿主——Claude for Desktop 中测试你的服务器。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.For example, if you have [VS Code](https://code.visualstudio.com/) installed:

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。如果文件不存在，请创建它。例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key.
The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.In this case, we'll add our single weather server like so:

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. Launch it by running `bundle exec ruby weather.rb` in the specified directory

1. 有一个名为 "weather" 的 MCP 服务器
2. 在指定目录中通过运行 `bundle exec ruby weather.rb` 来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-rust)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-rust)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* Rust programming language
* Async/await in Rust
* LLMs like Claude

- Rust 编程语言
- Rust 中的 async/await
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `println!()` or `print!()`, as they write to standard output (stdout). Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `println!()` 或 `print!()`，因为它们写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use a logging library that writes to stderr or files, such as `tracing` or `log` in Rust.
* Configure your logging framework to avoid stdout output.

- 使用写入 stderr 或文件的日志库，例如 Rust 中的 `tracing` 或 `log`。
- 配置日志框架以避免输出到 stdout。

### Quick Examples

### 快速示例

```rust
// Bad (STDIO)
println!("Processing request");

// Good (STDIO)
eprintln!("Processing request"); // writes to stderr
```

### System requirements

### 系统要求

* Rust 1.70 or higher installed.
* Cargo (comes with Rust installation).

- 已安装 Rust 1.70 或更高版本。
- 已安装 Cargo（随 Rust 一起安装）。

### Set up your environment

### 设置环境

First, let's install Rust if you haven't already. You can install Rust from [rust-lang.org](https://www.rust-lang.org/tools/install):

首先，如果你还没有安装 Rust，可以从 [rust-lang.org](https://www.rust-lang.org/tools/install) 安装：

Verify your Rust installation:

验证你的 Rust 安装：

```
rustc --version
cargo --version
```

Now, let's create and set up our project:

现在，让我们创建并设置项目：

Update your `Cargo.toml` to add the required dependencies:

更新你的 `Cargo.toml`，添加所需的依赖项：

Cargo.toml

```toml
[package]
name = "weather"
version = "0.1.0"
edition = "2024"

[dependencies]
rmcp = { version = "0.3", features = ["server", "macros", "transport-io"] }
tokio = { version = "1.46", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "std", "fmt"] }
```

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Importing packages and constants

### 导入包和常量

Open `src/main.rs` and add these imports and constants at the top:

打开 `src/main.rs`，在顶部添加以下导入和常量：

```rust
use anyhow::Result;
use rmcp::{
    ServerHandler, ServiceExt,
    handler::server::{router::tool::ToolRouter, tool::Parameters},
    model::*,
    schemars, tool, tool_handler, tool_router,
};
use serde::Deserialize;
use serde::de::DeserializeOwned;

const NWS_API_BASE: &str = "https://api.weather.gov";
const USER_AGENT: &str = "weather-app/1.0";
```

The `rmcp` crate provides the Model Context Protocol SDK for Rust, with features for server implementation, procedural macros, and stdio transport.

`rmcp` crate 提供了 Rust 的 Model Context Protocol SDK，包含服务器实现、过程宏和 stdio 传输功能。

### Data structures

### 数据结构

Next, let's define the data structures for deserializing responses from the National Weather Service API:

接下来，定义用于反序列化美国国家气象局 API 响应的数据结构：

```rust
#[derive(Debug, Deserialize)]
struct AlertsResponse {
    features: Vec<AlertFeature>,
}

#[derive(Debug, Deserialize)]
struct AlertFeature {
    properties: AlertProperties,
}

#[derive(Debug, Deserialize)]
struct AlertProperties {
    event: Option<String>,
    #[serde(rename = "areaDesc")]
    area_desc: Option<String>,
    severity: Option<String>,
    description: Option<String>,
    instruction: Option<String>,
}

#[derive(Debug, Deserialize)]
struct PointsResponse {
    properties: PointsProperties,
}

#[derive(Debug, Deserialize)]
struct PointsProperties {
    forecast: String,
}

#[derive(Debug, Deserialize)]
struct ForecastResponse {
    properties: ForecastProperties,
}

#[derive(Debug, Deserialize)]
struct ForecastProperties {
    periods: Vec<ForecastPeriod>,
}

#[derive(Debug, Deserialize)]
struct ForecastPeriod {
    name: String,
    temperature: i32,
    #[serde(rename = "temperatureUnit")]
    temperature_unit: String,
    #[serde(rename = "windSpeed")]
    wind_speed: String,
    #[serde(rename = "windDirection")]
    wind_direction: String,
    #[serde(rename = "detailedForecast")]
    detailed_forecast: String,
}
```

Now define the request types that MCP clients will send:

现在定义 MCP 客户端将发送的请求类型：

```rust
#[derive(serde::Deserialize, schemars::JsonSchema)]
pub struct MCPForecastRequest {
    latitude: f32,
    longitude: f32,
}

#[derive(serde::Deserialize, schemars::JsonSchema)]
pub struct MCPAlertRequest {
    state: String,
}
```

### Helper functions

### 辅助函数

Add helper functions for making API requests and formatting responses:

添加用于发起 API 请求和格式化响应的辅助函数：

```rust
async fn make_nws_request<T: DeserializeOwned>(url: &str) -> Result<T> {
    let client = reqwest::Client::new();
    let rsp = client
        .get(url)
        .header(reqwest::header::USER_AGENT, USER_AGENT)
        .header(reqwest::header::ACCEPT, "application/geo+json")
        .send()
        .await?
        .error_for_status()?;
    Ok(rsp.json::<T>().await?)
}

fn format_alert(feature: &AlertFeature) -> String {
    let props = &feature.properties;
    format!(
        "Event: {}\nArea: {}\nSeverity: {}\nDescription: {}\nInstructions: {}",
        props.event.as_deref().unwrap_or("Unknown"),
        props.area_desc.as_deref().unwrap_or("Unknown"),
        props.severity.as_deref().unwrap_or("Unknown"),
        props
            .description
            .as_deref()
            .unwrap_or("No description available"),
        props
            .instruction
            .as_deref()
            .unwrap_or("No specific instructions provided")
    )
}

fn format_period(period: &ForecastPeriod) -> String {
    format!(
        "{}:\nTemperature: {}°{}\nWind: {} {}\nForecast: {}",
        period.name,
        period.temperature,
        period.temperature_unit,
        period.wind_speed,
        period.wind_direction,
        period.detailed_forecast
    )
}
```

### Implementing the Weather server and tools

### 实现 Weather 服务器和工具

Now let's implement the main Weather server struct with the tool handlers:

现在让我们实现带有工具处理器的主 Weather 服务器结构体：

```rust
pub struct Weather {
    tool_router: ToolRouter<Weather>,
}

#[tool_router]
impl Weather {
    fn new() -> Self {
        Self {
            tool_router: Self::tool_router(),
        }
    }

    #[tool(description = "Get weather alerts for a US state.")]
    async fn get_alerts(
        &self,
        Parameters(MCPAlertRequest { state }): Parameters<MCPAlertRequest>,
    ) -> String {
        let url = format!(
            "{}/alerts/active/area/{}",
            NWS_API_BASE,
            state.to_uppercase()
        );

        match make_nws_request::<AlertsResponse>(&url).await {
            Ok(data) => {
                if data.features.is_empty() {
                    "No active alerts for this state.".to_string()
                } else {
                    data.features
                        .iter()
                        .map(format_alert)
                        .collect::<Vec<_>>()
                        .join("\n---\n")
                }
            }
            Err(_) => "Unable to fetch alerts or no alerts found.".to_string(),
        }
    }

    #[tool(description = "Get weather forecast for a location.")]
    async fn get_forecast(
        &self,
        Parameters(MCPForecastRequest {
            latitude,
            longitude,
        }): Parameters<MCPForecastRequest>,
    ) -> String {
        let points_url = format!("{NWS_API_BASE}/points/{latitude},{longitude}");
        let Ok(points_data) = make_nws_request::<PointsResponse>(&points_url).await else {
            return "Unable to fetch forecast data for this location.".to_string();
        };

        let forecast_url = points_data.properties.forecast;

        let Ok(forecast_data) = make_nws_request::<ForecastResponse>(&forecast_url).await else {
            return "Unable to fetch forecast data for this location.".to_string();
        };

        let periods = &forecast_data.properties.periods;
        let forecast_summary: String = periods
            .iter()
            .take(5) // Next 5 periods only
            .map(format_period)
            .collect::<Vec<String>>()
            .join("\n---\n");
        forecast_summary
    }
}
```

The `#[tool_router]` macro automatically generates the routing logic, and the `#[tool]` attribute marks methods as MCP tools.

`#[tool_router]` 宏自动生成路由逻辑，`#[tool]` 属性将方法标记为 MCP 工具。

### Implementing the ServerHandler

### 实现 ServerHandler

Implement the `ServerHandler` trait to define server capabilities:

实现 `ServerHandler` trait 以定义服务器能力：

```rust
#[tool_handler]
impl ServerHandler for Weather {
    fn get_info(&self) -> ServerInfo {
        ServerInfo {
            capabilities: ServerCapabilities::builder().enable_tools().build(),
            ..Default::default()
        }
    }
}
```

### Running the server

### 运行服务器

Finally, implement the main function to run the server with stdio transport:

最后，实现使用 stdio 传输运行服务器的主函数：

```rust
#[tokio::main]
async fn main() -> Result<()> {
    let transport = (tokio::io::stdin(), tokio::io::stdout());
    let service = Weather::new().serve(transport).await?;
    service.waiting().await?;
    Ok(())
}
```

Build your server with:

使用以下命令构建你的服务器：

```
cargo build --release
```

The compiled binary will be in `target/release/weather`.

编译后的二进制文件将位于 `target/release/weather`。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.For example, if you have [VS Code](https://code.visualstudio.com/) installed:

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。如果文件不存在，请创建它。例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key.
The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.In this case, we'll add our single weather server like so:

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. Launch it by running the compiled binary at the specified path

1. 有一个名为 "weather" 的 MCP 服务器
2. 通过运行指定路径的编译后二进制文件来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。

Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-go)

让我们开始构建天气服务器吧！[你可以在这里找到我们将要构建的完整代码。](https://github.com/modelcontextprotocol/quickstart-resources/tree/main/weather-server-go)

### Prerequisite knowledge

### 前置知识

This quickstart assumes you have familiarity with:

本快速入门假设你已熟悉：

* Go
* LLMs like Claude

- Go
- 像 Claude 这样的大语言模型

### Logging in MCP Servers

### MCP 服务器中的日志记录

When implementing MCP servers, be careful about how you handle logging:

在实现 MCP 服务器时，请注意日志记录的处理方式：

**For STDIO-based servers:** Never use `fmt.Println()` or `fmt.Printf()`, as they write to standard output (stdout). Writing to stdout will corrupt the JSON-RPC messages and break your server.

**对于基于 STDIO 的服务器：** 永远不要使用 `fmt.Println()` 或 `fmt.Printf()`，因为它们写入标准输出（stdout）。写入 stdout 会破坏 JSON-RPC 消息并导致服务器崩溃。

**For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

**对于基于 HTTP 的服务器：** 标准输出日志不会影响 HTTP 响应，因此没有问题。

### Best Practices

### 最佳实践

* Use `log.Println()` (which defaults to stderr) or a logging library that writes to stderr or files.
* Use `fmt.Fprintf(os.Stderr, ...)` to write to stderr explicitly.

- 使用 `log.Println()`（默认写入 stderr），或使用写入 stderr 或文件的日志库。
- 使用 `fmt.Fprintf(os.Stderr, ...)` 显式写入 stderr。

### Quick Examples

### 快速示例

```go
// Bad (STDIO)
fmt.Println("Processing request")

// Good (STDIO)
log.Println("Processing request") // defaults to stderr

// Good (STDIO)
fmt.Fprintln(os.Stderr, "Processing request")
```

### System requirements

### 系统要求

* Go 1.24 or higher installed.

- 已安装 Go 1.24 或更高版本。

### Set up your environment

### 设置环境

First, let's install Go if you haven't already. You can download and install Go from [go.dev](https://go.dev/dl/).Verify your Go installation:

首先，如果你还没有安装 Go，可以从 [go.dev](https://go.dev/dl/) 下载并安装。验证你的 Go 安装：

```
go version
```

Now, let's create and set up our project:

现在，让我们创建并设置项目：

Now let's dive into building your server.

现在让我们开始构建你的服务器。

## Building your server

## 构建服务器

### Importing packages and constants

### 导入包和常量

Add these to the top of your `main.go`:

将以下内容添加到你的 `main.go` 文件顶部：

```go
package main

import (
	"cmp"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"strings"

	"github.com/modelcontextprotocol/go-sdk/mcp"
)

const (
	NWSAPIBase = "https://api.weather.gov"
	UserAgent  = "weather-app/1.0"
)
```

### Data structures

### 数据结构

Next, let's define the data structures used by our tools:

接下来，定义我们的工具所使用的数据结构：

```go
type PointsResponse struct {
	Properties struct {
		Forecast string `json:"forecast"`
	} `json:"properties"`
}

type ForecastResponse struct {
	Properties struct {
		Periods []ForecastPeriod `json:"periods"`
	} `json:"properties"`
}

type ForecastPeriod struct {
	Name             string `json:"name"`
	Temperature      int    `json:"temperature"`
	TemperatureUnit  string `json:"temperatureUnit"`
	WindSpeed        string `json:"windSpeed"`
	WindDirection    string `json:"windDirection"`
	DetailedForecast string `json:"detailedForecast"`
}

type AlertsResponse struct {
	Features []AlertFeature `json:"features"`
}

type AlertFeature struct {
	Properties AlertProperties `json:"properties"`
}

type AlertProperties struct {
	Event       string `json:"event"`
	AreaDesc    string `json:"areaDesc"`
	Severity    string `json:"severity"`
	Description string `json:"description"`
	Instruction string `json:"instruction"`
}

type ForecastInput struct {
	Latitude  float64 `json:"latitude" jsonschema:"Latitude of the location"`
	Longitude float64 `json:"longitude" jsonschema:"Longitude of the location"`
}

type AlertsInput struct {
	State string `json:"state" jsonschema:"Two-letter US state code (e.g. CA, NY)"`
}
```

### Helper functions

### 辅助函数

Next, let's add our helper functions for querying and formatting the data from the National Weather Service API:

接下来，添加辅助函数，用于向美国国家气象局 API 查询数据并格式化结果：

```go
func makeNWSRequest[T any](ctx context.Context, url string) (*T, error) {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return nil, fmt.Errorf("failed to create request: %w", err)
	}

	req.Header.Set("User-Agent", UserAgent)
	req.Header.Set("Accept", "application/geo+json")

	client := http.DefaultClient
	resp, err := client.Do(req)
	if err != nil {
		return nil, fmt.Errorf("failed to make request to %s: %w", url, err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		return nil, fmt.Errorf("HTTP error %d: %s", resp.StatusCode, string(body))
	}

	var result T
	if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
		return nil, fmt.Errorf("failed to decode response: %w", err)
	}

	return &result, nil
}

func formatAlert(alert AlertFeature) string {
	props := alert.Properties
	event := cmp.Or(props.Event, "Unknown")
	areaDesc := cmp.Or(props.AreaDesc, "Unknown")
	severity := cmp.Or(props.Severity, "Unknown")
	description := cmp.Or(props.Description, "No description available")
	instruction := cmp.Or(props.Instruction, "No specific instructions provided")

	return fmt.Sprintf(`
Event: %s
Area: %s
Severity: %s
Description: %s
Instructions: %s
`, event, areaDesc, severity, description, instruction)
}

func formatPeriod(period ForecastPeriod) string {
	return fmt.Sprintf(`
%s:
Temperature: %d°%s
Wind: %s %s
Forecast: %s
`, period.Name, period.Temperature, period.TemperatureUnit,
		period.WindSpeed, period.WindDirection, period.DetailedForecast)
}
```

### Implementing tool execution

### 实现工具执行

The tool execution handler is responsible for actually executing the logic of each tool. Let's add it:

工具执行处理器负责实际执行每个工具的逻辑。让我们添加它：

```go
func getForecast(ctx context.Context, req *mcp.CallToolRequest, input ForecastInput) (
	*mcp.CallToolResult, any, error,
) {
	// Get points data
	pointsURL := fmt.Sprintf("%s/points/%f,%f", NWSAPIBase, input.Latitude, input.Longitude)
	pointsData, err := makeNWSRequest[PointsResponse](ctx, pointsURL)
	if err != nil {
		return &mcp.CallToolResult{
			Content: []mcp.Content{
				&mcp.TextContent{Text: "Unable to fetch forecast data for this location."},
			},
		}, nil, nil
	}

	// Get forecast data
	forecastURL := pointsData.Properties.Forecast
	if forecastURL == "" {
		return &mcp.CallToolResult{
			Content: []mcp.Content{
				&mcp.TextContent{Text: "Unable to fetch forecast URL."},
			},
		}, nil, nil
	}

	forecastData, err := makeNWSRequest[ForecastResponse](ctx, forecastURL)
	if err != nil {
		return &mcp.CallToolResult{
			Content: []mcp.Content{
				&mcp.TextContent{Text: "Unable to fetch detailed forecast."},
			},
		}, nil, nil
	}

	// Format the periods
	periods := forecastData.Properties.Periods
	if len(periods) == 0 {
		return &mcp.CallToolResult{
			Content: []mcp.Content{
				&mcp.TextContent{Text: "No forecast periods available."},
			},
		}, nil, nil
	}

	// Show next 5 periods
	var forecasts []string
	for i := range min(5, len(periods)) {
		forecasts = append(forecasts, formatPeriod(periods[i]))
	}

	result := strings.Join(forecasts, "\n---\n")

	return &mcp.CallToolResult{
		Content: []mcp.Content{
			&mcp.TextContent{Text: result},
		},
	}, nil, nil
}

func getAlerts(ctx context.Context, req *mcp.CallToolRequest, input AlertsInput) (
	*mcp.CallToolResult, any, error,
) {
	// Build alerts URL
	stateCode := strings.ToUpper(input.State)
	alertsURL := fmt.Sprintf("%s/alerts/active/area/%s", NWSAPIBase, stateCode)

	alertsData, err := makeNWSRequest[AlertsResponse](ctx, alertsURL)
	if err != nil {
		return &mcp.CallToolResult{
			Content: []mcp.Content{
				&mcp.TextContent{Text: "Unable to fetch alerts or no alerts found."},
			},
		}, nil, nil
	}

	// Check if there are any alerts
	if len(alertsData.Features) == 0 {
		return &mcp.CallToolResult{
			Content: []mcp.Content{
				&mcp.TextContent{Text: "No active alerts for this state."},
			},
		}, nil, nil
	}

	// Format alerts
	var alerts []string
	for _, feature := range alertsData.Features {
		alerts = append(alerts, formatAlert(feature))
	}

	result := strings.Join(alerts, "\n---\n")

	return &mcp.CallToolResult{
		Content: []mcp.Content{
			&mcp.TextContent{Text: result},
		},
	}, nil, nil
}
```

### Running the server

### 运行服务器

Finally, implement the main function to run the server:

最后，实现运行服务器的主函数：

```go
func main() {
	// Create MCP server
	server := mcp.NewServer(&mcp.Implementation{
		Name:    "weather",
		Version: "1.0.0",
	}, nil)

	// Add get_forecast tool
	mcp.AddTool(server, &mcp.Tool{
		Name:        "get_forecast",
		Description: "Get weather forecast for a location",
	}, getForecast)

	// Add get_alerts tool
	mcp.AddTool(server, &mcp.Tool{
		Name:        "get_alerts",
		Description: "Get weather alerts for a US state",
	}, getAlerts)

	// Run server on stdio transport
	if err := server.Run(context.Background(), &mcp.StdioTransport{}); err != nil {
		log.Fatal(err)
	}
}
```

Build your server with:

使用以下命令构建你的服务器：

```go build -o weather .
```

The compiled binary will be in `./weather`.

编译后的二进制文件将位于 `./weather`。

## Testing your server with Claude for Desktop

## 使用 Claude for Desktop 测试服务器

First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**
We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.
For example, if you have [VS Code](https://code.visualstudio.com/) installed:

首先，确保你已安装 Claude for Desktop。[你可以在这里下载最新版本。](https://claude.ai/download) 如果你已经安装了 Claude for Desktop，**请确保它已更新到最新版本。**
我们需要在 Claude for Desktop 中配置你想使用的 MCP 服务器。为此，请在文本编辑器中打开 Claude for Desktop 应用配置文件 `~/Library/Application Support/Claude/claude_desktop_config.json`。如果文件不存在，请创建它。
例如，如果你已安装 [VS Code](https://code.visualstudio.com/)：

You'll then add your servers in the `mcpServers` key.
The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.In this case, we'll add our single weather server like so:

然后在 `mcpServers` 键下添加你的服务器。只有当至少有一个服务器被正确配置时，Claude for Desktop 中的 MCP UI 元素才会显示。在本例中，我们将像这样添加单个天气服务器：

This tells Claude for Desktop:

这会告诉 Claude for Desktop：

1. There's an MCP server named "weather"
2. Launch it by running the compiled binary at the specified path

1. 有一个名为 "weather" 的 MCP 服务器
2. 通过运行指定路径的编译后二进制文件来启动它

Save the file, and restart **Claude for Desktop**.

保存文件，然后重启 **Claude for Desktop**。
