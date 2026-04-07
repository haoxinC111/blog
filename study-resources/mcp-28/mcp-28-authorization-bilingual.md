# Understanding Authorization in MCP

**Source URL:** https://modelcontextprotocol.io/docs/tutorials/security/authorization  
**整理日期:** 2026-04-07

---

Authorization in the Model Context Protocol (MCP) secures access to sensitive resources and operations exposed by MCP servers. If your MCP server handles user data or administrative actions, authorization ensures only permitted users can access its endpoints.
MCP uses standardized authorization flows to build trust between MCP clients and MCP servers. Its design doesn't focus on one specific authorization or identity system, but rather follows the conventions outlined for [OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13). For detailed information, see the [Authorization specification](/specification/latest/basic/authorization).

MCP（Model Context Protocol）中的授权机制用于保护对 MCP 服务器暴露的敏感资源和操作的访问。如果你的 MCP 服务器处理用户数据或管理操作，授权可以确保只有被允许的用户才能访问其端点。
MCP 遵循标准化的授权流程，以在 MCP 客户端和 MCP 服务器之间建立信任。其设计并不聚焦于某一种特定的授权或身份识别系统，而是遵循 [OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13) 中概述的约定。更多详情请参阅[授权规范](/specification/latest/basic/authorization)。

While authorization for MCP servers is **optional**, it is strongly recommended when:

虽然 MCP 服务器的授权是**可选的**，但在以下情况下强烈建议启用：

* Your server accesses user-specific data (emails, documents, databases)
* You need to audit who performed which actions
* Your server grants access to its APIs that require user consent
* You're building for enterprise environments with strict access controls
* You want to implement rate limiting or usage tracking per user

- 你的服务器访问用户特定数据（电子邮件、文档、数据库）
- 你需要审计谁执行了哪些操作
- 你的服务器授予对其 API 的访问权限，而这些 API 需要用户同意
- 你正在为具有严格访问控制的企业环境构建
- 你希望按用户实现速率限制或使用量跟踪

## The Authorization Flow: Step by Step / 授权流程：分步说明


Let's walk through what happens when a client wants to connect to your protected MCP server:

让我们看一下当客户端想要连接到你的受保护 MCP 服务器时会发生什么：

1

2

3

4

5

6

## Implementation Example / 实现示例


To get started with a practical implementation, we will use a [Keycloak](https://www.keycloak.org/) authorization server hosted in a Docker container. Keycloak is an open-source authorization server that can be easily deployed locally for testing and experimentation.
Make sure that you download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/). We will need it to deploy Keycloak on our development machine.

为了开始一个实际的实现，我们将使用托管在 Docker 容器中的 [Keycloak](https://www.keycloak.org/) 授权服务器。Keycloak 是一个开源授权服务器，可以轻松地在本地部署以进行测试和实验。
请确保你已下载并安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/)。我们需要用它来在开发机器上部署 Keycloak。

### Keycloak Setup / Keycloak 设置


From your terminal application, run the following command to start the Keycloak container:

在你的终端应用中，运行以下命令启动 Keycloak 容器：

```
docker run -p 127.0.0.1:8080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak start-dev
```

This command will pull the Keycloak container image locally and bootstrap the basic configuration. It will run on port `8080` and have an `admin` user with `admin` password.

该命令会在本地拉取 Keycloak 容器镜像并完成基本配置的引导。它将在 `8080` 端口运行，并创建一个用户名为 `admin`、密码为 `admin` 的管理员账户。

You will be able to access the Keycloak authorization server from your browser at `http://localhost:8080`.

你可以在浏览器中通过 `http://localhost:8080` 访问 Keycloak 授权服务器。

When running with the default configuration, Keycloak will already support many of the capabilities that we need for MCP servers, including Dynamic Client Registration. You can check this by looking at the OIDC configuration, available at:

使用默认配置运行时，Keycloak 已经支持我们在 MCP 服务器中所需的许多功能，包括动态客户端注册（Dynamic Client Registration）。你可以通过查看以下地址的 OIDC 配置来确认这一点：

```
http://localhost:8080/realms/master/.well-known/openid-configuration
```

We will also need to set up Keycloak to support our scopes and allow our host (local machine) to dynamically register clients, as the default policies restrict anonymous dynamic client registration.
Go to **Client scopes** in the Keycloak dashboard and create a new `mcp:tools` scope. We will use this to access all of the tools on our MCP server.

我们还需要配置 Keycloak 以支持我们的作用域（scopes），并允许我们的主机（本地机器）动态注册客户端，因为默认策略限制了匿名的动态客户端注册。
请在 Keycloak 控制台中进入 **Client scopes**，并创建一个名为 `mcp:tools` 的新作用域。我们将使用它来访问 MCP 服务器上的所有工具。

After creating the scope, make sure that you assign its type to **Default** and have flipped the **Include in token scope** switch, as this will be needed for token validation.

创建作用域后，请确保将其类型设为 **Default**，并打开 **Include in token scope** 开关，因为这在令牌验证时是必需的。

Let's now also set up an **audience** for our Keycloak-issued tokens. An audience is important to configure because it embeds the intended destination directly into the issued access token. This helps your MCP server to verify that the token it got was actually meant for it rather than some other API. This is key to help avoid token passthrough scenarios.

现在让我们为 Keycloak 签发的令牌设置一个 **audience**（受众）。配置 audience 很重要，因为它将预期的目标直接嵌入到签发的访问令牌中。这有助于你的 MCP 服务器验证收到的令牌确实是发给自己的，而不是给其他 API 的。这对于避免令牌传递（token passthrough）场景至关重要。

To do this, open your `mcp:tools` client scope and click on **Mappers**, followed by **Configure a new mapper**. Select **Audience**.

为此，请打开 `mcp:tools` 客户端作用域，点击 **Mappers**，然后点击 **Configure a new mapper**。选择 **Audience**。

For **Name**, use `audience-config`. Add a value for **Included Custom Audience**, set to `http://localhost:3000`. This will be the URI of our test server.

**Name** 使用 `audience-config`。**Included Custom Audience** 的值设为 `http://localhost:3000`。这将是我们测试服务器的 URI。

Now, navigate to **Clients**, then **Client registration**, and then **Trusted Hosts**. Disable the **Client URIs Must Match** setting and add the hosts from which you're testing. You can get your current host IP by running the `ifconfig` command on Linux or macOS, or `ipconfig` on Windows. You can see the IP address you need to add by looking at the keycloak logs for a line that looks like `Failed to verify remote host : 192.168.215.1`. Check that the IP address is associated with your host. This may be for a bridge network depending on your docker setup.

现在，导航到 **Clients**，然后 **Client registration**，再然后 **Trusted Hosts**。禁用 **Client URIs Must Match** 设置，并添加你正在测试的主机。在 Linux 或 macOS 上可以通过 `ifconfig` 命令获取当前主机 IP，在 Windows 上则使用 `ipconfig`。你可以在 Keycloak 日志中查找类似 `Failed to verify remote host : 192.168.215.1` 的行，来确定需要添加的 IP 地址。请确认该 IP 地址与你的主机相关联，具体取决于你的 Docker 设置，这可能是桥接网络的地址。

Lastly, we need to register a new client that we can use with the **MCP server itself** to talk to Keycloak for things like [token introspection](https://oauth.net/2/token-introspection/). To do that:

最后，我们需要注册一个新客户端，供 **MCP 服务器自身**使用，以便与 Keycloak 通信，进行[令牌内省](https://oauth.net/2/token-introspection/)等操作。具体步骤如下：

1. Go to **Clients**.
2. Click **Create client**.
3. Give your client a unique **Client ID** and click **Next**.
4. Enable **Client authentication** and click **Next**.
5. Click **Save**.

1. 进入 **Clients**。
2. 点击 **Create client**。
3. 给客户端一个唯一的 **Client ID**，然后点击 **Next**。
4. 启用 **Client authentication**，然后点击 **Next**。
5. 点击 **Save**。

Worth noting that token introspection is just *one of* the available approaches to validate tokens. This can also be done with the help of standalone libraries, specific to each language and platform.

值得注意的是，令牌内省只是验证令牌的可用方法之一。也可以借助针对各语言和平台的独立库来完成验证。

When you open the client details, go to **Credentials** and take note of the **Client Secret**.

打开客户端详情后，进入 **Credentials**，并记下 **Client Secret**。

With Keycloak configured, every time the authorization flow is triggered, your MCP server will receive a token like this:

完成 Keycloak 配置后，每次触发授权流程时，你的 MCP 服务器都会收到类似这样的令牌：

```
eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI1TjcxMGw1WW5MWk13WGZ1VlJKWGtCS3ZZMzZzb3JnRG5scmlyZ2tlTHlzIn0.eyJleHAiOjE3NTU1NDA4MTcsImlhdCI6MTc1NTU0MDc1NywiYXV0aF90aW1lIjoxNzU1NTM4ODg4LCJqdGkiOiJvbnJ0YWM6YjM0MDgwZmYtODQwNC02ODY3LTgxYmUtMTIzMWI1MDU5M2E4IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9tYXN0ZXIiLCJhdWQiOiJodHRwOi8vbG9jYWxob3N0OjMwMDAiLCJzdWIiOiIzM2VkNmM2Yi1jNmUwLTQ5MjgtYTE2MS1mMmY2OWM3YTAzYjkiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiI3OTc1YTViNi04YjU5LTRhODUtOWNiYS04ZmFlYmRhYjg5NzQiLCJzaWQiOiI4ZjdlYzI3Ni0zNThmLTRjY2MtYjMxMy1kYjA4MjkwZjM3NmYiLCJzY29wZSI6Im1jcDp0b29scyJ9.P5xCRtXORly0R0EXjyqRCUx-z3J4uAOWNAvYtLPXroykZuVCCJ-K1haiQSwbURqfsVOMbL7jiV-sD6miuPzI1tmKOkN_Yct0Vp-azvj7U5rEj7U6tvPfMkg2Uj_jrIX0KOskyU2pVvGZ-5BgqaSvwTEdsGu_V3_E0xDuSBq2uj_wmhqiyTFm5lJ1WkM3Hnxxx1_AAnTj7iOKMFZ4VCwMmk8hhSC7clnDauORc0sutxiJuYUZzxNiNPkmNeQtMCGqWdP1igcbWbrfnNXhJ6NswBOuRbh97_QraET3hl-CNmyS6C72Xc0aOwR_uJ7xVSBTD02OaQ1JA6kjCATz30kGYg
```

Decoded, it will look like this:

解码后，它看起来像这样：

```
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "5N710l5YnLZMwXfuVRJXkBKvY36sorgDnlrirgkeLys"
}.{
  "exp": 1755540817,
  "iat": 1755540757,
  "auth_time": 1755538888,
  "jti": "onrtac:b34080ff-8404-6867-81be-1231b50593a8",
  "iss": "http://localhost:8080/realms/master",
  "aud": "http://localhost:3000",
  "sub": "33ed6c6b-c6e0-4928-a161-f2f69c7a03b9",
  "typ": "Bearer",
  "azp": "7975a5b6-8b59-4a85-9cba-8faebdab8974",
  "sid": "8f7ec276-358f-4ccc-b313-db08290f376f",
  "scope": "mcp:tools"
}.[Signature]
```

### MCP Server Setup / MCP 服务器设置


We will now set up our MCP server to use the locally-running Keycloak authorization server. Depending on your programming language preference, you can use one of the supported [MCP SDKs](/docs/sdk).
For our testing purposes, we will create an extremely simple MCP server that exposes two tools - one for addition and another for multiplication. The server will require authorization to access these.

现在我们将设置 MCP 服务器，使其使用本地运行的 Keycloak 授权服务器。根据你的编程语言偏好，可以使用任意一个受支持的 [MCP SDK](/docs/sdk)。
出于测试目的，我们将创建一个极其简单的 MCP 服务器，公开两个工具——一个用于加法，另一个用于乘法。服务器将要求授权才能访问这些工具。

* TypeScript
* Python
* C#

- TypeScript
- Python
- C#

You can see the complete TypeScript project in the [sample repository](https://github.com/localden/min-ts-mcp-auth).Prior to running the code below, ensure that you have a `.env` file with the following content:

你可以在[示例仓库](https://github.com/localden/min-ts-mcp-auth)中查看完整的 TypeScript 项目。在运行以下代码之前，请确保你有一个包含以下内容的 `.env` 文件：

```
# Server host/port
HOST=localhost
PORT=3000

# Auth server location
AUTH_HOST=localhost
AUTH_PORT=8080
AUTH_REALM=master

# Keycloak OAuth client credentials
OAUTH_CLIENT_ID=<YOUR_SERVER_CLIENT_ID>
OAUTH_CLIENT_SECRET=<YOUR_SERVER_CLIENT_SECRET>
```

`OAUTH_CLIENT_ID` and `OAUTH_CLIENT_SECRET` are associated with the MCP server client we created earlier.In addition to implementing the MCP authorization specification, the server below also does token introspection via Keycloak to make sure that the token it receives from the client is valid. It also implements basic logging to allow you to easily diagnose any issues.

`OAUTH_CLIENT_ID` 和 `OAUTH_CLIENT_SECRET` 与我们之前创建的 MCP 服务器客户端相关联。除了实现 MCP 授权规范外，下面的服务器还通过 Keycloak 进行令牌内省，以确保从客户端收到的令牌是有效的。它还实现了基本日志记录，以便你能轻松诊断任何问题。

```typescript
import "dotenv/config";
import express from "express";
import { randomUUID } from "node:crypto";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import { isInitializeRequest } from "@modelcontextprotocol/sdk/types.js";
import { z } from "zod";
import cors from "cors";
import {
  mcpAuthMetadataRouter,
  getOAuthProtectedResourceMetadataUrl,
} from "@modelcontextprotocol/sdk/server/auth/router.js";
import { requireBearerAuth } from "@modelcontextprotocol/sdk/server/auth/middleware/bearerAuth.js";
import { OAuthMetadata } from "@modelcontextprotocol/sdk/shared/auth.js";
import { checkResourceAllowed } from "@modelcontextprotocol/sdk/shared/auth-utils.js";
const CONFIG = {
  host: process.env.HOST || "localhost",
  port: Number(process.env.PORT) || 3000,
  auth: {
    host: process.env.AUTH_HOST || process.env.HOST || "localhost",
    port: Number(process.env.AUTH_PORT) || 8080,
    realm: process.env.AUTH_REALM || "master",
    clientId: process.env.OAUTH_CLIENT_ID || "mcp-server",
    clientSecret: process.env.OAUTH_CLIENT_SECRET || "",
  },
};

function createOAuthUrls() {
  const authBaseUrl = new URL(
    `http://${CONFIG.auth.host}:${CONFIG.auth.port}/realms/${CONFIG.auth.realm}/`,
  );
  return {
    issuer: authBaseUrl.toString(),
    introspection_endpoint: new URL(
      "protocol/openid-connect/token/introspect",
      authBaseUrl,
    ).toString(),
    authorization_endpoint: new URL(
      "protocol/openid-connect/auth",
      authBaseUrl,
    ).toString(),
    token_endpoint: new URL(
      "protocol/openid-connect/token",
      authBaseUrl,
    ).toString(),
  };
}

function createRequestLogger() {
  return (req: any, res: any, next: any) => {
    const start = Date.now();
    res.on("finish", () => {
      const ms = Date.now() - start;
      console.log(
        `${req.method} ${req.originalUrl} -> ${res.statusCode} ${ms}ms`,
      );
    });
    next();
  };
}

const app = express();

app.use(
  express.json({
    verify: (req: any, _res, buf) => {
      req.rawBody = buf?.toString() ?? "";
    },
  }),
);

app.use(
  cors({
    origin: "*",
    exposedHeaders: ["Mcp-Session-Id"],
  }),
);

app.use(createRequestLogger());
```

```typescript
const mcpServerUrl = new URL(`http://${CONFIG.host}:${CONFIG.port}`);
const oauthUrls = createOAuthUrls();

const oauthMetadata: OAuthMetadata = {
  ...oauthUrls,
  response_types_supported: ["code"],
};

const tokenVerifier = {
  verifyAccessToken: async (token: string) => {
    const endpoint = oauthMetadata.introspection_endpoint;

    if (!endpoint) {
      console.error("[auth] no introspection endpoint in metadata");
      throw new Error("No token verification endpoint available in metadata");
    }

    const params = new URLSearchParams({
      token: token,
      client_id: CONFIG.auth.clientId,
    });

    if (CONFIG.auth.clientSecret) {
      params.set("client_secret", CONFIG.auth.clientSecret);
    }

    let response: Response;
    try {
      response = await fetch(endpoint, {
        method: "POST",
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: params.toString(),
      });
    } catch (e) {
      console.error("[auth] introspection fetch threw", e);
      throw e;
    }

    if (!response.ok) {
      const txt = await response.text();
      console.error("[auth] introspection non-OK", { status: response.status });

      try {
        const obj = JSON.parse(txt);
        console.log(JSON.stringify(obj, null, 2));
      } catch {
        console.error(txt);
      }
      throw new Error(`Invalid or expired token: ${txt}`);
    }

    let data: any;
    try {
      data = await response.json();
    } catch (e) {
      const txt = await response.text();
      console.error("[auth] failed to parse introspection JSON", {
        error: String(e),
        body: txt,
      });
      throw e;
    }

    if (data.active === false) {
      throw new Error("Inactive token");
    }

    if (!data.aud) {
      throw new Error("Resource indicator (aud) missing");
    }

    const audiences: string[] = Array.isArray(data.aud) ? data.aud : [data.aud];
    const allowed = audiences.some((a) =>
      checkResourceAllowed({
        requestedResource: a,
        configuredResource: mcpServerUrl,
      }),
    );
    if (!allowed) {
      throw new Error(
        `None of the provided audiences are allowed. Expected ${mcpServerUrl}, got: ${audiences.join(", ")}`,
      );
    }

    return {
      token,
      clientId: data.client_id,
      scopes: data.scope ? data.scope.split(" ") : [],
      expiresAt: data.exp,
    };
  },
};
app.use(
  mcpAuthMetadataRouter({
    oauthMetadata,
    resourceServerUrl: mcpServerUrl,
    scopesSupported: ["mcp:tools"],
    resourceName: "MCP Demo Server",
  }),
);

const authMiddleware = requireBearerAuth({
  verifier: tokenVerifier,
  requiredScopes: [],
  resourceMetadataUrl: getOAuthProtectedResourceMetadataUrl(mcpServerUrl),
});

const transports: { [sessionId: string]: StreamableHTTPServerTransport } = {};

function createMcpServer() {
  const server = new McpServer({
    name: "example-server",
    version: "1.0.0",
  });

  server.registerTool(
    "add",
    {
      title: "Addition Tool",
      description: "Add two numbers together",
      inputSchema: {
        a: z.number().describe("First number to add"),
        b: z.number().describe("Second number to add"),
      },
    },
    async ({ a, b }) => ({
      content: [{ type: "text", text: `${a} + ${b} = ${a + b}` }],
    }),
  );

  server.registerTool(
    "multiply",
    {
      title: "Multiplication Tool",
      description: "Multiply two numbers together",
      inputSchema: {
        x: z.number().describe("First number to multiply"),
        y: z.number().describe("Second number to multiply"),
      },
    },
    async ({ x, y }) => ({
      content: [{ type: "text", text: `${x} × ${y} = ${x * y}` }],
    }),
  );

  return server;
}

const mcpPostHandler = async (req: express.Request, res: express.Response) => {
  const sessionId = req.headers["mcp-session-id"] as string | undefined;
  let transport: StreamableHTTPServerTransport;

  if (sessionId && transports[sessionId]) {
    transport = transports[sessionId];
  } else if (!sessionId && isInitializeRequest(req.body)) {
    transport = new StreamableHTTPServerTransport({
      sessionIdGenerator: () => randomUUID(),
      onsessioninitialized: (sessionId) => {
        transports[sessionId] = transport;
      },
    });

    transport.onclose = () => {
      if (transport.sessionId) {
        delete transports[transport.sessionId];
      }
    };

    const server = createMcpServer();
    await server.connect(transport);
  } else {
    res.status(400).json({
      jsonrpc: "2.0",
      error: {
        code: -32000,
        message: "Bad Request: No valid session ID provided",
      },
      id: null,
    });
    return;
  }

  await transport.handleRequest(req, res, req.body);
};

const handleSessionRequest = async (
  req: express.Request,
  res: express.Response,
) => {
  const sessionId = req.headers["mcp-session-id"] as string | undefined;
  if (!sessionId || !transports[sessionId]) {
    res.status(400).send("Invalid or missing session ID");
    return;
  }

  const transport = transports[sessionId];
  await transport.handleRequest(req, res);
};

app.post("/", authMiddleware, mcpPostHandler);
app.get("/", authMiddleware, handleSessionRequest);
app.delete("/", authMiddleware, handleSessionRequest);

app.listen(CONFIG.port, CONFIG.host, () => {
  console.log(`🚀 MCP Server running on ${mcpServerUrl.origin}`);
  console.log(`📡 MCP endpoint available at ${mcpServerUrl.origin}`);
  console.log(
    `🔐 OAuth metadata available at ${getOAuthProtectedResourceMetadataUrl(mcpServerUrl)}`,
  );
});
```

When you run the server, you can add it to your MCP client, such as Visual Studio Code, by providing the MCP server endpoint.For more details about implementing MCP servers in TypeScript, refer to the [TypeScript SDK documentation](https://github.com/modelcontextprotocol/typescript-sdk).

运行服务器后，你可以通过提供 MCP 服务器端点将其添加到你的 MCP 客户端（例如 Visual Studio Code）。有关在 TypeScript 中实现 MCP 服务器的更多详细信息，请参阅 [TypeScript SDK 文档](https://github.com/modelcontextprotocol/typescript-sdk)。

You can see the complete Python project in the [sample repository](https://github.com/localden/min-py-mcp-auth).To simplify our authorization interaction, in Python scenarios we rely on [FastMCP](https://gofastmcp.com/getting-started/welcome). Many of the conventions around authorization, like the endpoints and token validation logic, are consistent across languages, but some offer simpler ways of integrating them in production scenarios.Prior to writing the actual server, we need to set up our configuration in `config.py` - the contents are entirely based on your local server setup:

你可以在[示例仓库](https://github.com/localden/min-py-mcp-auth)中查看完整的 Python 项目。为了简化我们的授权交互，在 Python 场景中我们使用 [FastMCP](https://gofastmcp.com/getting-started/welcome)。许多与授权相关的约定（例如端点和令牌验证逻辑）在不同语言中是一致的，但有些语言提供了更简单的方式将它们集成到生产场景中。在编写实际服务器之前，我们需要在 `config.py` 中设置配置——其内容完全基于你的本地服务器设置：

```python
"""Configuration settings for the MCP auth server."""

import os
from typing import Optional


class Config:
    """Configuration class that loads from environment variables with sensible defaults."""

    # Server settings
    HOST: str = os.getenv("HOST", "localhost")
    PORT: int = int(os.getenv("PORT", "3000"))

    # Auth server settings
    AUTH_HOST: str = os.getenv("AUTH_HOST", "localhost")
    AUTH_PORT: int = int(os.getenv("AUTH_PORT", "8080"))
    AUTH_REALM: str = os.getenv("AUTH_REALM", "master")

    # OAuth client settings
    OAUTH_CLIENT_ID: str = os.getenv("OAUTH_CLIENT_ID", "mcp-server")
    OAUTH_CLIENT_SECRET: str = os.getenv("OAUTH_CLIENT_SECRET", "UO3rmozkFFkXr0QxPTkzZ0LMXDidIikB")

    # Server settings
    MCP_SCOPE: str = os.getenv("MCP_SCOPE", "mcp:tools")
    OAUTH_STRICT: bool = os.getenv("OAUTH_STRICT", "false").lower() in ("true", "1", "yes")
    TRANSPORT: str = os.getenv("TRANSPORT", "streamable-http")

    @property
    def server_url(self) -> str:
        """Build the server URL."""
        return f"http://{self.HOST}:{self.PORT}"

    @property
    def auth_base_url(self) -> str:
        """Build the auth server base URL."""
        return f"http://{self.AUTH_HOST}:{self.AUTH_PORT}/realms/{self.AUTH_REALM}/"

    def validate(self) -> None:
        """Validate configuration."""
        if self.TRANSPORT not in ["sse", "streamable-http"]:
            raise ValueError(f"Invalid transport: {self.TRANSPORT}. Must be 'sse' or 'streamable-http'")


# Global configuration instance
config = Config()
```

The server implementation is as follows:

服务器实现如下：

```python
import datetime
import logging
from typing import Any

from pydantic import AnyHttpUrl

from mcp.server.auth.settings import AuthSettings
from mcp.server.fastmcp.server import FastMCP

from .config import config
from .token_verifier import IntrospectionTokenVerifier

logger = logging.getLogger(__name__)


def create_oauth_urls() -> dict[str, str]:
    """Create OAuth URLs based on configuration (Keycloak-style)."""
    from urllib.parse import urljoin

    auth_base_url = config.auth_base_url

    return {
        "issuer": auth_base_url,
        "introspection_endpoint": urljoin(auth_base_url, "protocol/openid-connect/token/introspect"),
        "authorization_endpoint": urljoin(auth_base_url, "protocol/openid-connect/auth"),
        "token_endpoint": urljoin(auth_base_url, "protocol/openid-connect/token"),
    }


def create_server() -> FastMCP:
    """Create and configure the FastMCP server."""

    config.validate()

    oauth_urls = create_oauth_urls()

    token_verifier = IntrospectionTokenVerifier(
        introspection_endpoint=oauth_urls["introspection_endpoint"],
        server_url=config.server_url,
        client_id=config.OAUTH_CLIENT_ID,
        client_secret=config.OAUTH_CLIENT_SECRET,
    )

    app = FastMCP(
        name="MCP Resource Server",
        instructions="Resource Server that validates tokens via Authorization Server introspection",
        host=config.HOST,
        port=config.PORT,
        debug=True,
        streamable_http_path="/",
        token_verifier=token_verifier,
        auth=AuthSettings(
            issuer_url=AnyHttpUrl(oauth_urls["issuer"]),
            required_scopes=[config.MCP_SCOPE],
            resource_server_url=AnyHttpUrl(config.server_url),
        ),
    )

    @app.tool()
    async def add_numbers(a: float, b: float) -> dict[str, Any]:
        """
        Add two numbers together.
        This tool demonstrates basic arithmetic operations with OAuth authentication.

        Args:
            a: The first number to add
            b: The second number to add
        """
        result = a + b
        return {
            "operation": "addition",
            "operand_a": a,
            "operand_b": b,
            "result": result,
            "timestamp": datetime.datetime.now().isoformat()
        }

    @app.tool()
    async def multiply_numbers(x: float, y: float) -> dict[str, Any]:
        """
        Multiply two numbers together.
        This tool demonstrates basic arithmetic operations with OAuth authentication.

        Args:
            x: The first number to multiply
            y: The second number to multiply
        """
        result = x * y
        return {
            "operation": "multiplication",
            "operand_x": x,
            "operand_y": y,
            "result": result,
            "timestamp": datetime.datetime.now().isoformat()
        }

    return app
```

```python
def main() -> int:
    """
    Run the MCP Resource Server.

    This server:
    - Provides RFC 9728 Protected Resource Metadata
    - Validates tokens via Authorization Server introspection
    - Serves MCP tools requiring authentication

    Configuration is loaded from config.py and environment variables.
    """
    logging.basicConfig(level=logging.INFO)

    try:
        config.validate()
        oauth_urls = create_oauth_urls()

    except ValueError as e:
        logger.error("Configuration error: %s", e)
        return 1

    try:
        mcp_server = create_server()

        logger.info("Starting MCP Server on %s:%s", config.HOST, config.PORT)
        logger.info("Authorization Server: %s", oauth_urls["issuer"])
        logger.info("Transport: %s", config.TRANSPORT)

        mcp_server.run(transport=config.TRANSPORT)
        return 0

    except Exception:
        logger.exception("Server error")
        return 1


if __name__ == "__main__":
    exit(main())
```

Lastly, the token verification logic is delegated entirely to `token_verifier.py`, ensuring that we can use the Keycloak introspection endpoint to verify the validity of any credential artifacts

最后，令牌验证逻辑完全委托给 `token_verifier.py`，以确保我们可以使用 Keycloak 的内省端点来验证任何凭据工件的有效性。

```python
"""Token verifier implementation using OAuth 2.0 Token Introspection (RFC 7662)."""

import logging
from typing import Any

from mcp.server.auth.provider import AccessToken, TokenVerifier
from mcp.shared.auth_utils import check_resource_allowed, resource_url_from_server_url

logger = logging.getLogger(__name__)


class IntrospectionTokenVerifier(TokenVerifier):
    """Token verifier that uses OAuth 2.0 Token Introspection (RFC 7662).
    """

    def __init__(
        self,
        introspection_endpoint: str,
        server_url: str,
        client_id: str,
        client_secret: str,
    ):
        self.introspection_endpoint = introspection_endpoint
        self.server_url = server_url
        self.client_id = client_id
        self.client_secret = client_secret
        self.resource_url = resource_url_from_server_url(server_url)

    async def verify_token(self, token: str) -> AccessToken | None:
        """Verify token via introspection endpoint."""
        import httpx

        if not self.introspection_endpoint.startswith(("https://", "http://localhost", "http://127.0.0.1")):
            return None

        timeout = httpx.Timeout(10.0, connect=5.0)
        limits = httpx.Limits(max_connections=10, max_keepalive_connections=5)

        async with httpx.AsyncClient(
            timeout=timeout,
            limits=limits,
            verify=True,
        ) as client:
            try:
                form_data = {
                    "token": token,
                    "client_id": self.client_id,
                    "client_secret": self.client_secret,
                }
                headers = {"Content-Type": "application/x-www-form-urlencoded"}

                response = await client.post(
                    self.introspection_endpoint,
                    data=form_data,
                    headers=headers,
                )

                if response.status_code != 200:
                    return None

                data = response.json()
                if not data.get("active", False):
                    return None

                if not self._validate_resource(data):
                    return None

                return AccessToken(
                    token=token,
                    client_id=data.get("client_id", "unknown"),
                    scopes=data.get("scope", "").split() if data.get("scope") else [],
                    expires_at=data.get("exp"),
                    resource=data.get("aud"),  # Include resource in token
                )

            except Exception as e:
                return None

    def _validate_resource(self, token_data: dict[str, Any]) -> bool:
        """Validate token was issued for this resource server.

        Rules:
        - Reject if 'aud' missing.
        - Accept if any audience entry matches the derived resource URL.
        - Supports string or list forms per JWT spec.
        """
        if not self.server_url or not self.resource_url:
            return False

        aud: list[str] | str | None = token_data.get("aud")
        if isinstance(aud, list):
            return any(self._is_valid_resource(a) for a in aud)
        if isinstance(aud, str):
            return self._is_valid_resource(aud)
        return False

    def _is_valid_resource(self, resource: str) -> bool:
        """Check if the given resource matches our server."""
        return check_resource_allowed(self.resource_url, resource)
```

For more details, see the [Python SDK documentation](https://github.com/modelcontextprotocol/python-sdk).

有关更多详细信息，请参阅 [Python SDK 文档](https://github.com/modelcontextprotocol/python-sdk)。

You can see the complete C# project in the [sample repository](https://github.com/localden/min-cs-mcp-auth).To set up authorization in your MCP server using the MCP C# SDK, you can lean on the standard ASP.NET Core builder pattern. Instead of using the introspection endpoint provided by Keycloak, we will use built-in ASP.NET Core capabilities for token validation.

你可以在[示例仓库](https://github.com/localden/min-cs-mcp-auth)中查看完整的 C# 项目。要在 MCP 服务器中使用 MCP C# SDK 设置授权，你可以依赖标准的 ASP.NET Core 构建器模式。我们不使用 Keycloak 提供的内省端点，而是使用 ASP.NET Core 内置的令牌验证功能。

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using ModelContextProtocol.AspNetCore.Authentication;
using ProtectedMcpServer.Tools;
using System.Security.Claims;

var builder = WebApplication.CreateBuilder(args);

var serverUrl = "http://localhost:3000/";
var authorizationServerUrl = "http://localhost:8080/realms/master/";

builder.Services.AddAuthentication(options =>
{
    options.DefaultChallengeScheme = McpAuthenticationDefaults.AuthenticationScheme;
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.Authority = authorizationServerUrl;
    var normalizedServerAudience = serverUrl.TrimEnd('/');
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidIssuer = authorizationServerUrl,
        ValidAudiences = new[] { normalizedServerAudience, serverUrl },
        AudienceValidator = (audiences, securityToken, validationParameters) =>
        {
            if (audiences == null) return false;
            foreach (var aud in audiences)
            {
                if (string.Equals(aud.TrimEnd('/'), normalizedServerAudience, StringComparison.OrdinalIgnoreCase))
                {
                    return true;
                }
            }
            return false;
        }
    };

    options.RequireHttpsMetadata = false; // Set to true in production

    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = context =>
        {
            var name = context.Principal?.Identity?.Name ?? "unknown";
            var email = context.Principal?.FindFirstValue("preferred_username") ?? "unknown";
            Console.WriteLine($"Token validated for: {name} ({email})");
            return Task.CompletedTask;
        },
        OnAuthenticationFailed = context =>
        {
            Console.WriteLine($"Authentication failed: {context.Exception.Message}");
            return Task.CompletedTask;
        },
    };
})
.AddMcp(options =>
{
    options.ResourceMetadata = new()
    {
        Resource = new Uri(serverUrl),
        ResourceDocumentation = new Uri("https://docs.example.com/api/math"),
        AuthorizationServers = { new Uri(authorizationServerUrl) },
        ScopesSupported = ["mcp:tools"]
    };
});

builder.Services.AddAuthorization();

builder.Services.AddHttpContextAccessor();
builder.Services.AddMcpServer()
    .WithTools<MathTools>()
    .WithHttpTransport();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapMcp().RequireAuthorization();

Console.WriteLine($"Starting MCP server with authorization at {serverUrl}");
Console.WriteLine($"Using Keycloak server at {authorizationServerUrl}");
Console.WriteLine($"Protected Resource Metadata URL: {serverUrl}.well-known/oauth-protected-resource");
Console.WriteLine("Exposed Math tools: Add, Multiply");
Console.WriteLine("Press Ctrl+C to stop the server");

app.Run(serverUrl);
```

For more details, see the [C# SDK documentation](https://github.com/modelcontextprotocol/csharp-sdk).

有关更多详细信息，请参阅 [C# SDK 文档](https://github.com/modelcontextprotocol/csharp-sdk)。

## Testing the MCP Server / 测试 MCP 服务器


For testing purposes, we will be using [Visual Studio Code](https://code.visualstudio.com/), but any client that supports MCP and the new authorization specification will fit.
Press `Cmd` + `Shift` + `P` and select **MCP: Add server…**. Select **HTTP** and enter `http://localhost:3000`. Give the server a unique name to be used inside Visual Studio Code. In `mcp.json` you should now see an entry like this:

出于测试目的，我们将使用 [Visual Studio Code](https://code.visualstudio.com/)，但任何支持 MCP 和新授权规范的客户端都可以。
按 `Cmd` + `Shift` + `P`，然后选择 **MCP: Add server…**。选择 **HTTP** 并输入 `http://localhost:3000`。为服务器取一个在 Visual Studio Code 中使用的唯一名称。在 `mcp.json` 中，你现在应该看到类似这样的条目：

```json
"my-mcp-server-18676652": {
  "url": "http://localhost:3000",
  "type": "http"
}
```

On connection, you will be taken to the browser, where you will be prompted to consent to Visual Studio Code having access to the `mcp:tools` scope.

连接时，你会被带到浏览器，随后会提示你同意 Visual Studio Code 访问 `mcp:tools` 作用域。

After consenting, you will see the tools listed right above the server entry in `mcp.json`.

同意后，你会在 `mcp.json` 中的服务器条目上方看到列出的工具。

You will be able to invoke individual tools with the help of the `#` sign in the chat view.

你可以在聊天视图中借助 `#` 符号调用各个工具。

## Common Pitfalls and How to Avoid Them / 常见陷阱及如何避免


For comprehensive security guidance, including attack vectors, mitigation strategies, and implementation best practices, make sure to read through [Security Best Practices](/specification/draft/basic/security_best_practices). A few key issues are called out below.

如需全面的安全指导（包括攻击向量、缓解策略和实施最佳实践），请务必阅读[安全最佳实践](/specification/draft/basic/security_best_practices)。下面重点指出了几个关键问题。

* **Do not implement token validation or authorization logic by yourself**. Use off-the-shelf, well-tested, and secure libraries for things like token validation or authorization decisions. Doing everything from scratch means that you're more likely to implement things incorrectly unless you are a security expert.

  **不要自己实现令牌验证或授权逻辑**。对于令牌验证或授权决策，请使用现成的、经过充分测试的安全库。除非你是安全专家，否则从头开始实现更容易出错。

* **Use short-lived access tokens**. Depending on the authorization server used, this setting might be customizable. We recommend to not use long-lived tokens - if a malicious actor steals them, they will be able to maintain their access for longer periods.

  **使用短期访问令牌**。根据所使用的授权服务器，该设置可能是可自定义的。我们建议不要使用长期有效的令牌——如果恶意行为者窃取了它们，他们将能够在更长的时间内保持访问权限。

* **Always validate tokens**. Just because your server received a token does not mean that the token is valid or that it's meant for your server. Always verify that what your MCP server is getting from the client matches the required constraints.

  **始终验证令牌**。你的服务器收到令牌并不意味着该令牌有效或就是发给你的服务器的。请务必验证 MCP 服务器从客户端获得的令牌是否符合所需约束。

* **Store tokens in secure, encrypted storage**. In certain scenarios, you might need to cache tokens server-side. If that is the case, ensure that the storage has the right access controls and cannot be easily exfiltrated by malicious parties with access to your server. You should also implement robust cache eviction policies to ensure that your MCP server is not re-using expired or otherwise invalid tokens.

  **将令牌存储在安全的加密存储中**。在某些场景中，你可能需要在服务器端缓存令牌。如果是这样，请确保存储具备正确的访问控制，并且无法被能够访问你服务器的恶意方轻易窃取。你还应实现稳健的缓存逐出策略，以确保 MCP 服务器不会重复使用已过期或其他无效的令牌。

* **Enforce HTTPS in production**. Do not accept tokens or redirect callbacks over plain HTTP except for `localhost` during development.

  **在生产环境中强制使用 HTTPS**。除开发期间的 `localhost` 外，请勿通过纯 HTTP 接受令牌或重定向回调。

* **Least-privilege scopes**. Don't use catch‑all scopes. Split access per tool or capability where possible and verify required scopes per route/tool on the resource server.

  **最小权限作用域**。不要使用万能作用域。尽可能按工具或能力拆分层访问权限，并在资源服务器上针对每个路由/工具验证所需作用域。

* **Don't log credentials**. Never log `Authorization` headers, tokens, codes, or secrets. Scrub query strings and headers. Redact sensitive fields in structured logs.

  **不要记录凭据**。永远不要记录 `Authorization` 头、令牌、代码或密钥。清理查询字符串和头部。在结构化日志中隐去敏感字段。

* **Separate app vs. resource server credentials**. Don't reuse your MCP server's client secret for end‑user flows. Store all secrets in a proper secret manager, not in source control.

  **区分应用凭据与资源服务器凭据**。不要为最终用户流程重复使用 MCP 服务器的客户端密钥。将所有密钥存储在合适的密钥管理器中，而不是源代码控制中。

* **Return proper challenges**. On 401, include `WWW-Authenticate` with `Bearer`, `realm`, and `resource_metadata` so clients can discover how to authenticate.

  **返回正确的质询**。在 401 响应中，包含带有 `Bearer`、`realm` 和 `resource_metadata` 的 `WWW-Authenticate`，以便客户端发现如何进行身份验证。

* **DCR (Dynamic Client Registration) controls**. If enabled, be aware of constraints specific to your organization, such as trusted hosts, required vetting, and audited registrations. Unauthenticated DCR means that anyone can register any client with your authorization server.

  **DCR（动态客户端注册）控制**。如果启用了 DCR，请注意组织特定的约束，例如受信任的主机、必需的审查和经过审计的注册。未经身份验证的 DCR 意味着任何人都可以在你的授权服务器上注册任何客户端。

* **Multi‑tenant/realm mix-ups**. Pin to a single issuer/tenant unless explicitly multi‑tenant. Reject tokens from other realms even if signed by the same authorization server.

  **多租户/realm 混淆**。除非明确需要多租户，否则固定在单个颁发者/租户上。即使由同一个授权服务器签名，也要拒绝来自其他 realm 的令牌。

* **Audience/resource indicator misuse**. Don't configure or accept generic audiences (like `api`) or unrelated resources. Require the audience/resource to match your configured server.

  **受众/资源指示符滥用**。不要配置或接受通用受众（如 `api`）或无关资源。要求受众/资源必须与你配置的服务器匹配。

* **Error detail leakage**. Return generic messages to clients, but log detailed reasons with correlation IDs internally to aid troubleshooting without exposing internals.

  **错误详情泄露**。向客户端返回通用消息，但在内部使用关联 ID 记录详细原因，以便在不暴露内部信息的情况下协助故障排除。

* **Session identifier hardening**. Treat `Mcp-Session-Id` as untrusted input; never tie authorization to it. Regenerate on auth changes and validate lifecycle server‑side.

  **会话标识符加固**。将 `Mcp-Session-Id` 视为不受信任的输入；永远不要将授权绑定到它上面。在身份验证变更时重新生成，并在服务器端验证生命周期。

MCP authorization builds on these well-established standards:

MCP 授权基于以下成熟的标准构建：

* **[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13)**: The core authorization framework
* **[RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414)**: Authorization Server Metadata discovery
* **[RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591)**: Dynamic Client Registration
* **[RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)**: Protected Resource Metadata
* **[RFC 8707](https://datatracker.ietf.org/doc/html/rfc8707)**: Resource Indicators

* **[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13)**：核心授权框架
* **[RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414)**：授权服务器元数据发现
* **[RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591)**：动态客户端注册
* **[RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)**：受保护资源元数据
* **[RFC 8707](https://datatracker.ietf.org/doc/html/rfc8707)**：资源指示符

For additional details, refer to:

如需更多详细信息，请参阅：

* [Authorization Specification](/specification/draft/basic/authorization)
* [Security Best Practices](/specification/draft/basic/security_best_practices)
* [Available MCP SDKs](/docs/sdk)

* [授权规范](/specification/draft/basic/authorization)
* [安全最佳实践](/specification/draft/basic/security_best_practices)
* [可用的 MCP SDK](/docs/sdk)

Understanding these standards will help you implement authorization correctly and troubleshoot issues when they arise.

理解这些标准将有助于你正确地实现授权，并在出现问题时进行故障排除。
