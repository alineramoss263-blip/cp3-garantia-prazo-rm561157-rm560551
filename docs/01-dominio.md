# Modelagem do domínio

## Linguagem do negócio
- **Prazo prometido**: data/hora limite de entrega definida pelo Frete.
- **Confirmação de entrega**: evento que registra a entrega concluída.
- **Atraso**: tempo entre o prazo prometido e a confirmação (RF01).
- **Política de crédito**: regra que transforma atraso e frete em valor de crédito, por perfil de cliente.
- **Crédito**: valor concedido ao cliente por atraso.
- **Nota de crédito**: documento fiscal/contábil do crédito, com número sequencial único.
- **Ausência comprovada**: destinatário ausente confirmado pela telemetria (RF05).

## Contextos delimitados
| Contexto | Responsabilidade | Tecnologia |
|---|---|---|
| Telemetria | Posição dos veículos a cada 30 s; fornece fatos de localização | Frota |
| Pedidos | Pedido e confirmação de entrega | Java |
| Frete | Prazo prometido e valor do frete | Python |
| **Garantia de Prazo (novo)** | Atraso, política de crédito, ausência comprovada, idempotência | Python |
| Faturamento | Nota de crédito e numeração sequencial | C# |
| Notificações | Canais, retentativas e falha final | Node/TS |
| Portal do Cliente | Visualização ao vivo e saldo | React |

## Responsável único por requisito
| Requisito | Contexto responsável | Observação |
|---|---|---|
| RF01 | Garantia de Prazo | Consome prazo (Frete) e confirmação (Pedidos) |
| RF02, RF03 | Garantia de Prazo | Uma política por perfil de cliente |
| RF04 | Garantia de Prazo | Novo perfil = nova política, sem tocar nas existentes |
| RF05 | Garantia de Prazo | Decide; a Telemetria só fornece posição e horário |
| RF06 | Faturamento | Numeração é regra do Faturamento |
| RF07 | Notificações | Adaptador isola a API do parceiro WhatsApp |
| RF08 | Notificações | Retentativas e registro da falha final |
| RF09 | Garantia de Prazo | Idempotência pela chave da confirmação |
| RF10 | Portal do Cliente | Lê telemetria, previsão e saldo |
| RNF01 | Telemetria / Pedidos | Perda tolerada vs. perda proibida |
| RNF02 | Garantia de Prazo | Orquestra o prazo de 5 min |
| RNF03 | Faturamento | Sequência no banco |
| RNF04 | Plataforma | Docker Compose |

## Modelo do contexto Garantia de Prazo
- **Agregado** `GarantiaDeEntrega` (identidade: id da confirmação de entrega). Estados: Pendente, Avaliada.
- **Objetos de valor**: `Atraso` (duração), `Dinheiro` (Decimal, 2 casas), `PerfilDeCliente`.
- **Serviço de domínio**: `PoliticaDeCredito` (uma implementação por perfil).
- **Eventos**: `EntregaConfirmada` (entrada), `CreditoConcedido` e `EntregaAtrasadaSemCredito` (saída).

## Fronteiras e relações
- Garantia → Frete: cliente/fornecedor, consulta do prazo.
- Garantia → Faturamento e Notificações: eventos publicados.
- Notificações ↔ parceiro WhatsApp: camada anticorrupção (adaptador).
