# Testes de APIs

## API 1 — Nominatim / OpenStreetMap

### Objetivo
Converter o nome de uma cidade em coordenadas geográficas.

### Método
GET

### Dados utilizados
- Nome da cidade
- Latitude
- Longitude
- País

### Autenticação
Não utilizada para esta consulta pública.

### Resultado
Integração realizada com sucesso por meio do módulo HTTP do Make.

---

## API 2 — Open-Meteo

### Objetivo
Consultar dados climáticos atuais a partir das coordenadas retornadas pela primeira API.

### Método
GET

### Dados utilizados
- Temperatura
- Sensação térmica
- Umidade relativa
- Velocidade do vento
- Fuso horário automático

### Autenticação
Não utilizada para esta consulta pública.

### Resultado
Integração realizada com sucesso por meio do módulo HTTP do Make.

---

## Tratamento dos Dados

O Make aplica uma regra de classificação sobre a temperatura recebida:

- abaixo de 30°C → Normal
- de 30°C a 35°C → Atenção
- acima de 35°C → Crítico

## Persistência

Após o tratamento, os registros são armazenados no Airtable.

A conexão entre Make e Airtable utiliza autenticação OAuth com acesso limitado à base necessária para o projeto.