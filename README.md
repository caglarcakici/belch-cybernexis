# Belch - Burp Suite REST API Extension

![Java](https://img.shields.io/badge/Java-17+-blue.svg)
![Burp Suite Pro](https://img.shields.io/badge/Burp%20Suite%20Pro-Required-orange.svg)

REST API extension for Burp Suite Professional providing programmatic access to proxy traffic, active scanner, scope management, and collaborative testing workflows. Designed for security teams implementing scripted testing workflows and custom tooling integrations.

**⚠️ Requires Burp Suite Professional - This is an extension, not a standalone tool.**

## Table of Contents

- [Features](#features)
- [Enhancements in this version](#enhancements-in-this-version)
- [Installation & Setup](#installation--setup)
- [API Authentication](#api-authentication)
- [Quick Start Examples](#quick-start-examples)
- [API Documentation](#api-documentation)
- [Requirements](#requirements)
- [Building from Source](#building-from-source)
- [Troubleshooting](#troubleshooting)
- [Acknowledgments](#acknowledgments)
- [License](#license)

## Features

**API Coverage**
- Proxy traffic management: search, filter, export HTTP requests/responses
- Scanner operations: trigger scans, retrieve vulnerability findings
- Scope configuration: programmatic include/exclude URL management
- Burp Collaborator integration for out-of-band testing
- Session tracking and traffic organization

**Integration Options**
- REST API with OpenAPI 3.0 specification
- WebSocket endpoints for real-time traffic streaming
- HAR and CSV export formats
- cURL command generation for request replay

**Operational Features**
- SQLite-based traffic storage with full-text search
- Request/response body handling up to 50MB
- Concurrent scan management and task tracking
- Session-based traffic filtering and organization

---

## Enhancements in this version

This fork adds the following improvements on top of the original Belch:

### API authentication (optional)
- **Token-based protection**: When enabled, all API endpoints (except `/`, `/health`, `/version`, `/docs`, `/openapi`, `/postman`) require a valid token.
- **Headers**: Send `X-Belch-Token: <token>` or `Authorization: Bearer <token>`.
- Configurable via **Belch Config** tab or `~/.belch/extension.properties` (`api.auth.enabled`, `api.auth.token`).

### Configuration UI (Belch Config tab)
- **API Port**: Change the API server port (e.g. 3333, 8000). Restart extension to apply.
- **API Authentication**:
  - Checkbox: *Require API token for requests*
  - **API Token** field (masked by default)
  - **Show / Hide**: Reveal token to read or copy
  - **Copy**: Copy token to clipboard
  - **Auto-Generate**: Generate a cryptographically strong random token (256-bit, URL-safe Base64)
- **Database path**, **Session tag**, **Verbose logging** as before.
- Settings are saved to `~/.belch/extension.properties` and **override** classpath defaults (port and auth apply after save; port change requires extension reload).

### Robustness and UX
- **Port in use**: If the configured port is already in use, a clear error message points to editing `~/.belch/extension.properties` and suggests alternative ports (e.g. 7850, 8000). On macOS, port 5000 is often used by AirPlay.
- **Project name NPE**: Safe handling when Burp project is not yet loaded; uses a fallback project name so database and queue metrics keep working.
- **Auth failure handling**: Invalid or missing token returns **401** and stops request processing (no scan or side effects).

---

## Installation & Setup

### Step 1: Download the extension

```bash
git clone https://github.com/cybernexis/belch.git
cd belch
mvn clean package
```

The JAR will be at `target/belch-1.0.1.jar`.

### Step 2: Load in Burp Suite Professional

1. Open **Burp Suite Professional**
2. **Extensions** → **Installed** → **Add**
3. Select **Java**, choose `belch-1.0.1.jar`
4. Click **Next** and **Close**

### Step 3: Configure (optional)

1. Open the **Belch Config** tab in Burp
2. Set **API Port** if you need something other than the default (e.g. 7850)
3. To enable API authentication: check *Require API token for requests*, set or **Auto-Generate** a token, then **Save Configuration**

### Step 4: Verify

The API runs on the port shown in Belch Config (default 7850). Examples:

- API base: `http://localhost:7850` (or your port)
- Docs: `http://localhost:7850/docs`
- Health: `http://localhost:7850/health`
- WebSocket: `ws://localhost:7850/ws/stream`

---

## API Authentication

If *Require API token for requests* is enabled in Belch Config:

**With token (example: port 3333)**

```bash
curl -X POST "http://localhost:3333/scanner/scan-url-list" \
  -H "Content-Type: application/json" \
  -H "X-Belch-Token: YOUR_TOKEN" \
  -d '{"urls": ["https://example.com/"]}'
```

**Without token** → `401 Unauthorized` with `"message": "Missing or invalid API token"`.

Public endpoints (no token): `/`, `/health`, `/version`, `/docs`, `/openapi`, `/postman`.

---

## Quick Start Examples

**Health check**
```bash
curl http://localhost:7850/health
```

**Proxy traffic search**
```bash
curl "http://localhost:7850/proxy/search?host=example.com&method=POST&limit=10"
```

**Submit URL for scanning** (add `-H "X-Belch-Token: YOUR_TOKEN"` if auth is enabled)
```bash
curl -X POST http://localhost:7850/scanner/scan-url-list \
  -H "Content-Type: application/json" \
  -d '{"urls": ["https://example.com/api"]}'
```

**Scan level** (optional): `audit_config` can be `LEGACY_PASSIVE_AUDIT_CHECKS` (lighter) or `LEGACY_ACTIVE_AUDIT_CHECKS` (default, deeper).

**Retrieve scan results**
```bash
curl http://localhost:7850/scanner/issues
```

---

## API Documentation

- **Interactive docs**: `http://localhost:7850/docs`
- **OpenAPI spec**: `http://localhost:7850/openapi`
- **Postman**: `http://localhost:7850/postman`

---

## Requirements

- Burp Suite Professional 2023.1 or later
- Java 17+ (build and run)
- Maven 3.6+ (build)
- Port of your choice available (default 7850)

---

## Building from Source

```bash
git clone https://github.com/cybernexis/belch.git
cd belch
mvn clean package
```

Output: `target/belch-1.0.1.jar`.

---

## Troubleshooting

- **Port already in use**: Change `api.port` in `~/.belch/extension.properties` (e.g. to 7850 or 8000) and reload the extension. On macOS, avoid port 5000 (often used by AirPlay).
- **API connection refused**: Ensure Burp is running with the extension loaded and the configured port is free.
- **401 Unauthorized**: Enable auth in Belch Config and send `X-Belch-Token` or `Authorization: Bearer <token>`.

---

## Acknowledgments

**Original Belch**

This project is based on [Belch](https://github.com/campbellcharlie/belch) by **[Charlie Campbell](https://github.com/campbellcharlie)**. Thank you for building and sharing the Burp REST API extension and for the clear, well-structured codebase.

Special thanks also to [Phil Thomas](https://github.com/fz42net) for the name “Belch”—it fits the extension perfectly.

**This fork**

The enhancements in this repository (API authentication, configuration UI with port/token/auto-generate/copy, and related robustness fixes) were added to make Belch easier to secure and operate in our environment. We are grateful to the original author and the community around Belch.

---

## License

MIT License - see [LICENSE](LICENSE).

**Disclaimer**  
Burp Suite is a trademark of PortSwigger Ltd. This project is an independent, third-party extension and is not affiliated with, endorsed by, or sponsored by PortSwigger Ltd. Use only in controlled environments and in compliance with PortSwigger’s license terms.
