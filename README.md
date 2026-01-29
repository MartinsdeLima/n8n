### Projeto básico para início de conhecimento sobre a plataforma n8n

- Esta implementação foi feita usando Docker Composer (previamente instalado) rodando em uma máquina Linux Ubuntu 24.04 com arquitetura AMDx86

- ## Instalação:
- Abaixo os passos básicos para instalação. O link com a documentação oficial segue ao final deste documento.
- Esta é a instalação básica para iniciar um projeto do zero. Os arquivos deste repositório foram alterados conforme a solução evouliu em meu ambiente.
  
  ## 1 - Crie um diretório para armazenar a configuração do n8n e os arquivos do Docker Compose e navegue:
      mkdir n8n-compose
      cd n8n-compose

  ## 2 - Dentro do diretório n8n-compose, crie um arquivo .env com o conteúdo a seguir. Ajuste as variáveis de acordo com suas necessidades: ##

```sh
   # DOMAIN_NAME and SUBDOMAIN together determine where n8n will be reachable from
   # The top level domain to serve from
   DOMAIN_NAME=example.com

   # The subdomain to serve from
   SUBDOMAIN=n8n

   # The above example serve n8n at: https://n8n.example.com

   # Optional timezone to set which gets used by Cron and other scheduling nodes
   # New York is the default value if not set
   GENERIC_TIMEZONE=Europe/Berlin

   # The email address to use for the TLS/SSL certificate creation
   SSL_EMAIL=user@example.com
```
  ## 3 - Crie uma nova pasta na raiz do projeto:
      mkdir local-files

  ## 4 - Crie o arquivo compose.yaml na pasta raiz do projeto com o conteúdo a seguir:
```sh
services:
  traefik:
    image: "traefik"
    restart: always
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.mytlschallenge.acme.tlschallenge=true"
      - "--certificatesresolvers.mytlschallenge.acme.email=${SSL_EMAIL}"
      - "--certificatesresolvers.mytlschallenge.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - traefik_data:/letsencrypt
      - /var/run/docker.sock:/var/run/docker.sock:ro

  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    labels:
      - traefik.enable=true
      - traefik.http.routers.n8n.rule=Host(`${SUBDOMAIN}.${DOMAIN_NAME}`)
      - traefik.http.routers.n8n.tls=true
      - traefik.http.routers.n8n.entrypoints=web,websecure
      - traefik.http.routers.n8n.tls.certresolver=mytlschallenge
      - traefik.http.middlewares.n8n.headers.SSLRedirect=true
      - traefik.http.middlewares.n8n.headers.STSSeconds=315360000
      - traefik.http.middlewares.n8n.headers.browserXSSFilter=true
      - traefik.http.middlewares.n8n.headers.contentTypeNosniff=true
      - traefik.http.middlewares.n8n.headers.forceSTSHeader=true
      - traefik.http.middlewares.n8n.headers.SSLHost=${DOMAIN_NAME}
      - traefik.http.middlewares.n8n.headers.STSIncludeSubdomains=true
      - traefik.http.middlewares.n8n.headers.STSPreload=true
      - traefik.http.routers.n8n.middlewares=n8n@docker
    environment:
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_HOST=${SUBDOMAIN}.${DOMAIN_NAME}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - N8N_RUNNERS_ENABLED=true
      - NODE_ENV=production
      - WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}/
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - TZ=${GENERIC_TIMEZONE}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./local-files:/files

volumes:
  n8n_data:
  traefik_data:
```
  ## 5 - Subindo o n8n (executar na pasta onde o arquivo compose.yaml se encontra):
     sudo docker compose up -d

  ## 6 - Parando a aplicação:
     sudo docker compose stop

  Fontes com mais detalhes e documentação da n8n:
  https://docs.n8n.io/hosting/installation/server-setups/docker-compose/#6-create-docker-compose-file
  
