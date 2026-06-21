# Nilabiru Data Hub Frps

A lightweight, self-hosted FRP server for the Nilabiru ecosystem, exposing internal services to the public internet via secure reverse tunneling.

---

## Overview

**Nilabiru Data Hub Frps** runs an [FRP](https://github.com/fatedier/frp) server (`frps`) that acts as the public-facing tunnel endpoint for the Nilabiru infrastructure. FRP clients (`frpc`) running on internal servers connect to this instance and expose their local services through it — allowing HTTP, HTTPS, and raw TCP/UDP traffic to be routed publicly without requiring the internal servers to have a public IP.

---

## Services

| Service           | Image                   | Port(s)             | Description                                                                                                 |
| ----------------- | ----------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------- |
| **nilabiru-frps** | `fatedier/frps:v0.69.1` | `7000`, `80`, `443` | FRP server; port `7000` accepts frpc client connections, ports `80`/`443` handle proxied HTTP/HTTPS traffic |

---

## Requirements

- Docker Engine `20.10+`
- Docker Compose `v2+`
- A server with a **public IP address**
- Ports `7000`, `80`, and `443` open/forwarded on the host firewall

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/andry-pebrianto/nilabiru-data-hub-frps.git
cd nilabiru-data-hub-frps
```

### 2. Configure frps

Create a `frps.toml` file in the repository root. At minimum, configure the bind port and an authentication token:

```toml
bindPort = 7000
auth.token = "your_frps_token"
```

> **Note:** The `auth.token` value here must match the `FRP_TOKEN` configured on every `frpc` client that connects to this server.

> **Note:** `frps.toml` may contain secrets (auth token). Add it to `.gitignore` or manage it via a secrets manager — never commit it with a real token.

### 3. Start the stack

```bash
docker compose up -d
```

To verify the service is running:

```bash
docker compose ps
docker compose logs -f nilabiru-frps
```

---

## Service Access

| Endpoint           | Description                                       |
| ------------------ | ------------------------------------------------- |
| `<PUBLIC_IP>:7000` | frpc client connection port                       |
| `<PUBLIC_IP>:80`   | Proxied HTTP traffic from connected frpc clients  |
| `<PUBLIC_IP>:443`  | Proxied HTTPS traffic from connected frpc clients |

---

## Data Persistence

| Mount         | Description                                |
| ------------- | ------------------------------------------ |
| `./frps.toml` | frps configuration (bind mount, read-only) |

There are no stateful volumes — frps itself is stateless.

---

## License

This project is licensed under the [MIT License](LICENSE).  
Copyright © 2026 Andry Pebrianto