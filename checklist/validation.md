# ✔ Observability Validation Checklist

## Infra
- [ ] Collector respondendo em :4317
- [ ] Grafana acessível em :3000
- [ ] Tempo conectado
- [ ] Loki conectado

## App
- [ ] App inicia com javaagent
- [ ] Endpoint responde
- [ ] Traces aparecem no Tempo
- [ ] Logs aparecem no Loki
- [ ] service.name correto
- [ ] env correto

## Correlação
- [ ] Trace → Logs funcionando
- [ ] Labels visíveis no Grafana
