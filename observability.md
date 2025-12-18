code.quarkus.io
   ↓
gera projeto (pom.xml pronto)
   ↓
cria APIs / serviços
   ↓
./mvnw package
   ↓
Docker build
   ↓
docker run



[ App Quarkus ]
      |
      |  (javaagent + OTEL_* env vars)
      v
[ OpenTelemetry Collector ]
      |
      +--> Traces → Tempo / Jaeger
      +--> Logs   → Loki
      +--> Metrics→ Prometheus





┌──────────────────────────┐
│ Java Quarkus App         │
│ + OpenTelemetry Agent    │
└───────────┬──────────────┘
            │ OTLP
            ▼
┌──────────────────────────┐
│ OpenTelemetry Collector  │
│                          │
│ receivers: otlp          │
│ processors: batch        │
│ exporters:               │
│  - tempo                 │
│  - loki                  │
│  - mimir                 │
└───────┬─────────┬────────┘
        │         │
        ▼         ▼
┌────────────┐ ┌────────────┐
│ Tempo       │ │ Loki        │
└────────────┘ └────────────┘
        │
        ▼
┌──────────────────────────┐
│ Mimir / Prometheus       │
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│ Grafana                  │
└──────────────────────────┘



encoding/
├── java_quarkus/
│   └── quarkus-random-simulator/
│       ├── pom.xml
│       ├── src/
│       └── target/
│
├── observability/
│   ├── docker/
│   │   ├── docker-compose.yml
│   │   └── .env
│   │
│   ├── otel/
│   │   ├── otel-collector.yaml
│   │   └── README.md
│   │
│   ├── grafana/
│   │   ├── provisioning/
│   │   │   ├── datasources/
│   │   │   │   └── datasources.yaml
│   │   │   └── dashboards/
│   │   │       └── dashboards.yaml
│   │   └── dashboards/
│   │       └── quarkus-overview.json
│   │
│   └── checklist/
│       └── validation.md




service.name
service.version
env
team
application
runtime



Diferença importante: -DskipTests vs -Dmaven.test.skip=true
Embora ambos evitem a execução dos testes, há uma pequena diferença técnica:
-DskipTests: Compila os testes, mas não os executa. (Mais recomendado).
-Dmaven.test.skip=true: Não compila e nem executa os testes. É o modo mais rápido de todos, mas você não saberá se seus testes sequer compilam.


## EXECUÇÃO JAVA QUARKUS SEM MONITORAÇÃO

### JAVA QUARKUS MODO DESENVOLVEDOR (BASH)

```bash
./mvnw quarkus:dev
```

&nbsp;
---

### JAVA QUARKUS COMO PRODUÇAO - LOCAL (BASH)

```bash
./mvnw package -DskipTests 
java -jar target/quarkus-app/quarkus-run.jar
```

>O parâmetro `-DskipTests` instrui o ***Maven*** a não executar a fase de testes unitários e de integração durante o ciclo de construção.

&nbsp;
---

### JAVA QUARKUS COMO PRODUÇAO - DOCKER (BASH)

```bash
./mvnw package -DskipTests
docker build -t quarkus-sim .
docker run -p 8080:8080 quarkus-sim
```

&nbsp;
---

## EXECUÇÃO JAVA QUARKUS COM MONITORAÇÃO OPENTELEMETRY

### JAVA QUARKUS COM AGENT - BASH

```bash
java ^
-javaagent:C:\otel\opentelemetry-javaagent.jar ^
-jar target/quarkus-app/quarkus-run.jar
```

&nbsp;
---

### JAVA QUARKUS COM AGENT - POWERSHELL

```powershell
java -javaagent:C:\otel\opentelemetry-javaagent.jar -jar target\quarkus-app\quarkus-run.jar

# ou

java `
-javaagent:C:\otel\opentelemetry-javaagent.jar `
-jar target\quarkus-app\quarkus-run.jar
```

&nbsp;
---

### VARIAVEIS DE AMBIENTE TEMPORÁRIAS - POWERSHELL

```powershell
$env:OTEL_METRICS_EXPORTER="otlp"
$env:OTEL_TRACES_EXPORTER="otlp"
$env:OTEL_LOGS_EXPORTER="otlp"

$env:OTEL_SERVICE_NAME="simulation-api"
$env:QUARKUS_OTEL_SERVICE_NAME="simulation-api"

$env:OTEL_RESOURCE_ATTRIBUTES="service.version=1.0.0,env=local,team=platform,application=simulator"

$env:OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4317"
$env:OTEL_EXPORTER_OTLP_PROTOCOL="grpc"
```

&nbsp;
---

### DOCKER COMPOSE PLATAFORMA OBSERVABILIDADE OPENTELEMETRY - BASH

```bash
docker compose down -v
docker compose up -d

### ou

docker compose down
docker compose up -d

```

&nbsp;
---

### LOGS CONTAINERS

```bash
docker compose logs -f otel-collector
docker compose logs -f grafana
docker compose logs -f loki
docker compose logs -f mimir
docker compose logs -f tempo
```

&nbsp;
---

### RUN PARA SIMULAÇÃO DE EXECUÇÃO

```bash
curl http://localhost:8080/simulation/run
curl "http://localhost:8080/simulation/batch?qtd=10"
```

&nbsp;
---

### LINKS SWAGGER
http://localhost:8080/swagger/
http://localhost:8080/swagger/#/Simulation%20Resource/get_simulation_run

&nbsp;
---

### LINKS GRAFANA
http://localhost:3000/explore
http://localhost:3000

&nbsp;
---

### QUERIES NO GRAFANA

```metricql
{job="simulation-api"}
```

```logql
{service_name="simulation-api"}
{service_name="simulation-api", log_level="ERROR"}
```

```traceql
service.name="simulation-api"
```

&nbsp;
---

## CAMADA 1: CONFIGURAÇÃO `simulation-api`, FIXAR NO APP

### application.properties`

Colocar `simulation-api` como `service_name` pelo app:

```
quarkus.application.name=simulation-api

quarkus.otel.enabled=true
quarkus.otel.service.name=simulation-api
```


### ENV VARS

Colocar `simulation-api` via *env vars* (melhor para ***K8S***)

```bash
OTEL_SERVICE_NAME=simulation-api
```

ou

```bash
QUARKUS_OTEL_SERVICE_NAME=simulation-api
```

&nbsp;
---

## CAMADA 2: GARANTIR NO OTEL COLLECTOR (FALLBACK SEGURO)

### `observability/otel/otel-collector.yaml`

Adicionar `resource`processor:

```yaml
processors:
  resource:
    attributes:
      - key: service.name
        value: simulation-api
        action: upsert
```

E usar nos pipelines:

```yaml
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [resource, batch]
      exporters: [otlp/tempo]

    logs:
      receivers: [otlp]
      processors: [resource, attributes, batch]
      exporters: [loki]

    metrics:
      receivers: [otlp]
      processors: [resource, batch]
      exporters: [prometheusremotewrite]
```


&nbsp;
---

## Validar Grafana - Explore

### Metrics - Mimir

```promql
up
```

```ini
service_name="simulation-api"
```

### Logs - Loki

```logql
{service_name="simulation-api"}
```

### Traces - Tempo

```ini
service.name = simulation-api
```


&nbsp;
---

## Mapping direto Docker → Helm

### O que é

Pega cada serviço do docker-compose e traduz quase 1:1:

```yaml
docker-compose → Deployment
ports → Service
volumes → PVC
env → env
```

### Exemplo mental
```yaml
# docker-compose
tempo:
  image: grafana/tempo
  ports:
    - "3200:3200"
```

vira:

```yaml
apiVersion: apps/v1
kind: Deployment
---
apiVersion: v1
kind: Service
```

### Vantagens

✔ Mais rápido de entender
✔ Ótimo para aprender Kubernetes
✔ Ideal para teste local (Kind)

### Limitações (importantes no seu caso)

❌ Configurações ficam espalhadas

❌ Difícil versionar

❌ Sem reuso entre clientes

❌ Naming e labels viram bagunça

❌ Atualizar versão vira “copiar e colar YAML”

### Conclusão:
Bom como ponte, ruim como plataforma.


&nbsp;
---

## Esqueleto Helm (charts + values)

### O que é

Cria uma plataforma de observabilidade como produto, usando:

charts/
├── observability/
│   ├── templates/
│   ├── values.yaml
│   ├── Chart.yaml


E tudo muda via values, não via YAML duplicado.

Como fica a arquitetura
observability/
├── charts/
│   └── observability/
│       ├── templates/
│       │   ├── tempo.yaml
│       │   ├── loki.yaml
│       │   ├── mimir.yaml
│       │   ├── otel-collector.yaml
│       │   ├── grafana.yaml
│       │   └── _helpers.tpl
│       ├── values.yaml
│       └── Chart.yaml
├── environments/
│   ├── local.yaml
│   ├── kind.yaml
│   └── prod.yaml

&nbsp;
---

### Vantagens (aqui está o ouro 🪙)

✔ Kubernetes-first
✔ Um único chart → múltiplos ambientes
✔ Clientes não veem complexidade
✔ Atualização centralizada
✔ Naming, labels e padrões forçados
✔ Ideal para:

- Multi-tenant
- Multi-cluster
- Times diferentes

&nbsp;
---

### Conclusão:
É o caminho certo para plataforma corporativa.

&nbsp;
---

## Analogia simples

Imagine que você está criando um sistema operacional:

- Docker → Helm mapping =  
“Rodar apps manualmente”

- Helm esqueleto =  
“Distribuição Linux com padrões”


Simplificar o máximo possível para o cliente e tudo mais complexo fique responsável pela a equipe:

- Isso é exatamente Helm + values.

O cliente:

- Só define:
    - service.name
    - environment
    - endpoint OTLP
- Nunca toca em:
    - Tempo
    - Loki
    - Mimir
    - Grafana
    - Collector

### Recomendação

| Etapa                       | Abordagem                         |
| --------------------------- | --------------------------------- |
| Agora (local / aprendizado) | **Mapping mental Docker → Helm**  |
| Próximo passo               | **Esqueleto Helm desde o início** |
| Produção                    | Helm + GitOps                     |

> ⚠️ Não vale a pena escrever YAML “puro” agora, porque você já sabe que vai migrar.