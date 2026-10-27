# Arquitetura

## Comunicações e protocolos
| Origem → Destino | Protocolo | Requisito que justifica |
|---|---|---|
| Veículo → Telemetria | MQTT QoS 0 (ou UDP), at-most-once | RNF01: tolera perda, prioriza baixo custo |
| Telemetria → Portal | WebSocket (ou SSE) | RF10: atualização ao vivo |
| Pedidos → Garantia (confirmação) | Fila durável (RabbitMQ), publisher confirms, ack manual, outbox | RNF01: confirmação não tolera perda |
| Garantia → Frete (prazo) | REST síncrono | Consulta simples, resposta imediata |
| Garantia → Telemetria (ausência) | REST síncrono (consulta de histórico) | RF05 |
| Garantia → Faturamento | Fila durável, mensagem com chave de idempotência | RF06, RNF02 (5 min) |
| Garantia → Notificações | Evento assíncrono na fila | RF07, RF08, desacopla falhas de envio |
| Notificações → parceiro WhatsApp | HTTPS com adaptador | RF07: formato de API diferente |

## Padrões de projeto aplicados
- **Strategy (RF02, RF03, RF04)**: `PoliticaDeCredito` com `PoliticaVarejo`, `PoliticaCorporativa`, `PoliticaCortesia`. Um registro (Factory/Registry) escolhe a política pelo perfil. Novo perfil = nova classe e novo registro, sem alterar as existentes (aberto/fechado).
- **Adapter (RF07)**: interface `CanalDeNotificacao` com `EmailAdapter` (serviço atual) e `WhatsAppAdapter` (API do parceiro). O domínio não conhece o formato do parceiro.
- **Retry com backoff + Dead Letter Queue (RF08)**: 3 tentativas; a falha final vai para a DLQ e gera log/alerta, nunca descartada em silêncio.

## Onde NÃO aplicar padrão (decisão explícita)
- **Cálculo do atraso (RF01)**: subtração de datas; uma função pura basta.
- **Ausência comprovada (RF05)**: um predicado (distância ≤ 200 m e permanência ≥ 10 min); sem hierarquia.
- **Numeração (RF06, RNF03)**: sem Singleton em memória (quebra com várias instâncias); usa `SEQUENCE` do banco de dados.

## RF09: confirmação duplicada
Tabela `confirmacoes_processadas` com **chave única** no id da confirmação. O consumidor tenta inserir antes de processar: se já existir, ignora a mensagem (idempotent consumer). A mesma chave vai ao Faturamento, que também rejeita duplicata. Isso cobre reentrega do broker e reenvio pelo Pedidos.

## RNF02 e RNF03
- Crédito em até 5 min: consumo assíncrono contínuo com alerta se a fila atrasar.
- Sem repetição de número: `SEQUENCE` do banco com transação; teste de 1.000 emissões concorrentes.

## RNF04
Um `docker-compose.yml` sobe broker, bancos e os serviços (`docker compose up`).
