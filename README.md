# Stack de Aplicação com Docker Compose

Este projeto define uma stack de serviços orquestrados com Docker Compose, projetada para suportar uma aplicação robusta com banco de dados, cache, automação, monitoramento, armazenamento de objetos e filas de mensagens. A stack é ideal para ambientes de desenvolvimento, testes ou produção, com foco em escalabilidade e facilidade de manutenção.

## Índice
- [Serviços Incluídos](#serviços-incluídos)
- [Pré-requisitos](#pré-requisitos)
- [Configuração](#configuração)
- [Como Executar](#como-executar)
- [Gerenciamento](#gerenciamento)
- [Observações](#observações)
- [Licenciamento](#licenciamento)

## Serviços Incluídos
- **PostgreSQL**: Banco de dados relacional com suporte a UTF-8 e localização pt-BR.
- **Adminer**: Interface web para gerenciamento do PostgreSQL.
- **Postgres Backup**: Backups automáticos diários do PostgreSQL.
- **Postgres Exporter**: Coleta métricas do PostgreSQL para monitoramento.
- **Redis**: Cache em memória para otimização de performance.
- **Redis Exporter**: Coleta métricas do Redis.
- **n8n**: Plataforma de automação de fluxos de trabalho.
- **Evolution API**: Integração com WhatsApp para mensagens.
- **Ollama**: Execução de modelos de linguagem locais.
- **Prometheus**: Sistema de monitoramento de métricas.
- **Node Exporter**: Métricas do sistema operacional.
- **Grafana**: Visualização de métricas e logs.
- **Loki**: Sistema de armazenamento de logs.
- **Promtail**: Agente de coleta de logs para o Loki.
- **MinIO**: Armazenamento de objetos compatível com S3.
- **RabbitMQ**: Sistema de filas para comunicação assíncrona.

## Pré-requisitos
- Docker e Docker Compose instalados (compatível com versão 3.8).
- Arquivo `.env` com variáveis de ambiente necessárias (veja `.env.example`).
- Conexão de rede para baixar imagens Docker.
- Portas configuradas no `docker-compose.yml` (ex.: 5432, 3000) livres no host.
- Arquivos de configuração (`prometheus.yml`, `loki-config.yml`, `promtail-config.yml`) criados no diretório do projeto.

## Configuração
1. Clone este repositório:
   ```bash
   git clone https://github.com/opastorello/n8n/
   cd n8n
   ```
2. Crie o arquivo `.env` com base no modelo `.env.example`:
   ```bash
   cp .env.example .env
   ```
   - Edite `.env` com valores apropriados (ex.: `POSTGRES_USER`, `REDIS_PASSWORD`).
3. Crie os arquivos de configuração:
   - `prometheus.yml`: Configuração do Prometheus.
   - `loki-config.yml`: Configuração do Loki.
   - `promtail-config.yml`: Configuração do Promtail.
   Exemplo para `promtail-config.yml`:
   ```yaml
   server:
     http_listen_port: 9080
     grpc_listen_port: 0
   positions:
     filename: /tmp/positions.yaml
   clients:
     - url: http://loki:3100/loki/api/v1/push
   scrape_configs:
     - job_name: system
       static_configs:
         - targets:
             - localhost
           labels:
             job: varlogs
             __path__: /var/log/*.log
   ```
4. Verifique as permissões dos arquivos:
   ```bash
   chmod 644 *.yml
   ```

## Como Executar
1. Inicie os serviços em segundo plano:
   ```bash
   docker-compose up -d
   ```
2. Monitore os logs, se necessário:
   ```bash
   docker-compose logs -f
   ```
3. Acesse os serviços pelas portas configuradas:
   - Grafana: `http://localhost:3000`
   - Adminer: `http://localhost:8081`
   - n8n: `http://localhost:5678`
   - MinIO: `http://localhost:9001`
   - RabbitMQ: `http://localhost:15672`

## Gerenciamento
- **Parar os serviços**:
  ```bash
  docker-compose down
  ```
- **Remover volumes** (limpa dados persistentes):
  ```bash
  docker-compose down --volumes
  ```
- **Verificar status**:
  ```bash
  docker-compose ps
  ```
- **Reiniciar um serviço específico**:
  ```bash
  docker-compose restart <serviço>
  ```

## Observações
- **Segurança**: Não versione o arquivo `.env`. Use ferramentas como Vault em produção.
- **Portas**: Resolva conflitos de portas antes de iniciar a stack.
- **Volumes**: Dados são persistidos em volumes nomeados (ex.: `postgres_data`).
- **Healthchecks**: Configurados para garantir disponibilidade dos serviços.
- **Produção**: Considere HTTPS, redes privadas e ajustes de segurança (ex.: restringir `OLLAMA_ORIGINS`).
- **Configurações**: Arquivos `*.yml` devem ser validados antes de subir a stack.

## Licenciamento
Este projeto está licenciado sob a [MIT License](LICENSE), salvo indicação em contrário para componentes de terceiros.
