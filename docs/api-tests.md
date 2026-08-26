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

## Validação dos cenários climáticos reais

Para validar a lógica de classificação do ClimaSeguro sem manipular artificialmente os dados do Make, foram utilizados cenários reais de previsão do tempo. A estratégia foi selecionar cidades cujas condições meteorológicas apresentassem características compatíveis com os resultados esperados pelo sistema.

Essa abordagem tornou o teste mais próximo de um cenário real de uso, pois o status climático foi determinado pelos dados retornados pela previsão meteorológica e não por valores inseridos manualmente.

### Cenário Crítico

Para o teste de cenário crítico foi utilizada a cidade de Fukuoka, no Japão.

A escolha foi baseada na previsão meteorológica disponível para o horário do evento, que indicava condições compatíveis com um nível elevado de risco climático.

Evento utilizado no teste:

- Nome: Teste crítico final validado
- Tipo: Externo
- Cidade: Fukuoka
- Data: 26/08/2026
- Horário: 14:00

Durante o processamento, o sistema recebeu aproximadamente os seguintes valores:

- Temperatura prevista: 31 °C
- Sensação térmica: 37,3 °C
- Probabilidade de chuva: 79%
- Velocidade do vento: 4 km/h

A regra de classificação para eventos externos considera o cenário Crítico quando pelo menos uma das condições críticas é atingida, incluindo probabilidade de chuva igual ou superior a 70%.

Como a previsão apresentou 79% de probabilidade de chuva, o resultado esperado era:

`Crítico`

Resultado obtido:

`event_risk = Crítico`

A recomendação gerada também foi compatível com o risco identificado:

`Recomenda-se reavaliar a realização do evento, considerar mudança de horário, local coberto ou adiamento.`

Após o processamento:

- o registro foi persistido no Airtable;
- o evento foi atualizado no Supabase;
- o frontend passou a exibir o status Crítico;
- o filtro de alerta permitiu a execução do Gmail;
- o responsável recebeu o alerta climático por e-mail.

Esse teste validou o fluxo completo de classificação crítica, recomendação, persistência, atualização da interface e envio de alerta.

### Cenário Normal

Para o teste do cenário Normal foi utilizada a cidade de Kushiro, em Hokkaido, Japão.

A cidade foi escolhida após consulta a condições meteorológicas favoráveis, buscando um local com temperatura, sensação térmica, probabilidade de chuva e vento abaixo dos limites de Atenção definidos pelo backend.

Evento utilizado no teste:

- Nome: Teste cenário normal Kushiro
- Tipo: Externo
- Cidade: Kushiro
- Data: 26/08/2026
- Horário: 16:00

Para eventos externos, o sistema considera condições de Atenção a partir de valores como:

- sensação térmica igual ou superior a 30 °C;
- temperatura elevada conforme as regras configuradas no Make;
- probabilidade de chuva igual ou superior a 40%;
- velocidade do vento igual ou superior a 24 km/h.

Como a previsão recebida para Kushiro permaneceu abaixo dos limites de Atenção, o resultado esperado era:

`Normal`

Resultado obtido:

`event_risk = Normal`

A recomendação gerada foi:

`Condições favoráveis no momento. Recomenda-se manter o acompanhamento até a realização do evento.`

Após o processamento:

- o evento foi persistido normalmente;
- o Supabase foi atualizado;
- o frontend passou a exibir o status Normal em verde;
- o filtro de alertas avaliou as condições Atenção e Crítico como falsas;
- o módulo Gmail não foi executado.

Esse comportamento é importante porque comprova que o sistema não gera alertas desnecessários para eventos classificados como Normal.

### Evidências do cenário Crítico

As seguintes capturas registram a validação do fluxo crítico:

- `76-critical-risk-output-final-success.png`
- `77-critical-recommendation-output-final-success.png`
- `78-critical-test-supabase-patch-success.png`
- `79-critical-alert-filter-inspector.png`
- `80-critical-alert-filter-configuration.png`
- `81-critical-alert-gmail-module-success.png`
- `82-critical-alert-email-success.png`
- `83-critical-test-frontend-success.png`

### Evidências do cenário Normal

As seguintes capturas registram a validação do fluxo normal:

- `84-normal-risk-output-success.png`
- `85-normal-recommendation-output-success.png`
- `86-normal-test-frontend-success.png`
- `87-normal-alert-filter-blocked-success.png`

### Conclusão dos testes

Os testes demonstraram que o ClimaSeguro consegue produzir comportamentos diferentes a partir de dados meteorológicos reais.

No cenário crítico, o sistema:

`Previsão de risco → Crítico → recomendação preventiva → persistência → atualização do frontend → envio de alerta`

No cenário normal, o sistema:

`Previsão favorável → Normal → recomendação de acompanhamento → persistência → atualização do frontend → alerta bloqueado`

A validação com cidades e previsões reais reforça a proposta do projeto, pois demonstra que o resultado climático não é escolhido pelo usuário e nem definido manualmente durante o teste. A classificação é consequência direta dos dados meteorológicos recebidos e das regras de negócio implementadas no backend.

## Validação final do cenário Atenção

Após as validações dos cenários Crítico e Normal, foi realizado um teste final do estado Atenção utilizando uma cidade com previsão meteorológica intermediária.

A estratégia foi selecionar uma cidade cujas condições reais de previsão ultrapassassem os limites de Normal, mas permanecessem abaixo dos gatilhos críticos.

### Cidade selecionada

Foi utilizada a cidade de Kobe, no Japão.

Evento cadastrado:

- Nome: Teste cenário atenção Kobe
- Tipo: Externo
- Cidade: Kobe
- Data: 26/08/2026
- Horário: 16:00

Os dados meteorológicos processados pelo ClimaSeguro foram aproximadamente:

- Temperatura prevista: 29,5 °C
- Sensação térmica: 35,6 °C
- Probabilidade de chuva: 48%
- Velocidade do vento: 10,6 km/h

Esses valores ultrapassaram os limites de Atenção definidos para eventos externos, principalmente pela sensação térmica e pela probabilidade de chuva, sem atingir os limites críticos.

Resultado esperado:

`Atenção`

Resultado obtido:

`event_risk = Atenção`

A recomendação gerada pelo sistema foi:

`Recomenda-se reforçar medidas preventivas, acompanhar a previsão e preparar alternativas para a atividade.`

Após o processamento:

- o registro foi persistido no Airtable;
- o evento foi atualizado no Supabase;
- o frontend passou a exibir o status Atenção;
- o filtro de alertas permitiu a execução do Gmail;
- o responsável recebeu o alerta climático por e-mail.

### Evidências do cenário Atenção

- `88-attention-risk-output-success.png`
- `89-attention-recommendation-output-success.png`
- `90-attention-airtable-record-success.png`
- `91-attention-supabase-patch-success.png`
- `92-attention-gmail-module-success.png`
- `93-attention-alert-email-success.png`
- `94-attention-frontend-success.png`

### Conclusão

Com esse teste, os três estados principais do ClimaSeguro foram validados utilizando previsões reais:

- Normal
- Atenção
- Crítico

O resultado climático não é escolhido manualmente pelo usuário. Ele é consequência dos dados meteorológicos recebidos e das regras de negócio aplicadas pelo backend.

Assim, o ClimaSeguro demonstrou comportamento coerente nos três níveis de risco, incluindo recomendação, persistência, atualização da interface e controle de alertas.