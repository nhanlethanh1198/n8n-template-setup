# n8n with Qdrant and Postgres Setup (No Nginx)

This project sets up a Docker Compose stack with n8n (workflow automation), Qdrant (vector database for RAG), Postgres (database), and Traefik (reverse proxy for HTTPS) on a VPS without Nginx installed. Traefik uses default ports 80 (HTTP) and 443 (HTTPS) for simplicity. The setup is designed as a reusable template with dynamic ports and domains defined in a `.env` file, ideal for automation workflows with Retrieval-Augmented Generation (RAG).

## Prerequisites

- VPS with Docker and Docker Compose installed.
- No Nginx or other services running on ports 80 or 443.
- A domain name (e.g., `bubu.io.vn`) with a subdomain (e.g., `n8n.bubu.io.vn`) pointing to the VPS's public IP.
- Basic knowledge of Docker and DNS configuration.

## Features

- **n8n**: Workflow automation platform for creating RAG workflows.
- **Qdrant**: Vector database for storing and retrieving context for RAG.
- **Postgres**: Persistent database for n8n and Qdrant.
- **Traefik**: Reverse proxy for automatic HTTPS with Let's Encrypt (HTTP-01 challenge).
- **Dynamic Configuration**: Ports and domains defined in `.env` for reusability.
- **Persistent Storage**: Docker volumes for data persistence.
- **Future-Proofing**: n8n environment variables address deprecation warnings.

## Setup Instructions

1. **Clone or Create Project Files**  
   Create a project directory and add the following files:

   - `docker-compose.yml` (see project repository or copy from below).
   - `.env` (configuration file for environment variables).

2. **Configure DNS**  
   Ensure the subdomain (e.g., `n8n.bubu.io.vn`) has an A record pointing to your VPS's public IP (e.g., `192.168.1.1`).  
   Verify with:

   ```bash
   dig n8n.bubu.io.vn
   ```

   Allow 1-2 hours for DNS propagation.

3. **Create .env File**  
   Create a `.env` file in the project directory with the following content:

   ```env
   DOMAIN_NAME=yourdomain.com  # e.g., example.com
   SUBDOMAIN=n8n
   SSL_EMAIL=your@email.com  # Email for Let's Encrypt
   POSTGRES_DB=n8n
   POSTGRES_USER=n8n_user
   POSTGRES_PASSWORD=your_secure_password
   POSTGRES_HOST=postgres
   POSTGRES_PORT=5433  # Port for Postgres, change if needed
   N8N_USER=admin
   N8N_PASSWORD=your_secure_n8n_password
   TRAEFIK_HTTP_PORT=8080  # Port HTTP for Traefik, change if needed
   TRAEFIK_HTTPS_PORT=8443  # Port HTTPS for Traefik, change if needed
   ```

   Replace `your@email.com`, `your_secure_password`, and `your_secure_n8n_password` with your values.  
   Adjust `POSTGRES_PORT` if 5433 is in use by another Postgres instance.

4. **Verify Ports**  
   Ensure ports 80 and 443 are free:

   ```bash
   sudo netstat -tulnp | grep -E ":80|:443"
   ```

   If occupied, stop conflicting services or choose different ports (update `TRAEFIK_HTTP_PORT` and `TRAEFIK_HTTPS_PORT` in `.env`).

5. **Open Firewall Ports**  
   Allow traffic on ports 80 and 443:

   ```bash
   sudo ufw allow 80
   sudo ufw allow 443
   ```

   If using a VPS provider firewall (e.g., AWS, GCP), open these ports in their control panel.

6. **Deploy the Stack**  
   Run the Docker Compose stack:

   ```bash
   docker compose up -d
   ```

   Check container status:

   ```bash
   docker ps
   ```

   Verify Traefik logs for certificate issuance:

   ```bash
   docker compose logs traefik
   ```

7. **Access n8n**  
   Open `https://n8n.bubu.io.vn` in your browser (no port needed, as Traefik uses 443).  
   Log in with the credentials set in `N8N_USER` and `N8N_PASSWORD`.

8. **Configure RAG with n8n and Qdrant**  
   In n8n, create a workflow using Qdrant nodes to connect to `http://qdrant:8080`.  
   Use Qdrant to store and retrieve vectors for context.  
   Combine with n8n's AI nodes for Retrieval-Augmented Generation (RAG) workflows.

### Reusing the Template

To deploy this stack on another server or project:

- Copy `docker-compose.yml` and `.env` to the new project directory.
- Update `.env` with new values for:
  - `DOMAIN_NAME` and `SUBDOMAIN` for the new domain.
  - `SSL_EMAIL`, `POSTGRES_PASSWORD`, `N8N_PASSWORD` for security.
  - `POSTGRES_PORT` if needed to avoid conflicts.
- Ensure ports 80 and 443 are free on the new server.
- Configure DNS for the new subdomain.
- Run `docker compose up -d`.

## Troubleshooting

- **Port Conflicts**: If 80, 443, or 5433 are in use, check with:

  ```bash
  sudo lsof -i :80
  sudo lsof -i :443
  sudo lsof -i :5433
  ```

  Update `TRAEFIK_HTTP_PORT`, `TRAEFIK_HTTPS_PORT`, or `POSTGRES_PORT` in `.env`.

- **Certificate Errors**: If Let's Encrypt fails, check Traefik logs:

  ```bash
  docker compose logs traefik
  ```

  Test with Let's Encrypt staging by adding to Traefik's command in `docker-compose.yml`:

  ```yaml
  - "--certificatesresolvers.myhttpchallenge.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory"
  ```

  Remove this line after successful testing.

- **DNS Issues**: Ensure the subdomain resolves to the VPS's IP:
  ```bash
  dig n8n.bubu.io.vn
  ```

## Notes

- **Security**: Keep `.env` private and use strong passwords.
- **Qdrant Persistence**: Data is stored in the `qdrant_data` volume to prevent loss.
- **n8n Deprecations**: Environment variables in `docker-compose.yml` address n8n deprecation warnings for future compatibility.

## Further Assistance

For further assistance, refer to:

- [n8n Documentation](https://docs.n8n.io/)
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
