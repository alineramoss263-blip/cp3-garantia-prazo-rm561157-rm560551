# Tela do Portal do Cliente

## Tela como função do estado
Tela = f(estado). O estado vem de três fontes: posição (WebSocket), previsão de chegada e saldo de créditos (REST).

## Estados
| Estado | Condição | O que a tela mostra |
|---|---|---|
| Carregando | Primeira carga | Skeleton no mapa, no painel e no saldo |
| Ao vivo | Posição recebida há menos de 90 s | Mapa com veículo, previsão, saldo |
| Sem sinal | Sem posição há 90 s ou mais (telemetria tolera perda) | Última posição, aviso "sinal indisponível", horário da última atualização |
| Atrasado | Previsão acima do prazo prometido | Destaque de atraso e aviso de possível crédito |
| Entregue | Confirmação recebida | Entrega concluída, crédito (se houver) |
| Erro | Falha na API | Mensagem e botão "tentar novamente" |
| Sem créditos | Saldo zero | Saldo R$ 0,00 e texto explicativo |

## Componentes
| Componente | Estados que exibe |
|---|---|
| `MapaVeiculo` | Ao vivo, Sem sinal |
| `PrevisaoChegada` | Ao vivo, Atrasado, Entregue |
| `BannerDeStatus` | Sem sinal, Atrasado, Erro |
| `SaldoDeCreditos` | Todos, com variação de Sem créditos |
| `SkeletonPortal` | Carregando |
| `ErroComRetentativa` | Erro |

## Atualização ao vivo
WebSocket com reconexão automática; o estado "sem sinal" é calculado por temporizador sobre o horário da última posição.
