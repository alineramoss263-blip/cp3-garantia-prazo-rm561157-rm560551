# Plano de testes

## Pirâmide (proporção aproximada)
- **~70% unitários**: regras de crédito, atraso, ausência (rápidos, sem I/O).
- **~20% integração/contrato**: fila, banco (idempotência, sequência), adaptador WhatsApp.
- **~10% ponta a ponta**: fluxo confirmação → crédito → nota → notificação.
Justificativa: as regras de negócio concentram o risco e são baratas de testar; E2E são lentos e frágeis.

## Casos de teste
| ID | Requisito | Caso | Entrada | Resultado esperado | Dublê |
|---|---|---|---|---|---|
| T01 | RF02 | Atraso no limite | Varejo, 2 h 00 min | R$ 0 | nenhum |
| T02 | RF02 | Logo acima | Varejo, 2 h 01 min, frete 100 | R$ 10 | nenhum |
| T03 | RF02 | Limite 24 h | Varejo, 24 h 00 min | 10% | nenhum |
| T04 | RF02 | Acima de 24 h | Varejo, 24 h 01 min | 25% | nenhum |
| T05 | RF02 | Limite 72 h | Varejo, 72 h 00 min | 25% | nenhum |
| T06 | RF02 | Acima de 72 h | Varejo, 72 h 01 min | 50% | nenhum |
| T07 | RF02 | Teto exato | Varejo, 72 h 01 min, frete 1.000 | R$ 500,00 | nenhum |
| T08 | RF02 | Acima do teto | Varejo, frete 1.001 | R$ 500,00 | nenhum |
| T09 | RF03 | Percentuais dobrados | Corporativo, 2 h 01 min, frete 100 | R$ 20 (20%) | nenhum |
| T10 | RF03 | Teto corporativo | Corporativo, 72 h 01 min, frete 2.001 | R$ 2.000 | nenhum |
| T11 | RF03 | Cortesia | Frete 0, 80 h de atraso | R$ 0 e notificação enviada | Spy no notificador |
| T12 | RF04 | Novo perfil | Registrar perfil novo | Existentes inalterados | nenhum |
| T13 | RF05 | Distância no limite | 200 m, 10 min | Sem crédito | Stub de telemetria |
| T14 | RF05 | Acima da distância | 200,01 m, 10 min | Com crédito | Stub de telemetria |
| T15 | RF05 | Permanência insuficiente | 200 m, 9 min 59 s | Com crédito | Stub de telemetria |
| T16 | RF01 | Cálculo do atraso | Prazo 10:00, entrega 12:01 | 2 h 01 min | Relógio fixo |
| T17 | RF06/RNF03 | 1.000 emissões concorrentes | 1.000 threads | 1.000 números distintos | Banco real (contêiner) |
| T18 | RF07 | Canal preferido | Preferência WhatsApp | Adaptador WhatsApp chamado | Mock do parceiro |
| T19 | RF08 | Retentativas | API falha 3 vezes | 3 tentativas, mensagem na DLQ e log | Stub que sempre falha |
| T20 | RF09 | Confirmação duplicada | Mesma confirmação 2x | 1 crédito | Repositório fake |
| T21 | RF10 | Estados da tela | Sem telemetria > 90 s | Exibe "sem sinal" | Mock do WebSocket |

## Escolha dos dublês
- **Stub** quando só preciso de uma resposta fixa (prazo do Frete, telemetria).
- **Spy/Mock** quando o que verifico é a interação (notificação enviada, adaptador chamado).
- **Fake** (repositório em memória) para idempotência no nível unitário.
- **Nunca** dublê na regra de cálculo: ela é o objeto sob teste.
- **Banco real em contêiner** para T17: sequência concorrente não se prova com fake.
