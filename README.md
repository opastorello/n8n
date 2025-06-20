# Sistema de Atendimento ao Cliente e Monitoramento

Este projeto configura uma stack completa para atendimento ao cliente, automação de fluxos, armazenamento de dados e monitoramento utilizando Docker Compose. A seguir, apresentamos uma visão geral do sistema, instruções de configuração e execução.

## Visão Geral

O sistema integra as seguintes ferramentas:

- **Chatwoot**: Plataforma de atendimento ao cliente com suporte a chats em tempo real (interface web e workers Sidekiq).
- **PostgreSQL**: Banco de dados relacional com suporte a vetores (pgvector).
- **Redis**: Cache em memória para processamento assíncrono.
- **RabbitMQ**: Sistema de mensagens para filas.
- **Baileys API**: Integração com WhatsApp.
- **n8n**: Plataforma de automação de fluxos de trabalho.
- **Adminer**: Interface web para gerenciamento do PostgreSQL.
- **Prometheus**: Sistema de monitoramento de métricas.
- **Grafana**: Visualização de métricas e logs.
- **Loki e Promtail**: Sistema de coleta e armazenamento de logs.
- **Node Exporter e Exporters**: Exportadores de métricas para PostgreSQL e Redis.
- **MinIO**: Armazenamento de objetos compatível com S3.

Todos os serviços são configurados para rodar em uma rede Docker chamada `minha_rede`, com persistência de dados via volumes.

## Pré-requisitos

- **Docker** e **Docker Compose** instalados.
- Um arquivo `.env` com as variáveis de ambiente necessárias (veja a seção abaixo).

## Configuração

1. **Crie o arquivo `.env`**:
   Crie um arquivo `.env` na raiz do projeto com as seguintes variáveis. Substitua os valores conforme necessário:

   ```env
   # Configurações gerais
   SERVICE_PASSWORD_64_SECRETKEYBASE=insira_uma_chave_secreta_de_64_caracteres
   SERVICE_PASSWORD_64_BAILEYSDEFAULTAPIKEY=insira_uma_chave_secreta_de_64_caracteres
   GENERIC_TIMEZONE=America/Sao_Paulo
   FRONTEND_URL=http://localhost:3001
   WEBHOOK_URL=http://localhost:5678
   LOG_LEVEL=info
   BAILEYS_LOG_LEVEL=error

   # PostgreSQL
   SERVICE_USER_POSTGRES=chatwoot
   SERVICE_PASSWORD_POSTGRES=sua_senha_segura
   POSTGRES_DB=chatwoot_production

   # Redis
   SERVICE_PASSWORD_REDIS=sua_senha_segura

   # RabbitMQ
   RABBITMQ_USER=guest
   RABBITMQ_PASSWORD=sua_senha_segura

   # SMTP (e-mail)
   MAILER_SENDER_EMAIL=seu_email@dominio.com
   SMTP_ADDRESS=smtp.seu_provedor.com
   SMTP_PORT=587
   SMTP_USERNAME=seu_usuario_smtp
   SMTP_PASSWORD=sua_senha_smtp

   # n8n
   N8N_USER=admin
   N8N_PASSWORD=sua_senha_segura

   # Grafana
   GRAFANA_USER=admin
   GRAFANA_PASSWORD=sua_senha_segura

   # MinIO
   MINIO_USER=admin
   MINIO_PORT_ROOT_PASSWORD=sua_senha_segura
   ```

2. **Arquivos de configuração adicionais**:
   - Crie o arquivo `prometheus.yml` para o Prometheus com as configurações de scraping.
   - Crie o arquivo `loki-config.yml` para o Loki.
   - Crie o arquivo `promtail-config.yml` para o Promtail.

   Exemplo básico para `prometheus.yml`:

   ```yaml
   global:
     scrape_interval: 15s
   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
     - job_name: 'postgres'
       static_configs:
         - targets: ['postgres_exporter:9187']
     - job_name: 'redis'
       static_configs:
         - targets: ['redis_exporter:9121']
     - job_name: 'node'
       static_configs:
         - targets: ['node_exporter:9100']
   ```

   Consulte a documentação oficial de Loki e Promtail para configurar `loki-config.yml` e `promtail-config.yml`.

## Executando o Projeto

1. **Inicie os serviços**:
   Na raiz do projeto, execute:

   ```bash
   docker compose up -d
   ```

2. **Acesse os serviços**:
   - **Chatwoot**: `http://localhost:3001`
   - **n8n**: `http://localhost:5678`
   - **Adminer**: `http://localhost:8081`
   - **Prometheus**: `http://localhost:9090`
   - **Grafana**: `http://localhost:3000`
   - **Loki**: `http://localhost:3100`
   - **RabbitMQ**: `http://localhost:15672`
   - **MinIO**: `http://localhost:9001` (console) e `http://localhost:9000` (API)

3. **Verifique os logs**:
   Para verificar os logs de um serviço específico:

   ```bash
   docker compose logs <nome_do_servico>
   ```

## Monitoramento e Logs

- **Grafana**: Configure dashboards para visualizar métricas do Prometheus e logs do Loki.
- **Prometheus**: Monitore métricas do PostgreSQL, Redis e do host.
- **Loki/Promtail**: Coleta logs dos containers e do sistema.

## Parando o Projeto

Para parar todos os serviços:

```bash
docker compose down
```

Para parar e remover os volumes (cuidado, isso apagará os dados persistentes):

```bash
docker compose down -v
```

## Notas Adicionais

- Certifique-se de que as portas mapeadas (3001, 5678, 8081, etc.) não estão em uso.
- Ajuste as variáveis de ambiente no arquivo `.env` para corresponder ao seu ambiente.
- Para ambientes de produção, configure HTTPS e ajuste as senhas para valores seguros.
- Consulte a documentação oficial de cada ferramenta para configurações avançadas.

## Licença

Este projeto é configurado para uso interno e não inclui uma licença específica. Consulte as licenças dos projetos individuais (Chatwoot, n8n, etc.) para detalhes.
