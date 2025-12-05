# Ambiente de Monitoramento — Prometheus + Grafana + Node Exporter

Este repositório contém a estrutura completa para subir um ambiente de monitoramento utilizando Docker Compose, incluindo:

- Prometheus (coleta e armazenamento de métricas)
- Grafana (visualização)
- Node Exporter (métricas do host local)
- Descoberta automática de exportadores externos via arquivo
- Regras de alerta básicas

## Como subir o ambiente

1. Pré-requisitos

- Docker instalado
- Docker Compose instalado
- Permissão para usar volumes e montar o filesystem host

2. Subir os serviços

    ```docker
    docker-compose up -d
    ```

3. Acessos

    | Serviço       | URL                                                            | Porta |
    | ------------- | -------------------------------------------------------------- | ----- |
    | Prometheus    | [http://localhost:9090](http://localhost:9090)                 | 9090  |
    | Grafana       | [http://localhost:3000](http://localhost:3000)                 | 3000  |
    | Node Exporter | [http://localhost:9100/metrics](http://localhost:9100/metrics) | 9100  |

4. Login padrão do Grafana

- __Usuário__: admin
- __Senha__: admin

## Estrutura do Repositório

```text
.
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── alert.rules
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── datasource.yml
│   │   └── dashboards/
│   │       └── dashboard.yml
│   └── dashboards/
├── config/
│   └── node_exporter_targets.yml   # Editado com IPs reais antes do deploy
```

## Configurações Importantes

__Prometheus__ (```prometheus/prometheus.yml```)

- scrape_interval de 15s
- Coleta:
    - O próprio Prometheus
    - Node Exporter local (container)
    - Exportadores externos via ```file_sd_configs```

__Alertas__ (```prometheus/alert.rules```)

Regra incluída:

- NodeExporterDown: alerta se o Node Exporter ficar offline por mais de 1 minuto.

__Node Exporters externos__ (```config/node_exporter_targets.yml```)

Antes de usar em produção, preencher com os IPs reais:

```yaml
- targets:
    - '192.168.1.100:9100'
    - '192.168.1.101:9100'
  labels:
    group: 'production-servers'
```

__Grafana__ (```grafana/provisioning```)

- Datasource Prometheus provisionado automaticamente
- Dashboard folder via arquivo
- Suporte para dashboards customizados na pasta ```grafana/dashboards```

## Persistência de Dados

Docker volumes usados:

- ```prometheus_data```: mantém o histórico de métricas
- ```grafana_data```: mantém dashboards, usuários e configurações

## Observações para Produção

- Alterar usuário e senha padrão do Grafana
- Configurar autenticação/HTTPS se expor externamente
- Ajustar ```node_exporter_targets.yml``` com IPs reais
- Ajustar retenção do Prometheus conforme necessidade:

```ini
--storage.tsdb.retention.time=200h
```
