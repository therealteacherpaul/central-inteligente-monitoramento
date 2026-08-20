# Central Inteligente de Monitoramento

Projeto desenvolvido para a disciplina de Integração de APIs da UniFECAF.

## Objetivo

Desenvolver uma central capaz de integrar serviços externos por meio de APIs, tratar informações, armazenar os dados em uma base no-code e apresentá-los posteriormente em uma interface centralizada.

## Arquitetura prevista

Usuário → Interface → Webhook → Make → API de Geolocalização → API Climática → Airtable

## Tecnologias

- Make
- Airtable
- OpenStreetMap Nominatim API
- Open-Meteo API
- Softr
- GitHub

## Fluxo Atual

Webhook → Nominatim → Open-Meteo → Tratamento → Airtable

## Funcionalidades Implementadas

- Recebimento de cidade por webhook
- Geocodificação por API
- Consulta de clima por API
- Tratamento e classificação dos dados
- Persistência no Airtable

## Evidências

Os screenshots da implementação estão disponíveis na pasta `screenshots/`.

### Evidências do Dia 1

1. `01-webhook-success.png` — recebimento da cidade pelo webhook
2. `02-nominatim-geocoding-success.png` — retorno da geocodificação
3. `03-open-meteo-weather-success.png` — retorno dos dados climáticos
4. `04-status-variable-success.png` — classificação dos dados
5. `05-make-airtable-module-success.png` — execução do módulo Airtable no Make
6. `06-airtable-table-record-success.png` — registro persistido no Airtable
7. `07-make-full-workflow.png` — visão completa do cenário no Make

## Próximos Passos

- Construir interface no Softr
- Criar dashboard
- Criar tela de nova consulta
- Criar tela de histórico
- Adicionar automação de alerta

## Status

🚧 Em desenvolvimento