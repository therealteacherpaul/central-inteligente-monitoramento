# Arquitetura da Solução

## Visão Geral

A Central Inteligente de Monitoramento recebe o nome de uma cidade por meio de um webhook e utiliza duas APIs externas para enriquecer os dados antes de armazená-los no Airtable.

## Fluxo de Integração

1. O webhook do Make recebe o nome da cidade.
2. A API Nominatim / OpenStreetMap converte o nome da cidade em latitude e longitude.
3. As coordenadas são enviadas para a API Open-Meteo.
4. A Open-Meteo retorna dados climáticos atuais.
5. O Make aplica uma regra de classificação operacional.
6. Os dados tratados são persistidos no Airtable.

## Componentes

### Webhook
Responsável pela entrada de dados da aplicação.

### Nominatim / OpenStreetMap
Responsável pela geocodificação da cidade.

### Open-Meteo
Responsável pela consulta dos dados climáticos.

### Make
Responsável pela orquestração, transformação e integração dos dados.

### Airtable
Responsável pela persistência das informações.

## Regra de Classificação

- Temperatura abaixo de 30°C: Normal
- Temperatura entre 30°C e 35°C: Atenção
- Temperatura acima de 35°C: Crítico

Os limites são regras operacionais definidas para fins de demonstração do protótipo e não representam uma classificação meteorológica oficial.