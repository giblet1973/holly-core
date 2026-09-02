# Security notes for this deployment

This repository is intended for a local development machine and is not hardened for direct internet exposure.

## Current exposure model

- **Open WebUI**: Exposed on port 3000 for development access.
- **Grafana**: Exposed on port 3001 for development access.
- **Ollama**: Exposed on port 11434 for development access and reachable by Docker service names.
- **NVIDIA Exporter**: Intentionally internal-only and reachable only by Docker service name.
## Secrets

- Do not commit real application secrets.
- Use environment variables or CI/CD secret stores for secrets.
- Keep the real `.env` file local and ignored by Git.

## Before production deployment

If this stack is moved to a production or shared-host environment, tighten it in the following ways:

1. Bind WebUI and Grafana to localhost or a trusted proxy/VPN only.
2. Put Open WebUI behind HTTPS and authentication.
3. Keep Ollama and the exporter off host ports.
4. Store `WEBUI_SECRET_KEY` in your deployment secret manager, not in source control.
5. Review firewall rules and restrict inbound ports to trusted hosts only.

## Example secret

See [.env.example](.env.example) for the format of the expected secret.

