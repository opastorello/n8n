# Sistema de Automação e Atendimento

Este projeto configura um ambiente Docker Compose com serviços para automação, atendimento ao cliente, armazenamento, monitoramento e integração com WhatsApp. Abaixo está a documentação para configurar, executar e gerenciar o sistema.

## Visão Geral

O ambiente inclui os seguintes serviços:
- **Chatwoot**: Plataforma de atendimento ao cliente com suporte a chats em tempo real e integração com WhatsApp via Baileys API.
- **n8n**: Plataforma de automação de fluxos para integração de serviços.
- **Baileys API**: Integração com WhatsApp para envio e recebimento de mensagens.
- **PostgreSQL**: Banco de dados relacional (com suporte a `pgvector` para extensões vetoriais).
- **Redis**: Cache em memória e filas para Chatwoot e n8n.
- **RabbitMQ**: Sistema de mensagens para processamento assíncrono.
- **MinIO**: Armazenamento de objetos compatível com S3.
- **Prometheus**: Monitoramento de métricas.
- **Grafana**: Visualização de métricas e logs.
- **Loki**: Armazenamento de logs.
- **Promtail**: Coleta de logs para o Loki.
- **Ollama**: Execução de modelos de linguagem locais.
- **Adminer**: Interface web para gerenciamento do PostgreSQL.
- **Exporters**: `postgres_exporter`, `redis_exporter` e `node_exporter` para métricas específicas.

Todos os serviços estão conectados via rede `minha_rede` e usam volumes para persistência de dados.

## Pré-requisitos

- **Docker** e **Docker Compose** instalados.
- Pelo menos 8 GB de RAM e 4 CPUs recomendados para rodar todos os serviços.
- Sistema operacional compatível (Linux, Windows com WSL2, ou macOS com Docker Desktop).

## Configuração

1. **Clone o repositório ou copie os arquivos**:
   - `docker-compose.yml`
   - `prometheus.yml`
   - `loki-config.yml`
   - `promtail-config.yml`
   - `.env.example`

2. **Crie o arquivo `.env`**:
   - Copie o arquivo `.env.example` para `.env`:
     ```bash
     cp .env.example .env
     ```
   - Edite o `.env` com valores seguros:
     ```env
     # PostgreSQL
     SERVICE_USER_POSTGRES=postgres
     SERVICE_PASSWORD_POSTGRES=sua_senha_forte
     POSTGRES_DB=chatwoot_production

     # Redis
     SERVICE_PASSWORD_REDIS=sua_senha_forte

     # n8n
     N8N_USER=admin
     N8N_PASSWORD=sua_senha_forte
     WEBHOOK_URL=http://host.docker.internal:5678
     GENERIC_TIMEZONE=America/Sao_Paulo

     # Chatwoot
     SERVICE_PASSWORD_64_SECRETKEYBASE=sua_chave_secreta_64_caracteres
     MAILER_SENDER_EMAIL=support@seu_dominio.com
     SMTP_ADDRESS=smtp.seu_provedor.com
     SMTP_PORT=587
     SMTP_USERNAME=seu_usuario_smtp
     SMTP_PASSWORD=sua_senha_smtp
     FRONTEND_URL=http://localhost:3001
     BAILEYS_PROVIDER_DEFAULT_CLIENT_NAME=Chatwoot
     SERVICE_PASSWORD_64_BAILEYSDEFAULTAPIKEY=sua_chave_api_64_caracteres

     # MinIO
     MINIO_USER=minioadmin
     MINIO_PORT_ROOT_PASSWORD=sua_senha_forte

     # RabbitMQ
     RABBITMQ_USER=guest
     RABBITMQ_PASSWORD=sua_senha_forte

     # Grafana
     GRAFANA_USER=admin
     GRAFANA_PASSWORD=sua_senha_forte

     # Baileys API
     LOG_LEVEL=info
     BAILEYS_LOG_LEVEL=error
     ```
   - Gere chaves seguras para `SERVICE_PASSWORD_64_SECRETKEYBASE` e `SERVICE_PASSWORD_64_BAILEYSDEFAULTAPIKEY` (ex.: use `openssl rand -hex 32`).

3. **Crie o bucket no MinIO**:
   - Após iniciar os serviços, acesse `http://localhost:9001` e crie um bucket chamado `chatwoot` para armazenamento de arquivos do Chatwoot.

## Como Executar

1. **Inicie os serviços**:
   ```bash
   docker-compose up -d
   ```

2. **Verifique o status**:
   - Use `docker-compose ps` para confirmar que todos os serviços estão rodando.
   - Acesse os healthchecks via logs: `docker-compose logs <nome_do_servico>`.

3. **Acesse os serviços**:
   - **Chatwoot**: `http://localhost:3001` (configure a conta de administrador na primeira inicialização).
   - **n8n**: `http://localhost:5678` (use `N8N_USER` e `N8N_PASSWORD`).
   - **Adminer**: `http://localhost:8081` (gerenciamento do PostgreSQL).
   - **MinIO**: `http://localhost:9001` (use `MINIO_USER` e `MINIO_PORT_ROOT_PASSWORD`).
   - **Grafana**: `http://localhost:3000` (use `GRAFANA_USER` e `GRAFANA_PASSWORD`).
   - **Prometheus**: `http://localhost:9090` (verifique métricas em `/targets`).
   - **RabbitMQ**: `http://localhost:15672` (use `RABBITMQ_USER` e `RABBITMQ_PASSWORD`).
   - **Ollama**: `http://localhost:11434` (API para modelos de linguagem).
   - **Baileys API**: `http://localhost:3025` (integração com WhatsApp).

## Configuração do Grafana

1. **Adicione fontes de dados**:
   - **Prometheus**: URL `http://prometheus:9090`.
   - **Loki**: URL `http://loki:3100`.
2. **Crie dashboards**:
   - Visualize métricas do **node_exporter** (CPU, memória), **postgres_exporter** (banco de dados), e **redis_exporter** (cache).
   - Consulte logs com queries como `{job="chatwoot"}` ou `{job="n8n"}`.

## Notas para Produção

- **HTTPS**: Configure um proxy reverso (Nginx/Traefik) com certificados SSL para todos os serviços expostos (`Chatwoot`, `n8n`, `Grafana`, etc.).
- **Segurança**:
  - Substitua `host.docker.internal` em `WEBHOOK_URL` e `N8N_HOST` por um domínio público.
  - Restrinja `OLLAMA_ORIGINS` para origens específicas (não use `*`).
  - Crie vhosts específicos no **RabbitMQ** para cada serviço.
- **Monitoramento**:
  - Habilite endpoints `/metrics` no **Chatwoot** e **n8n**, se disponíveis, e atualize `prometheus.yml`.
  - Ajuste a retenção de logs no `loki-config.yml` (atualmente 14 dias).
- **Backup**:
  - Considere adicionar um serviço de backup para o **PostgreSQL** (ex.: `prodrigestivill/postgres-backup-local`).
- **Escalabilidade**:
  - Configure armazenamento externo (ex.: S3 com MinIO) para o **Loki**, se necessário.
  - Ajuste os recursos (CPU, RAM) alocados para cada serviço.

## Estrutura de Arquivos

- `docker-compose.yml`: Define os serviços, volumes e rede.
- `.env`: Variáveis de ambiente (crie a partir de `.env.example`).
- `prometheus.yml`: Configuração do Prometheus para coleta de métricas.
- `loki-config.yml`: Configuração do Loki para armazenamento de logs.
- `promtail-config.yml`: Configuração do Promtail para coleta de logs.

## Solução de Problemas

- **Serviço não inicia**: Verifique logs com `docker-compose logs <nome_do_servico>`.
- **Portas em conflito**: Ajuste as portas no `docker-compose.yml` se houver conflitos.
- **Métricas ausentes**: Conf全世界

Se precisar de ajuda adicional, entre em contato pelo e-mail de suporte ou abra uma issue no repositório.
