# Nilabiru Data Hub FRPS

A lightweight, self-hosted FRP server for the Nilabiru ecosystem, exposing internal services to the public internet via secure reverse tunneling.

---

## Overview

**Nilabiru Data Hub Frps** runs an [FRP](https://github.com/fatedier/frp) server (`frps`) that acts as the public-facing tunnel endpoint for the Nilabiru infrastructure. FRP clients (`frpc`) running on internal servers connect to this instance and expose their local services through it — allowing HTTP, HTTPS, and raw TCP/UDP traffic to be routed publicly without requiring the internal servers to have a public IP.

---

## Services

| Service                      | Image                    | Port(s)                     | Description                                                                                                      |
| ---------------------------- | ------------------------ | --------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **nilabiru-frps**            | `fatedier/frps:v0.69.1`  | `7000`, `80`, `443`, `7500` | FRP server; port `7000` accepts frpc connections, `80`/`443` handle proxied traffic, `7500` is the web dashboard |
| **nilabiru-portainer-agent** | `portainer/agent:latest` | `9001`                      | Portainer agent for remote Docker management                                                                     |

---

## Requirements

- Docker Engine `20.10+`
- Docker Compose `v2+`
- A server with a **public IP address**
- Ports `7000`, `80`, `443`, `7500`, and `9001` open on the host firewall

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/andry-pebrianto/nilabiru-data-hub-frps.git
cd nilabiru-data-hub-frps
```

### 2. Configure environment variables

Create a `.env` file in the repository root:

```env
FRP_TOKEN=your_frps_token
FRP_USER=your_dashboard_username
FRP_PASSWORD=your_dashboard_password
```

> **Note:** The `FRP_TOKEN` value must match the `FRP_TOKEN` configured on every `frpc` client that connects to this server.
> **Note:** Never commit `.env` with real secrets — it is already listed in `.gitignore`.

### 3. Configure frps

Create a `frps.toml` file in the repository root:

```toml
bindPort = 7000
auth.token = "{{ .Envs.FRP_TOKEN }}"

webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "{{ .Envs.FRP_USER }}"
webServer.password = "{{ .Envs.FRP_PASSWORD }}"
```

> **Note:** `frps.toml` uses Go template syntax to read values from environment variables — secrets stay in `.env`, not in this file.
> **Note:** `frps.toml` is mounted read-only into the container.

### 4. Start the stack

```bash
docker compose up -d
```

To verify the services are running:

```bash
docker compose ps
docker compose logs -f nilabiru-frps
```

---

## Service Access

| Endpoint           | Description                                              |
| ------------------ | -------------------------------------------------------- |
| `<PUBLIC_IP>:7000` | frpc client connection port                              |
| `<PUBLIC_IP>:80`   | Proxied HTTP traffic from connected frpc clients         |
| `<PUBLIC_IP>:443`  | Proxied HTTPS traffic from connected frpc clients        |
| `<PUBLIC_IP>:7500` | FRP web dashboard (login with `FRP_USER`/`FRP_PASSWORD`) |
| `<PUBLIC_IP>:9001` | Portainer agent endpoint                                 |

---

## Data Persistence

| Mount                     | Description                                |
| ------------------------- | ------------------------------------------ |
| `./frps.toml`             | frps configuration (bind mount, read-only) |
| `/var/run/docker.sock`    | Docker socket for Portainer agent          |
| `/var/lib/docker/volumes` | Docker volumes for Portainer agent         |

There are no stateful volumes — frps itself is stateless.

---

## License

This project is licensed under the [MIT License](LICENSE).  
Copyright © 2026 Andry Pebrianto