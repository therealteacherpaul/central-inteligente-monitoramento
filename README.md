# ClimaSeguro

Sistema inteligente de monitoramento de risco climático para eventos educacionais.

Projeto acadêmico desenvolvido para a UniFECAF, com foco em integração de APIs, automação digital, aplicações no-code/low-code e construção de uma solução funcional baseada em dados climáticos reais.

---

## 1. Visão geral

O **ClimaSeguro** é uma aplicação web criada para auxiliar instituições educacionais na tomada de decisão sobre eventos internos e externos sujeitos a condições climáticas adversas.

A solução permite cadastrar eventos contendo informações como:

- nome do evento;
- tipo de evento;
- cidade;
- data;
- horário;
- e-mail do responsável.

A partir dessas informações, o sistema consulta serviços externos de geolocalização e previsão meteorológica, processa os dados automaticamente e classifica o evento em três níveis de risco:

- **Normal**
- **Atenção**
- **Crítico**

Além da classificação, o sistema gera uma recomendação e pode enviar um alerta por e-mail ao responsável quando o evento apresenta risco climático relevante.

---

## 2. Problema identificado

Instituições educacionais realizam frequentemente atividades como:

- festivais;
- feiras;
- excursões;
- atividades esportivas;
- eventos culturais;
- reuniões externas;
- cerimônias;
- apresentações;
- eventos comunitários.

Essas atividades podem ser impactadas por fatores como:

- calor excessivo;
- sensação térmica elevada;
- chuva;
- vento;
- precipitação.

Em muitos casos, a análise dessas informações é feita manualmente, consultando diferentes fontes e dependendo da interpretação do responsável.

O ClimaSeguro foi desenvolvido para centralizar e automatizar esse processo.

---

## 3. Objetivo

O objetivo do projeto é criar uma solução capaz de:

1. receber o cadastro de um evento;
2. localizar geograficamente a cidade informada;
3. consultar a previsão meteorológica correspondente;
4. identificar as condições previstas especificamente para a data e horário do evento;
5. classificar automaticamente o risco climático;
6. gerar uma recomendação;
7. armazenar os resultados;
8. apresentar essas informações em uma interface web;
9. enviar alertas quando necessário.

---

## 4. Evolução do projeto

A proposta inicial consistia em um monitor genérico de condições climáticas.

Durante o desenvolvimento, o projeto foi reformulado para resolver um problema mais específico e melhor contextualizado academicamente.

A solução passou a ser direcionada para:

> Monitoramento de risco climático aplicado a eventos educacionais.

Essa mudança resultou no conceito atual do **ClimaSeguro**.

A evolução permitiu transformar uma simples consulta meteorológica em um fluxo completo de:

**cadastro → geolocalização → previsão → processamento → classificação → recomendação → persistência → alerta → visualização**

---

## 5. Arquitetura da solução

A arquitetura atual pode ser representada da seguinte forma:

    Usuário
       ↓
    Lovable
       ↓
    Supabase Auth
       ↓
    Supabase Database
       ↓
    Make.com Webhook
       ↓
    OpenStreetMap / Nominatim
       ↓
    Open-Meteo
       ↓
    Processamento das condições climáticas
       ↓
    Classificação do risco
       ↓
    Geração de recomendação
       ↓
    Airtable
       ↓
    Supabase
       ↓
    Gmail
       ↓
    Dashboard ClimaSeguro

---

## 6. Tecnologias utilizadas

### Lovable

Utilizado para o desenvolvimento da interface web da aplicação.

Responsável pelas telas de:

- login;
- cadastro;
- dashboard;
- cadastro de evento;
- listagem de eventos;
- detalhes;
- configurações;
- arquivamento de eventos.

### Supabase

Utilizado como backend principal da aplicação.

Responsável por:

- autenticação;
- gerenciamento de usuários;
- banco de dados;
- políticas RLS;
- perfis de instituições;
- eventos;
- armazenamento dos resultados processados;
- Edge Functions.

### Make.com

Utilizado como ferramenta central de automação e orquestração das integrações.

O cenário automatizado executa:

1. recebimento do evento via webhook;
2. geocodificação da cidade;
3. consulta meteorológica;
4. extração da previsão no horário do evento;
5. classificação do risco;
6. geração da recomendação;
7. registro no Airtable;
8. atualização do Supabase;
9. envio condicional de alerta por e-mail.

### OpenStreetMap / Nominatim

API utilizada para geocodificação.

A cidade informada pelo usuário é convertida em:

- latitude;
- longitude;
- país.

Essas coordenadas são utilizadas posteriormente pela API meteorológica.

### Open-Meteo

API utilizada para obtenção da previsão meteorológica.

São utilizados dados horários como:

- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento.

### Airtable

Utilizado como camada adicional de persistência e auditoria.

O Airtable mantém registros dos eventos processados com os dados retornados pelas APIs e pela automação.

### Gmail

Utilizado para envio automatizado de alertas climáticos.

Foi criada uma conta específica para o projeto, identificada como **Clima Seguro**.

Os e-mails são enviados automaticamente quando um evento é classificado como:

- Atenção;
- Crítico.

Eventos classificados como Normal não geram alerta.

---

## 7. Autenticação

A aplicação utiliza autenticação por e-mail e senha através do Supabase Auth.

O fluxo implementado é:

1. usuário cria uma conta;
2. informa o nome da instituição;
3. recebe um e-mail de confirmação;
4. confirma o endereço;
5. acessa a aplicação;
6. visualiza apenas seus próprios dados.

Também foram implementados:

- logout;
- login persistente;
- proteção de rotas;
- identificação da instituição conectada.

---

## 8. Estrutura de usuários

Foi criada a tabela:

    profiles

Principais campos:

    id
    institution_name
    created_at

O perfil é criado automaticamente após o cadastro do usuário.

Foram configuradas políticas de segurança RLS para impedir que usuários tenham acesso aos dados de outros usuários.

---

## 9. Estrutura de eventos

A tabela principal do Supabase é:

    events

Entre os principais campos estão:

    id
    user_id
    name
    type
    city
    event_date
    event_time
    responsible_email
    country
    latitude
    longitude
    temperature
    apparent_temperature
    humidity
    rain_probability
    precipitation
    wind_speed
    weather_status
    recommendation
    forecast_status
    processing_status
    weather_updated_at
    created_at
    updated_at
    archived
    archived_at

---

## 10. Cadastro de eventos

O usuário cadastra um evento informando:

    Nome do evento
    Tipo
    Cidade
    Data
    Horário
    E-mail do responsável

Os tipos disponíveis são:

    Interno
    Externo

Após o cadastro, o evento entra no fluxo de processamento automático.

---

## 11. Processamento climático

Após o recebimento do evento, o Make consulta o Nominatim para localizar a cidade.

Em seguida, as coordenadas são utilizadas para consultar o Open-Meteo.

A previsão é solicitada em resolução horária.

O sistema identifica especificamente o índice correspondente ao horário cadastrado para o evento.

Exemplo:

    Evento: 14:00
    ↓
    Busca da posição correspondente no array horário
    ↓
    Extração dos valores climáticos daquela hora

Isso evita utilizar apenas dados médios do dia.

---

## 12. Variáveis climáticas analisadas

O sistema considera:

    Temperatura
    Sensação térmica
    Umidade
    Probabilidade de chuva
    Precipitação
    Velocidade do vento

Esses valores são armazenados em variáveis intermediárias dentro do Make.

---

## 13. Classificação de risco

O ClimaSeguro utiliza regras condicionais para determinar o nível de risco.

### Eventos externos

Um evento externo pode ser classificado como **Crítico** quando condições extremas são identificadas, como:

    Sensação térmica >= 40°C
    Temperatura >= 35°C
    Probabilidade de chuva >= 70%
    Velocidade do vento >= 40 km/h

A classificação **Atenção** utiliza limites intermediários, como:

    Sensação térmica >= 30°C
    Probabilidade de chuva >= 40%
    Velocidade do vento >= 24 km/h

Quando nenhuma condição relevante é atingida:

    Normal

### Eventos internos

Eventos internos possuem menor exposição direta a determinadas condições meteorológicas.

A classificação considera principalmente:

    temperatura
    sensação térmica

---

## 14. Recomendações automáticas

Após a classificação do evento, o sistema gera uma recomendação.

### Normal

    Condições favoráveis no momento.
    Recomenda-se manter o acompanhamento até a realização do evento.

### Atenção

    Recomenda-se reforçar medidas preventivas,
    acompanhar a previsão e preparar alternativas para a atividade.

### Crítico

    Recomenda-se reavaliar a realização do evento,
    considerar mudança de horário, local coberto ou adiamento.

---

## 15. Alertas por e-mail

O envio de alertas é controlado por um filtro no Make.

O Gmail somente é acionado quando:

    event_risk = Atenção
    OU
    event_risk = Crítico

O e-mail contém:

- nome do evento;
- tipo;
- cidade;
- data;
- horário;
- status climático;
- temperatura;
- sensação térmica;
- probabilidade de chuva;
- velocidade do vento;
- recomendação.

---

## 16. Integração com Airtable

Cada evento processado também é registrado no Airtable.

Entre os campos armazenados estão:

    Nome do Evento
    Tipo
    Cidade
    País
    Latitude
    Longitude
    Data do Evento
    Horário do Evento
    E-mail do Responsável
    Temperatura Prevista
    Sensação Térmica
    Umidade
    Probabilidade de chuva
    Precipitação
    Velocidade do vento
    Status Climático
    Recomendação
    Status da Previsão
    Última Atualização

O Airtable funciona como evidência adicional da integração e como registro independente da automação.

---

## 17. Atualização do Supabase pelo Make

Após processar o evento, o Make envia os resultados novamente para o Supabase.

A atualização é realizada por requisição HTTP.

O Supabase passa então a armazenar:

- coordenadas;
- dados meteorológicos;
- status climático;
- recomendação;
- status do processamento.

Com isso, o frontend consegue apresentar os resultados calculados pela automação.

---

## 18. Dashboard

O Dashboard apresenta um resumo dos eventos do usuário.

Indicadores exibidos:

    Total de eventos
    Eventos futuros
    Normais
    Atenção
    Críticos
    Pendentes

Também são apresentados:

- eventos recentes;
- próximo evento.

---

## 19. Edge Function `events-summary`

Foi criada uma Edge Function no Supabase para gerar o resumo utilizado pelo Dashboard.

A função calcula:

    total_events
    future_events
    risk_summary.normal
    risk_summary.attention
    risk_summary.critical
    risk_summary.pending
    next_event

Também considera o timezone informado pela aplicação.

Durante os testes foi identificado um problema no frontend: alguns cards procuravam os valores diretamente na raiz da resposta JSON.

Exemplo incorreto:

    summary.normal
    summary.critical

Entretanto, a API retornava:

    summary.risk_summary.normal
    summary.risk_summary.critical

A correção foi realizada no frontend.

Após o ajuste, os contadores passaram a refletir corretamente os dados retornados pela Edge Function.

---

## 20. Timezone dinâmico

Durante os testes foi identificado que o cálculo de eventos futuros não poderia depender de um fuso horário fixo.

A Edge Function `events-summary` foi ajustada para receber o timezone do cliente através da query string.

Exemplos:

    /functions/v1/events-summary?timezone=Asia/Tokyo

    /functions/v1/events-summary?timezone=America/Sao_Paulo

O frontend detecta automaticamente o timezone do navegador através de:

    Intl.DateTimeFormat().resolvedOptions().timeZone

Caso um timezone válido não esteja disponível, o sistema utiliza UTC como fallback.

Foram realizados testes simulando:

- Japão;
- Brasil.

O Dashboard apresentou resultados diferentes e corretos conforme o contexto temporal do usuário.

---

## 21. Arquivamento lógico de eventos

Durante a evolução da interface foi identificada a necessidade de evitar que eventos antigos permanecessem indefinidamente na lista principal.

Em vez de excluir registros históricos, foi adotado um modelo de **arquivamento lógico**.

Foram adicionados à tabela `events`:

    archived boolean default false
    archived_at timestamptz

Essa decisão permite manter os registros no banco de dados e, ao mesmo tempo, removê-los da área principal da aplicação.

---

## 22. Decisão de arquivamento

A adoção de arquivamento lógico foi uma decisão intencional de arquitetura e experiência do usuário.

A exclusão definitiva passou a ser utilizada apenas para registros redundantes gerados durante desenvolvimento e testes.

Eventos operacionais podem ser arquivados sem perda histórica.

Essa abordagem permite:

- preservar histórico;
- evitar poluição visual da lista principal;
- manter rastreabilidade;
- restaurar registros quando necessário;
- evitar exclusão acidental de dados relevantes.

---

## 23. Interface de arquivamento

A tela **Meus Eventos** passou a possuir duas visualizações:

    Ativos
    Arquivados

Na aba Ativos, eventos passados podem utilizar a ação:

    Arquivar

Na aba Arquivados, cada evento possui:

    Restaurar

Eventos futuros não devem oferecer a ação de arquivamento enquanto ainda não tiverem ocorrido.

---

## 24. Comportamento do arquivamento

Ao arquivar um evento:

    archived = true
    archived_at = timestamp atual

O evento:

- deixa de aparecer na lista de ativos;
- passa para a aba Arquivados;
- deixa de ser considerado nos indicadores do Dashboard.

---

## 25. Restauração de eventos

Ao restaurar:

    archived = false
    archived_at = null

O evento retorna à lista de ativos.

O Dashboard também volta a considerar o registro nos seus cálculos.

---

## 26. Ajuste da Edge Function para arquivamento

A Edge Function `events-summary` foi atualizada para trabalhar apenas com eventos ativos.

Foi aplicado o filtro:

    .eq("archived", false)

Dessa forma, os registros arquivados não afetam:

    total_events
    future_events
    risk_summary.normal
    risk_summary.attention
    risk_summary.critical
    risk_summary.pending
    next_event

A lógica existente de timezone foi preservada.

---

## 27. Validação do arquivamento

O fluxo foi testado em diferentes etapas.

### Teste de arquivamento

Um evento foi arquivado pela interface.

Foi confirmado que:

- o registro desapareceu de Ativos;
- apareceu em Arquivados;
- o Dashboard deixou de contabilizá-lo;
- o evento também deixou de aparecer na lista de eventos recentes.

### Teste de restauração

Um evento previamente arquivado foi restaurado.

Após a restauração:

- o registro voltou à lista de ativos;
- voltou a aparecer no Dashboard;
- os indicadores foram recalculados corretamente.

---

## 28. Validação do Dashboard após arquivamento

Durante os testes finais de arquivamento, havia três eventos ativos:

- 1 Normal;
- 1 Atenção;
- 1 Crítico.

O Dashboard apresentava:

    Total de eventos: 03
    Normais: 01
    Atenção: 01
    Críticos: 01
    Pendentes: 00

Após arquivar o evento Normal:

    Total de eventos: 02
    Normais: 00
    Atenção: 01
    Críticos: 01
    Pendentes: 00

O evento arquivado também deixou de aparecer na lista de eventos recentes.

Esse teste confirmou que os cálculos da Edge Function respeitam corretamente o estado `archived`.

---

## 29. Validação da restauração no Dashboard

Após restaurar um evento previamente arquivado, o Dashboard voltou a considerar o registro.

O total voltou a aumentar e o evento restaurado reapareceu na lista de eventos recentes.

Isso confirmou o ciclo completo:

    Ativo
    ↓
    Arquivado
    ↓
    Removido dos indicadores
    ↓
    Restaurado
    ↓
    Reintegrado aos indicadores

---

## 30. Estratégia de testes climáticos

Para validar o sistema de classificação, não era suficiente testar apenas uma cidade.

Foi necessário encontrar localidades cujas condições meteorológicas permitissem reproduzir diferentes cenários.

Foram analisadas previsões meteorológicas para identificar cidades que apresentassem condições adequadas aos testes.

Essa etapa foi importante porque os testes utilizam dados climáticos reais e variáveis.

A pesquisa externa foi utilizada apenas para selecionar candidatos adequados aos casos de teste.

A classificação final continuou sendo produzida exclusivamente pelo fluxo real do ClimaSeguro e pelos dados recebidos da Open-Meteo.

---

## 31. Cenário Crítico

Foi utilizada:

    Fukuoka, Japão

O cenário apresentou condições suficientes para atingir os critérios de risco crítico.

Entre os dados observados durante o teste:

    Temperatura: aproximadamente 31°C
    Sensação térmica: aproximadamente 37°C
    Probabilidade de chuva: aproximadamente 79%

A probabilidade de chuva ultrapassou o limite definido para risco crítico.

Resultado:

    Crítico

Também foram validados:

- recomendação crítica;
- registro no Airtable;
- atualização do Supabase;
- atualização do frontend;
- envio do alerta por Gmail.

---

## 32. Cenário Normal

Para validar o cenário sem risco relevante foi utilizada:

    Kushiro, Japão

As condições retornadas permaneceram abaixo dos limites definidos para Atenção.

Resultado:

    Normal

Também foi validado que eventos normais:

- são persistidos;
- aparecem corretamente no frontend;
- não acionam alerta climático por e-mail.

O Filter Inspector do Make confirmou que as condições Atenção e Crítico foram avaliadas como falsas.

---

## 33. Cenário Atenção

Para reproduzir o cenário intermediário foi utilizada:

    Kobe, Japão

Durante o teste foram observados aproximadamente:

    Temperatura: 29,5°C
    Sensação térmica: 35,6°C
    Probabilidade de chuva: 48%
    Velocidade do vento: 10,6 km/h

Resultado:

    Atenção

Foi confirmado:

- status Atenção;
- recomendação preventiva;
- registro no Airtable;
- atualização no Supabase;
- exibição no frontend;
- envio de alerta por e-mail.

---

## 34. Matriz de testes finais

| Cenário | Cidade | Resultado esperado | Resultado obtido |
|---|---|---|---|
| Normal | Kushiro | Normal | Normal |
| Atenção | Kobe | Atenção | Atenção |
| Crítico | Fukuoka | Crítico | Crítico |

Os três níveis principais da lógica de classificação foram validados com sucesso utilizando previsões reais.

---

## 35. Correções realizadas durante o desenvolvimento

O projeto passou por diversas validações e correções.

### Extração do horário

Foi necessário adaptar o índice das previsões horárias para a estrutura de arrays utilizada pelo Make.

### Fórmula de risco

A lógica inicial apresentou dificuldades ao trabalhar diretamente com arrays da resposta HTTP.

A solução foi criar variáveis intermediárias para os valores específicos do evento.

Também foi necessário substituir uma lógica de condições mal agrupadas por `if()` aninhados, garantindo que qualquer condição crítica pudesse produzir corretamente o estado Crítico.

### Comparações textuais

Foram identificados problemas relacionados a valores textuais e espaços.

Foi utilizada normalização com:

    trim()

### Constraint do Supabase

A tabela `events` possuía uma restrição de valores permitidos para `weather_status`.

A constraint foi atualizada para aceitar:

    Normal
    Atenção
    Crítico

### Recomendações

A comparação utilizada na recomendação também foi ajustada para garantir correspondência consistente com o valor de `event_risk`.

### Filtro de e-mail

O filtro foi configurado para aceitar duas condições em lógica OR:

    Atenção
    OU
    Crítico

### Dashboard

Foi identificado que alguns cards utilizavam caminhos JSON incorretos.

Após a correção, os dados passaram a ser lidos de:

    risk_summary.normal
    risk_summary.attention
    risk_summary.critical
    risk_summary.pending

Também foi adicionada invalidação/refetch do resumo para reduzir inconsistências de cache.

### Próximo evento

Foi identificado que o frontend utilizava um evento da lista como fallback mesmo quando `next_event` era `null`.

Esse fallback foi removido.

Quando não há eventos futuros, o Dashboard passa a exibir:

    Nenhum evento futuro

### Timezone

A primeira versão utilizava um timezone fixo.

A solução foi permitir timezone dinâmico informado pelo navegador.

### Arquivamento

A Edge Function foi atualizada para ignorar eventos com:

    archived = true

### Revisão de código sugerido por IA

Durante a implementação do filtro de arquivamento, foi sugerida uma nova versão completa da Edge Function que alterava também a lógica temporal.

Antes do deploy, o código foi revisado e foi identificado risco de regressão na lógica de timezone.

A solução escolhida foi preservar a função já validada e adicionar somente o filtro:

    .eq("archived", false)

Essa decisão reduziu o risco de regressão e preservou o comportamento previamente testado no Japão e no Brasil.

---

## 36. APIs REST próprias

Além de consumir APIs externas, o ClimaSeguro possui duas APIs REST próprias implementadas através de Supabase Edge Functions.

### Validate Event

Endpoint:

    POST /functions/v1/validate-event

Responsável por centralizar a validação dos dados enviados pelo formulário de Novo Evento.

A API valida:

- nome;
- tipo;
- cidade;
- data;
- horário;
- e-mail do responsável.

Ela é utilizada diretamente pelo frontend antes da criação do evento.

### Events Summary

Endpoint:

    GET /functions/v1/events-summary?timezone=<IANA_TIMEZONE>

Responsável por consolidar os dados utilizados pelo Dashboard.

Retorna informações como:

    total_events
    future_events
    risk_summary
    next_event
    timezone
    current_date
    current_time

A função:

- utiliza autenticação do usuário;
- respeita RLS;
- considera timezone dinâmico;
- ignora eventos arquivados.

---

## 37. Uso real das APIs próprias

As APIs próprias não foram criadas apenas para demonstração isolada.

Elas fazem parte do fluxo real da aplicação.

A `validate-event` é utilizada no cadastro de novos eventos.

A `events-summary` alimenta o Dashboard.

Isso permite demonstrar no projeto:

    consumo de APIs externas
    +
    criação de APIs REST próprias
    +
    uso real dessas APIs pelo frontend

---

## 38. Papel do Supabase e do Airtable

O uso conjunto de Supabase e Airtable é intencional.

### Supabase

Atua como fonte transacional principal da aplicação.

Responsável por:

- usuários;
- autenticação;
- perfis;
- eventos;
- segurança;
- RLS;
- APIs próprias;
- dados utilizados pelo frontend;
- arquivamento.

### Airtable

Atua como camada complementar de integração e auditoria operacional.

Responsável por:

- espelhar resultados processados;
- facilitar inspeção visual;
- acompanhar automações;
- preservar uma camada no-code operacional.

Dessa forma, Supabase e Airtable não representam duplicação acidental.

Cada ferramenta possui responsabilidade diferente dentro da arquitetura.

---

## 39. Segurança

O sistema utiliza Row Level Security no Supabase.

As políticas garantem que cada usuário tenha acesso apenas aos próprios registros.

A autenticação é utilizada tanto no frontend quanto nas operações de consulta ao banco.

Chaves administrativas não são expostas diretamente no navegador.

Credenciais server-side utilizadas pelo Make são mantidas fora do frontend e do repositório público.

---

## 40. Decisões de projeto

O projeto priorizou:

- simplicidade;
- baixo custo;
- ferramentas gratuitas sempre que possível;
- rápida implementação;
- modularidade;
- integração entre serviços;
- facilidade de demonstração;
- clareza acadêmica;
- preservação de histórico;
- redução de complexidade desnecessária.

Foi adotada uma arquitetura low-code/no-code sempre que possível.

Código customizado foi utilizado apenas em pontos específicos, como:

- Edge Functions;
- lógica de interface;
- integração do frontend com o Supabase.

---

## 41. Evidências

Durante o desenvolvimento foram produzidos screenshots demonstrando:

- webhook;
- Nominatim;
- Open-Meteo;
- extração das variáveis climáticas;
- classificação de risco;
- recomendações;
- Airtable;
- Gmail;
- autenticação;
- Supabase;
- frontend;
- Dashboard;
- testes Normal, Atenção e Crítico;
- timezone dinâmico;
- arquivamento;
- restauração;
- Edge Functions;
- diagnóstico de valores;
- correções de binding;
- correções realizadas durante o desenvolvimento.

As imagens estão organizadas na pasta de evidências do projeto.

Entre as evidências mais recentes estão:

    95-supabase-events-archive-columns-created.png
    96-supabase-events-archive-columns-table-success.png
    97-supabase-redundant-test-records-selected.png
    98-supabase-redundant-test-records-deleted.png
    99-supabase-events-clean-state-success.png
    100-frontend-events-clean-state-success.png
    101-lovable-event-archive-implementation-summary.png
    102-lovable-event-archive-files-and-build-success.png
    103-frontend-active-events-before-archive.png
    104-frontend-active-events-after-archive.png
    105-frontend-archived-events-success.png
    106-frontend-event-restore-success.png
    107-supabase-event-restore-success.png
    108-lovable-events-summary-archive-filter-analysis.png
    109-lovable-events-summary-generated-code-review.png
    110-supabase-events-summary-archive-filter-ready.png
    111-supabase-events-summary-archive-filter-deployed.png
    112-dashboard-archive-filter-validation.png
    113-supabase-weather-status-raw-values-diagnostic.png
    114-events-summary-response-diagnostic.png
    115-lovable-dashboard-risk-summary-fix.png
    116-dashboard-risk-summary-correct.png
    117-dashboard-after-event-archive.png
    118-dashboard-after-event-restore.png

---

## 42. Fluxo funcional completo

    1. Usuário cria uma conta
    2. Confirma o e-mail
    3. Realiza login
    4. Cadastra evento
    5. validate-event verifica os dados
    6. Evento é salvo no Supabase
    7. Webhook inicia o processamento no Make
    8. Nominatim localiza a cidade
    9. Open-Meteo fornece a previsão
    10. Make identifica o horário do evento
    11. Variáveis meteorológicas são extraídas
    12. Sistema calcula o risco
    13. Recomendação é gerada
    14. Registro é criado no Airtable
    15. Supabase é atualizado
    16. Se necessário, Gmail envia alerta
    17. Frontend apresenta o resultado
    18. events-summary recalcula o Dashboard
    19. Eventos passados podem ser arquivados
    20. Eventos arquivados deixam de afetar o Dashboard
    21. Eventos arquivados podem ser restaurados
    22. O Dashboard passa a considerá-los novamente

---

## 43. Estado atual do projeto

Atualmente estão funcionais:

- autenticação;
- criação de conta;
- confirmação de e-mail;
- login;
- logout;
- proteção de rotas;
- cadastro de evento;
- validação através de API REST própria;
- persistência no Supabase;
- integração frontend → Make;
- geocodificação;
- consulta de previsão meteorológica;
- análise por data e horário;
- cálculo de risco;
- recomendação;
- integração com Airtable;
- atualização do Supabase;
- envio de alerta por Gmail;
- listagem de eventos;
- Dashboard;
- indicadores por nível de risco;
- Edge Function de resumo;
- timezone dinâmico;
- arquivamento de eventos;
- restauração de eventos;
- exclusão de eventos de teste redundantes;
- validação dos cenários Normal, Atenção e Crítico;
- duas APIs REST próprias integradas ao produto.

---

## 44. Próximas evoluções

Entre as evoluções possíveis estão:

- arquivamento automático após determinado período;
- filtros por período;
- busca por cidade;
- histórico climático;
- configurações personalizadas de limiares;
- notificações adicionais;
- relatórios;
- gráficos;
- reprocessamento de previsões;
- atualização automática da previsão próxima ao evento;
- painel administrativo;
- exportação de relatórios.

Esses itens são considerados evoluções futuras e não são necessários para o funcionamento principal da versão acadêmica atual.

---

## 45. Conclusão

O ClimaSeguro demonstra uma aplicação prática de integração de APIs, automação digital, no-code, low-code e desenvolvimento personalizado.

O projeto combina diferentes serviços para resolver um problema real: apoiar instituições educacionais na avaliação de riscos climáticos relacionados à realização de eventos.

A solução evoluiu de uma consulta meteorológica simples para um sistema integrado capaz de:

- autenticar usuários;
- registrar eventos;
- validar dados;
- localizar cidades;
- consultar APIs externas;
- interpretar dados horários;
- classificar riscos;
- gerar recomendações;
- persistir informações;
- enviar alertas;
- apresentar indicadores;
- disponibilizar APIs próprias;
- considerar timezone do usuário;
- preservar histórico através de arquivamento lógico.

Os testes realizados com diferentes cidades permitiram validar os três cenários principais da solução:

    Normal
    Atenção
    Crítico

O arquivamento lógico tornou a aplicação mais coerente com um sistema real, permitindo preservar dados históricos sem comprometer a visualização operacional.

O projeto também demonstrou um processo incremental de desenvolvimento, com testes, identificação de inconsistências, correções e validação técnica das soluções implementadas.

---

## Autor

Projeto desenvolvido para fins acadêmicos na **UniFECAF**.

Área:

**Inteligência Artificial e Automação Digital**

Projeto:

**ClimaSeguro — Monitoramento Inteligente de Risco Climático para Eventos Educacionais**