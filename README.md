Here's a descriptive `README.md` for your repository, incorporating the existing content, detailing the new API endpoints, and suggesting placements for your images.

---
<img width="1343" height="763" alt="cover" src="https://github.com/user-attachments/assets/8ff97782-5911-4d6f-bc10-04bd4c75fc4f" />

# X402 Modular Compute Protocol (MCP) Platform

This repository provides the core components and a platform for deploying and managing Modular Compute Protocols (MCPs) with X402 payments. It leverages Render for service deployments and Supabase for database management, offering a seamless experience for developers to publish and users to discover and utilize on-chain AI/compute services.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Using MCPs on the Platform](#using-mcps-on-the-platform)
  - [Adding X402 Sample MCPs](#adding-x402-sample-mcps)
- [API Endpoints](#api-endpoints)
  - [`/api/create-mcp` (POST)](#apicreate-mcp-post)
  - [`/api/deployment-status` (GET)](#apideployment-status-get)
  - [`/api/marketplace` (GET, PATCH)](#apimarketplace-get-patch)
  - [`/api/mcp-tools` (GET, POST)](#apimcp-tools-get-post)
  - [`/api/user-services` (GET)](#apiuser-services-get)
  - [`/api/user` (POST, GET)](#apiuser-post-get)
- [Frontend Components](#frontend-components)
- [Deployment Notes](#deployment-notes)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

The X402 MCP Platform simplifies the deployment and interaction with on-chain compute services, enabling a marketplace where users can discover and integrate various Modular Compute Protocols. It handles the complexities of service deployment, status tracking, and X402 payment flows, allowing developers to focus on building powerful MCPs.

## Features

- **MCP Deployment:** Easily deploy your GitHub-hosted MCPs as web services via the platform, leveraging Render's infrastructure.
- **X402 Payment Integration:** Automatic handling of X402 payment flows for invoking MCP tools.
- **Marketplace Discovery:** A centralized marketplace for users to browse, search, and integrate available MCPs.
- **Deployment Status Tracking:** Real-time monitoring of MCP deployment status (pending, building, live, failed).
- **User Management:** Basic user profile management and service association.
- **Tool Listing:** Fetches and displays available tools from deployed MCP servers.
- **Sample MCPs:** Includes examples like a YouTube clipping tool and a TODO list service to get started quickly.

## Getting Started

### Prerequisites

Before you begin, ensure you have the following:

- Node.js (v18 or higher)
- npm or Yarn
- Git
- A GitHub account
- A Render account (for `RENDER_OWNER_ID` and API key)
- A Supabase project for database management
- Access to Base Sepolia testnet funds (USDC/ETH) for X402 payments. Use the [Coinbase faucet](https://portal.cdp.coinbase.com/products/faucet) to obtain these.

### Using MCPs on the Platform

Follow these steps to enable MCPs with X402 payments in your client:

1.  **Clone the X402 MCP repo and build it**

    ```bash
    git clone https://github.com/Nirmal2000/x402-mcp.git
    cd x402-mcp
    npm run build
    ```

2.  **Configure your MCP client to use the client proxy**

    You can copy the MCP client config directly from the Marketplace UI. For reference, here is the manual form you can paste into your client config (e.g., Claude Desktop `claude_desktop_config.json`):

    ```json
    "youtube-video-clipper": {
      "command": "node",
      "args": [
        "[path to repo]/x402-mcp/dist/scripts/client-proxy.js"
      ],
      "env": {
        "PRIVATE_KEY": "your private key",
        "TARGET_URL": "the server url from the platform"
      }
    }
    ```

    -   `PRIVATE_KEY`: your wallet’s private key for payments.
    -   `TARGET_URL`: the MCP server URL provided by the platform (the tool endpoint).

    Once added, restart your MCP client. You can now invoke tools exposed by the server, and the proxy will handle X402 payment flows automatically.

    **Testnet funds:** Use the Coinbase faucet to obtain USDC/ETH on Base Sepolia for tool payments and gas:
    https://portal.cdp.coinbase.com/products/faucet

### Adding X402 Sample MCPs

You can try these sample MCPs with the platform:

1.  **YouTube clipping example**
    -   [https://github.com/Nirmal2000/youtube-video-clip](https://github.com/Nirmal2000/youtube-video-clip)
2.  **TODO sample** (same repo as the X402 MCP toolkit)
    -   [https://github.com/Nirmal2000/x402-mcp.git](https://github.com/Nirmal2000/x402-mcp.git)

Follow the instructions in each repository/website to configure environment variables and run the servers. The tools are already added to the platform. You can still test it out by deploying those MCPs to the platform. Instructions for deploying them are in their respective repositories READMEs.

---

## API Endpoints

This section describes the backend API routes that power the MCP platform.

### `/api/create-mcp` (POST)

This endpoint handles the creation and deployment of new MCP services. It integrates with Render to provision new web services and stores the deployment details in Supabase.

**Request Body:**

```json
{
  "userId": "string",             // Required: The ID of the user deploying the MCP
  "name": "string",               // Required: Name of the MCP service
  "repo": "string",               // Required: GitHub repository URL (e.g., "https://github.com/user/repo")
  "branch": "string",             // Optional: Git branch to deploy from (default: "main")
  "envVars": [{                   // Optional: Array of environment variables
    "key": "string",
    "value": "string"
  }],
  "buildCommand": "string",       // Optional: Build command for Render service
  "startCommand": "string",       // Optional: Start command for Render service
  "rootDir": "string",            // Optional: Root directory for the service
  "runtime": "string",            // Optional: Runtime environment (e.g., "node")
  "plan": "string"                // Optional: Render service plan (e.g., "starter")
}
```

**Response:**

```json
{
  "service": {
    "id": "string",                // Internal database ID of the MCP
    "name": "string",
    "ownerId": "string",           // Render owner ID
    "repo": "string",
    "branch": "string",
    "rootDir": "string",
    "createdAt": "datetime",
    "updatedAt": "datetime",
    "type": "web_service",
    "serviceDetails": {
      "url": "string",             // Deployed service URL
      "buildCommand": "string",
      "startCommand": "string",
      "buildPlan": "string",
      "env": "string"
    }
  },
  "deployId": "string"             // Render deployment ID
}
```

**Error Codes:**
- `400 Bad Request`: Missing required fields, invalid GitHub URL, or missing user ID.
- `500 Internal Server Error`: Render API errors, missing `RENDER_OWNER_ID`, or database storage failures.

Here's an example of the form used to create a new MCP: 

<img width="1845" height="994" alt="connect_wallet" src="https://github.com/user-attachments/assets/732e1f88-83f6-4708-b147-097024f39d02" />

<img width="1024" height="1024" alt="create_new_mcp" src="https://github.com/user-attachments/assets/fb4ebd1f-3e8d-4ede-a131-2c29e902ea09" />



### `/api/deployment-status` (GET)

This endpoint provides the real-time deployment status of a specific MCP from the Render API.

**Query Parameters:**
- `serviceId`: (Required) The Render service ID associated with the MCP.
- `deployId`: (Required) The Render deployment ID for the specific deployment.

**Response:**

```json
{
  "status": "string" // e.g., "build_in_progress", "live", "failed"
}
```

**Error Codes:**
- `400 Bad Request`: Missing `serviceId` or `deployId`.
- `500 Internal Server Error`: Render API call failure.

### `/api/marketplace` (GET, PATCH)

This endpoint manages the MCP marketplace, allowing retrieval of all deployed MCPs and updating their deployment statuses.

#### `GET /api/marketplace`

Retrieves a list of all MCPs stored in the database, including their current status and fetched tool information for 'live' services.

**Response:**

```json
{
  "mcps": [
    {
      "id": "string",
      "user_id": "string",
      "name": "string",
      "repo": "string",
      "branch": "string",
      "build_command": "string",
      "start_command": "string",
      "root_dir": "string",
      "runtime": "string",
      "env_vars": [],
      "plan": "string",
      "deploy_id": "string",
      "deploy_url": "string", // URL where the MCP is deployed
      "render_service_id": "string",
      "status": "string",     // Current deployment status
      "input_json": {},       // Original input for deployment
      "output_json": {},      // Raw response from Render API
      "created_at": "datetime",
      "updated_at": "datetime",
      "tools": [              // (Only for live MCPs) Array of tools provided by the MCP
        {
          "name": "string",
          "description": "string",
          "parameters": {},
          "endpoint": "string"
        }
      ],
      "description": "string",// (Only for live MCPs) Description from MCP tools API
      "environment_variables": {},// (Only for live MCPs) Env vars required by MCP
      "pricing": 0            // Placeholder for future pricing integration
    }
  ]
}
```

Here's a view of the marketplace showing various deployed MCPs: 

<img width="1024" height="1024" alt="choose_agent" src="https://github.com/user-attachments/assets/09676f86-de5b-4727-a21f-e338b1c046d3" />

<img width="1824" height="994" alt="agents" src="https://github.com/user-attachments/assets/c24306b4-e92a-4c34-8065-48766a1c789e" />



#### `PATCH /api/marketplace`

Updates the deployment status of a specific MCP by checking the Render API and persisting any changes to the database.

**Request Body:**

```json
{
  "id": "string" // Required: The internal database ID of the MCP to update
}
```

**Response:**

```json
{
  "mcp": { ... },           // The updated MCP object
  "statusChanged": boolean, // True if the status was updated, false otherwise
  "oldStatus": "string",    // Previous status
  "newStatus": "string"     // New status
}
```

**Error Codes:**
- `400 Bad Request`: Missing `id`, or `render_service_id`/`deploy_id` are missing in the MCP record.
- `404 Not Found`: MCP with the given `id` not found.
- `500 Internal Server Error`: Render API call failure or database update error.

### `/api/mcp-tools` (GET, POST)

This endpoint is used to fetch the list of tools exposed by one or more live MCP servers.

#### `GET /api/mcp-tools`

**Query Parameters:**
- `url`: (Optional) The base URL of a single MCP server.
- `urls`: (Optional) A comma-separated list of MCP server URLs.

**Response (single URL):**

```json
{
  "info": {
    "name": "string",
    "description": "string",
    "environmentVariables": {} // Environment variables required by the MCP
  },
  "tools": [
    {
      "name": "string",
      "description": "string",
      "parameters": {}, // JSON schema for tool parameters
      "endpoint": "string" // Relative path to the tool endpoint
    }
  ]
}
```

**Response (multiple URLs):**

```json
[
  {
    "url": "string",
    "info": { ... },
    "tools": [ ... ]
  },
  // ... for each URL
]
```

#### `POST /api/mcp-tools`

**Request Body:**

```json
{
  "urls": ["string", "string"] // Required: An array of MCP server URLs
}
```

**Response:**

(Same as `GET` with multiple URLs)

**Error Codes:**
- `400 Bad Request`: Missing `url`/`urls` parameter or invalid `urls` array in POST.
- `500 Internal Server Error`: Failure to fetch tools from the MCP server.

### `/api/user-services` (GET)

This endpoint provides a summary of services associated with the authenticated user, along with statistics.

**Response:**

```json
{
  "user": {
    "id": "string",
    "data": {}, // Additional user data
    "stats": {
      "totalServices": number,
      "liveServices": number,
      "buildingServices": number,
      "failedServices": number
    }
  },
  "services": [], // Array of services associated with the user
  "summary": {
    "total": number,
    "live": number,
    "building": number,
    "failed": number
  }
}
```

**Error Codes:**
- `500 Internal Server Error`: Failure to retrieve user or service data.

### `/api/user` (POST, GET)

This endpoint handles user creation and retrieval.

#### `POST /api/user`

Creates a new user record or updates an existing one in Supabase.

**Request Body:**

```json
{
  "user_id": "string",    // Required: Unique ID for the user
  "email": "string",      // Optional: User's email
  "username": "string",   // Optional: User's username
  "avatar_url": "string", // Optional: URL to user's avatar
  "metadata": {}          // Optional: Arbitrary JSON metadata
}
```

**Response:**

```json
{
  "user": {
    "user_id": "string",
    "email": "string",
    "username": "string",
    "avatar_url": "string",
    "metadata": {},
    "created_at": "datetime",
    "updated_at": "datetime"
  }
}
```

#### `GET /api/user`

Retrieves a user's profile information.

**Query Parameters:**
- `user_id`: (Required) The unique ID of the user.

**Response:**

```json
{
  "user": {
    "user_id": "string",
    "email": "string",
    "username": "string",
    "avatar_url": "string",
    "metadata": {},
    "created_at": "datetime",
    "updated_at": "datetime"
  }
}
```

**Error Codes:**
- `400 Bad Request`: Missing `user_id` parameter.
- `404 Not Found`: User not found.
- `500 Internal Server Error`: Database error.

---

## Frontend Components

The platform includes several React components to provide a user-friendly interface:

-   **`MarketplacePage.js`**: The main page for discovering MCPs. It fetches available MCPs, displays their details, and provides search and filtering capabilities. It also polls for status updates of deploying MCPs to provide real-time feedback.
    
    Here's a close-up of an individual MCP card in the marketplace, showing its tools and status:

    <img width="1024" height="1024" alt="sentiment_analyzer" src="https://github.com/user-attachments/assets/1e9f876c-8d0d-470d-babd-e848de7d2759" />
    

