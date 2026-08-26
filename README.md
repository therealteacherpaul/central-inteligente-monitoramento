# ClimaSeguro

Projeto acadêmico desenvolvido para a disciplina de Integração de APIs da UniFECAF, no contexto do curso de IA e Automação Digital.

## Objetivo

O ClimaSeguro é uma aplicação voltada ao apoio à gestão de eventos institucionais, com foco na análise de riscos climáticos.

A plataforma permite cadastrar eventos, consultar condições meteorológicas previstas para a data e horário informados, classificar o risco climático, gerar recomendações e emitir alertas aos responsáveis.

O projeto combina autenticação, banco de dados, automações, APIs externas, APIs REST próprias e uma interface web desenvolvida com ferramentas no-code e low-code.

---

## Problema

Instituições de ensino realizam eventos internos e externos que podem ser impactados por chuva, calor excessivo, vento forte e outras condições climáticas.

A consulta manual de diferentes fontes meteorológicas pode dificultar a tomada de decisão e aumentar o risco operacional.

O ClimaSeguro foi criado para concentrar esse processo em um único fluxo:

- cadastro do evento;
- validação dos dados;
- consulta da previsão;
- análise de risco;
- geração de recomendação;
- armazenamento dos resultados;
- atualização do banco principal;
- registro operacional complementar;
- alerta ao responsável;
- exibição dos resultados no frontend.

---

## Arquitetura Atual

Fluxo principal da aplicação:

```text
Usuário
   ↓
Lovable
   ↓
Supabase Auth
   ↓
validate-event
   ↓
Supabase Database
   ↓
event_id
   ↓
Make
   ↓
OpenStreetMap Nominatim
   ↓
Open-Meteo
   ↓
Tratamento dos dados
   ↓
Classificação de risco
   ↓
Recomendação
   ↓
Airtable
   ↓
Supabase REST API
   ↓
Atualização do evento
   ↓
Filtro de alerta
   ↓
Gmail
   ↓
Lovable exibe dados atualizados
```

Arquitetura consolidada:

```text
Lovable
   ↓
Supabase
   ├── Auth
   ├── profiles
   ├── events
   └── Edge Functions
          ↓
        Make
          ↓
   Nominatim + Open-Meteo
          ↓
   Risco + Recomendação
          ↓
   Airtable + Supabase
          ↓
        Gmail
```

---

## Tecnologias

- Lovable
- Supabase Database
- Supabase Auth
- Supabase Edge Functions
- Supabase REST API
- Make
- Airtable
- OpenStreetMap Nominatim API
- Open-Meteo API
- Gmail
- GitHub

---

## Funcionalidades Implementadas

- Autenticação por e-mail e senha com Supabase
- Confirmação de cadastro por e-mail
- Login e logout
- Rotas protegidas
- Cadastro automático de perfil institucional
- Row Level Security no Supabase
- Estrutura real da tabela de eventos
- API REST própria para validação de eventos
- API REST própria para resumo de eventos
- Integração das APIs próprias com o frontend
- Webhook para recebimento dos dados do evento
- Identificação do evento através de `event_id`
- Geocodificação de cidade
- Consulta de previsão climática por data e horário
- Tratamento dos dados meteorológicos
- Classificação de risco climático
- Geração de recomendação
- Persistência complementar no Airtable
- Atualização automática do evento no Supabase após o processamento climático
- Integração Make → Supabase via REST API
- Sincronização do status, recomendação e dados meteorológicos com a tabela `events`
- Controle de processamento através do campo `processing_status`
- Envio de alertas por e-mail
- Filtro para envio de alertas apenas quando necessário
- Dashboard com dados reais
- Tela Meus Eventos com dados reais
- Tela Detalhes do Evento com dados meteorológicos reais
- Tela Configurações simplificada e sem controles não funcionais
- Cálculo de eventos futuros considerando data e horário
- Suporte a timezone dinâmico na API `events-summary`
- Detecção automática do timezone do navegador no frontend
- Integração automática do timezone com a API
- Validação do comportamento temporal em Japão e Brasil
- Fluxo ponta a ponta validado pelo frontend

---

## Banco de Dados

### Tabela `profiles`

Armazena as informações básicas da instituição vinculada ao usuário autenticado.

Principais campos:

- `id`
- `institution_name`
- `created_at`

O registro de perfil é criado automaticamente após o cadastro do usuário.

### Tabela `events`

A tabela `events` é a fonte transacional principal dos eventos da aplicação.

Principais campos:

- `id`
- `user_id`
- `name`
- `type`
- `city`
- `event_date`
- `event_time`
- `responsible_email`
- `country`
- `latitude`
- `longitude`
- `temperature`
- `apparent_temperature`
- `humidity`
- `rain_probability`
- `precipitation`
- `wind_speed`
- `weather_status`
- `recommendation`
- `forecast_status`
- `processing_status`
- `weather_updated_at`
- `created_at`
- `updated_at`

O campo `id` identifica cada evento de forma única.

O campo `user_id` identifica o usuário proprietário do evento.

O campo `processing_status` permite acompanhar o estado do processamento climático.

Estados utilizados:

```text
pending
processed
error
```

Fluxo de atualização:

```text
Evento criado
↓
processing_status = pending
↓
Make processa o evento
↓
Supabase é atualizado
↓
processing_status = processed
```

---

## Segurança

O projeto utiliza Supabase Auth para autenticação dos usuários.

A tabela `events` possui Row Level Security.

As políticas implementadas garantem que cada usuário autenticado possa:

- visualizar somente os próprios eventos;
- cadastrar somente eventos vinculados à própria conta;
- atualizar somente os próprios eventos;
- excluir somente os próprios eventos.

As rotas internas do frontend também exigem autenticação.

As APIs REST próprias utilizam JWT do Supabase para identificar o usuário autenticado.

A identificação do usuário é obtida através da sessão, evitando a necessidade de receber `user_id` diretamente por parâmetros controlados pelo cliente.

O frontend utiliza somente credenciais apropriadas para uso client-side.

O fluxo do Make utiliza uma credencial server-side do Supabase para realizar atualizações de backend na tabela `events`.

Essa credencial é mantida exclusivamente no ambiente da automação e não é exposta no frontend, no GitHub ou na documentação pública.

---

## APIs Externas Utilizadas

### OpenStreetMap Nominatim

Utilizada para transformar o nome da cidade informada no cadastro em coordenadas geográficas.

Fluxo:

```text
Cidade
↓
Nominatim
↓
Latitude + Longitude
```

As coordenadas retornadas são utilizadas posteriormente na consulta meteorológica.

### Open-Meteo

Utilizada para consultar a previsão climática correspondente à cidade, data e horário do evento.

Entre os dados utilizados estão:

- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento.

O processamento utiliza especificamente o horário cadastrado para o evento.

---

## APIs REST Próprias

Além de consumir APIs externas, o ClimaSeguro possui APIs REST próprias implementadas com Supabase Edge Functions.

### Validate Event

Endpoint:

```http
POST /functions/v1/validate-event
```

Responsável pela validação dos dados de um evento antes de sua persistência.

Valida:

- nome;
- tipo;
- cidade;
- data;
- horário;
- e-mail do responsável.

O campo `type` aceita somente:

```text
Interno
Externo
```

A API exige usuário autenticado.

Fluxo implementado:

```text
Formulário Novo Evento
↓
validate-event
↓
Dados válidos?
├── Não → exibe erros
└── Sim → INSERT em public.events
```

Quando o evento é válido, o frontend insere o registro com:

```text
processing_status = pending
```

e recebe o UUID real do evento.

Esse UUID é enviado ao Make como:

```text
event_id
```

### Events Summary

Endpoint:

```http
GET /functions/v1/events-summary
```

A API aceita o timezone do cliente:

```http
GET /functions/v1/events-summary?timezone=<IANA_TIMEZONE>
```

Exemplos:

```http
GET /functions/v1/events-summary?timezone=Asia/Tokyo
```

```http
GET /functions/v1/events-summary?timezone=America/Sao_Paulo
```

A função é responsável por gerar um resumo dos eventos pertencentes ao usuário autenticado.

A resposta inclui:

- total de eventos;
- eventos futuros;
- eventos normais;
- eventos em atenção;
- eventos críticos;
- eventos pendentes;
- próximo evento;
- timezone utilizado;
- data atual considerada;
- horário atual considerado.

Contrato utilizado pelo frontend:

```text
summary.total_events
summary.future_events
summary.risk_summary.normal
summary.risk_summary.attention
summary.risk_summary.critical
summary.risk_summary.pending
summary.next_event
```

---

## Documentação das APIs

A documentação técnica completa das APIs REST próprias está disponível em:

```text
docs/rest-apis.md
```

O documento apresenta:

- endpoints;
- métodos HTTP;
- autenticação;
- requests;
- responses;
- códigos HTTP;
- regras de negócio;
- testes;
- arquitetura;
- integração com o frontend;
- tratamento de timezone;
- justificativas técnicas.

Os testes das APIs também estão registrados em:

```text
docs/api-tests.md
```

---

## Processamento Climático

O webhook do Make recebe os seguintes dados:

```text
event_id
nome_evento
tipo
cidade
data_evento
horario_evento
email_responsavel
```

O campo `event_id` permite que o Make atualize exatamente o mesmo registro criado anteriormente no Supabase.

Fluxo:

```text
event_id
↓
Make
↓
Nominatim
↓
Open-Meteo
↓
Variáveis climáticas
↓
Risco
↓
Recomendação
↓
Airtable
↓
PATCH no Supabase
```

---

## Classificação de Risco Climático

O ClimaSeguro classifica os eventos em três níveis:

```text
Normal
Atenção
Crítico
```

A análise utiliza informações como:

- temperatura;
- sensação térmica;
- probabilidade de chuva;
- precipitação;
- velocidade do vento;
- tipo do evento.

Eventos classificados como `Externo` possuem maior sensibilidade às condições climáticas.

A lógica de classificação é executada no Make.

---

## Recomendações

Após a classificação, o sistema gera uma recomendação correspondente ao risco identificado.

### Normal

Condições favoráveis no momento. Recomenda-se manter o acompanhamento até a realização do evento.

### Atenção

Recomenda-se reforçar medidas preventivas, acompanhar a previsão e preparar alternativas para a atividade.

### Crítico

Recomenda-se reavaliar a realização do evento, considerar mudança de horário, local coberto ou adiamento.

---

## Atualização do Supabase pelo Make

Após o processamento climático, o Make utiliza a Supabase REST API para atualizar o registro correspondente na tabela `events`.

A atualização é realizada utilizando o `event_id`.

Exemplo conceitual:

```text
PATCH /rest/v1/events?id=eq.<event_id>
```

Entre os campos atualizados estão:

- `country`
- `latitude`
- `longitude`
- `temperature`
- `apparent_temperature`
- `humidity`
- `rain_probability`
- `precipitation`
- `wind_speed`
- `weather_status`
- `recommendation`
- `forecast_status`
- `processing_status`
- `weather_updated_at`

Após o processamento bem-sucedido:

```text
processing_status = processed
```

Essa etapa permite que os resultados calculados pela automação sejam disponibilizados novamente para a aplicação.

---

## Alertas

Quando o risco climático exige atenção, o Make envia uma notificação por e-mail ao responsável informado no cadastro do evento.

O alerta apresenta:

- nome do evento;
- tipo;
- cidade;
- data;
- horário;
- classificação;
- temperatura;
- sensação térmica;
- probabilidade de chuva;
- velocidade do vento;
- recomendação.

Os e-mails são enviados utilizando uma conta exclusiva do projeto ClimaSeguro.

Existe um filtro no fluxo para impedir o envio desnecessário quando o evento está classificado como `Normal`.

Fluxo:

```text
weather_status
↓
Normal?
├── Sim → não envia alerta
└── Não → Gmail
```

---

## Supabase e Airtable

O uso conjunto de Supabase e Airtable é uma decisão arquitetural intencional do projeto.

### Supabase

O Supabase atua como fonte de verdade transacional do ClimaSeguro.

Responsabilidades:

- autenticação;
- usuários;
- perfis;
- eventos;
- segurança;
- Row Level Security;
- APIs próprias;
- dados utilizados pelo frontend;
- armazenamento dos resultados processados.

### Airtable

O Airtable é mantido como uma camada complementar de integração e auditoria operacional.

Responsabilidades:

- espelhar dados processados;
- facilitar a inspeção dos resultados;
- acompanhar visualmente as automações;
- permitir conferência operacional;
- manter uma camada no-code de integração.

Dessa forma, Supabase e Airtable não representam duplicação acidental.

O Supabase atende às necessidades transacionais, de autenticação e segurança da aplicação.

O Airtable oferece visibilidade operacional, facilidade de inspeção e integração no-code.

A separação também preserva a evolução do projeto, que inicialmente utilizava o Airtable como base principal antes da introdução de autenticação, usuários e isolamento de dados.

---

## Interface

O frontend foi desenvolvido no Lovable.

As principais telas são:

- Login
- Cadastro
- Dashboard
- Novo Evento
- Meus Eventos
- Detalhes do Evento
- Configurações

A aplicação possui autenticação real integrada ao Supabase.

Os dados mockados principais foram substituídos por dados reais.

---

## Novo Evento

A tela Novo Evento utiliza dados reais.

Fluxo implementado:

```text
Usuário preenche formulário
↓
validate-event
↓
Dados válidos
↓
INSERT em public.events
↓
processing_status = pending
↓
UUID real retornado
↓
event_id enviado ao Make
↓
Usuário é redirecionado
```

Enquanto o Make processa o evento, a interface exibe:

```text
Processando previsão
```

Após a atualização do Supabase, o evento passa a exibir o status climático real.

---

## Meus Eventos

A tela Meus Eventos deixou de utilizar mocks.

Os eventos são carregados diretamente de:

```text
public.events
```

A consulta respeita as policies RLS do Supabase.

A interface exibe:

- nome;
- cidade;
- data;
- horário;
- tipo;
- status climático;
- status de processamento.

Estados apresentados:

```text
pending
→ Processando previsão

processed
→ Exibe weather_status

error
→ Erro no processamento
```

---

## Detalhes do Evento

A tela Detalhes do Evento utiliza dados reais armazenados no Supabase.

São exibidos:

- nome do evento;
- cidade;
- data;
- horário;
- tipo;
- e-mail do responsável;
- status de processamento;
- status climático;
- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento;
- recomendação.

Campos utilizados:

```text
temperature
apparent_temperature
humidity
rain_probability
precipitation
wind_speed
weather_status
recommendation
```

Durante a validação da interface foi identificado que a recomendação estava sendo exibida com aspas extras.

A causa era a combinação de aspas já existentes no texto com aspas adicionadas pelo JSX.

O frontend foi corrigido para exibir somente o conteúdo de `recommendation`.

---

## Dashboard

O Dashboard utiliza dados reais fornecidos pela API:

```http
GET /functions/v1/events-summary
```

Os cards exibem:

- total de eventos;
- eventos futuros;
- normais;
- atenção;
- críticos;
- pendentes.

O painel também apresenta o próximo evento.

Durante os testes foi identificado um erro de binding no card `Atenção`.

O frontend buscava:

```text
summary.attention
summary.atencao
```

mas o contrato correto da API é:

```text
summary.risk_summary.attention
```

A correção foi realizada no frontend.

Após o ajuste, o Dashboard passou a apresentar corretamente os dados retornados pela API.

---

## Eventos Futuros e Timezone Dinâmico

Durante os testes foi identificado que o cálculo de eventos futuros não poderia depender de um fuso horário fixo.

Inicialmente, a Edge Function `events-summary` utilizava:

```text
Asia/Tokyo
```

Essa abordagem funcionava durante o desenvolvimento realizado no Japão, mas produziria resultados incorretos para usuários acessando o sistema em outros países, como o Brasil.

A solução adotada foi tornar o timezone dinâmico.

A API passou a aceitar o timezone através da query string:

```http
GET /functions/v1/events-summary?timezone=Asia/Tokyo
```

ou:

```http
GET /functions/v1/events-summary?timezone=America/Sao_Paulo
```

O parâmetro utiliza identificadores IANA de timezone.

Exemplos:

```text
Asia/Tokyo
America/Sao_Paulo
America/New_York
Europe/London
```

A função valida o timezone recebido.

Caso o valor seja inválido ou não seja informado, utiliza:

```text
UTC
```

como fallback seguro.

O cálculo de eventos futuros considera:

```text
data do evento
+
horário do evento
+
timezone do usuário
```

A resposta da API também informa:

```text
timezone
current_date
current_time
```

Isso facilita testes, auditoria e diagnóstico do comportamento temporal da aplicação.

---

## Integração Automática de Timezone no Frontend

O frontend detecta automaticamente o timezone do navegador através de:

```javascript
Intl.DateTimeFormat().resolvedOptions().timeZone
```

Exemplos:

```text
Usuário no Japão
→ Asia/Tokyo

Usuário em São Paulo
→ America/Sao_Paulo
```

O valor detectado é enviado automaticamente para a API:

```text
/functions/v1/events-summary?timezone=<timezone-do-navegador>
```

A URL utiliza `encodeURIComponent` para codificar corretamente o valor.

Caso o navegador não retorne um timezone válido, o frontend utiliza:

```text
UTC
```

como fallback.

Nenhuma configuração manual de timezone é necessária para o usuário.

---

## Teste de Timezone na API

A API `events-summary` foi testada no mesmo instante utilizando diferentes fusos horários.

### Asia/Tokyo

A API identificou:

```text
timezone = Asia/Tokyo
current_date = 2026-08-26
```

Nesse contexto, os eventos cadastrados já haviam ocorrido.

Resultado:

```text
future_events = 0
next_event = null
```

### America/Sao_Paulo

No mesmo instante, a API identificou:

```text
timezone = America/Sao_Paulo
current_date = 2026-08-25
```

Nesse contexto, ainda existia um evento futuro.

Resultado:

```text
future_events = 1
next_event = Teste Frontend ClimaSeguro
```

O teste comprovou que o resultado da API muda corretamente de acordo com o timezone informado pelo cliente.

Evidência:

```text
screenshots/60-events-summary-dynamic-timezone-test.png
```

---

## Teste de Timezone pelo Frontend

Após a integração automática do timezone no Lovable, foram realizados testes diretamente no Dashboard.

### Teste no Japão

Com o navegador utilizando:

```text
Asia/Tokyo
```

o Dashboard apresentou:

```text
Eventos Futuros = 00
Próximo Evento = Nenhum evento futuro
```

Esse resultado estava correto porque os eventos cadastrados já haviam ocorrido no horário local do Japão.

### Teste simulando São Paulo

O Chrome DevTools foi utilizado para simular:

```text
Location = São Paulo
Timezone ID = America/Sao_Paulo
```

Após a atualização da página, o frontend detectou automaticamente o novo timezone.

O Dashboard passou a apresentar:

```text
Eventos Futuros = 01
Próximo Evento = Teste Frontend ClimaSeguro
```

Isso confirmou que:

```text
navegador
↓
timezone local
↓
Lovable
↓
events-summary
↓
cálculo temporal correto
↓
Dashboard
```

funciona de forma dinâmica.

Evidências:

```text
screenshots/63-lovable-dashboard-timezone-japan-success.png
screenshots/64-lovable-dashboard-timezone-brazil-success.png
```

---

## Correção do Painel Próximo Evento

Durante os testes de timezone foi identificado um problema adicional no Dashboard.

Quando a API retornava:

```text
future_events = 0
next_event = null
```

o frontend utilizava incorretamente o primeiro evento da lista como fallback.

Isso fazia com que um evento já ocorrido aparecesse no painel `Próximo Evento`.

O fallback foi removido.

O comportamento correto passou a ser:

```text
summary.next_event existe
→ exibe próximo evento

summary.next_event = null
→ exibe "Nenhum evento futuro"
```

Essa correção garante que o painel reflita exatamente o contrato da API.

Evidência da correção:

```text
screenshots/62-lovable-next-event-fallback-fix.png
```

---

## Configurações da Aplicação

A tela de Configurações inicialmente possuía controles para limites climáticos de chuva e vento.

Esses campos foram removidos porque os limites climáticos são atualmente definidos pelo backend.

A interface informa:

```text
Os limites climáticos são definidos pelo backend e não são editáveis aqui.
```

São exibidos:

- nome da instituição;
- e-mail para notificações.

Os campos são apresentados apenas como informação.

O botão `Salvar preferências` foi removido porque não possuía persistência real.

Essa decisão evita apresentar controles que aparentem possuir funcionalidade sem produzir efeito no sistema.

---

## Decisão de Escopo

O projeto inicialmente foi desenvolvido como uma central genérica de consulta climática.

Durante a evolução da solução, o escopo foi refinado para atender a um problema mais específico e relevante: o monitoramento de riscos climáticos em eventos institucionais.

Essa evolução deu origem ao ClimaSeguro.

A mudança permitiu adicionar:

- contexto de negócio;
- cadastro de eventos;
- autenticação;
- usuários institucionais;
- classificação de risco;
- recomendações;
- alertas;
- banco de dados;
- APIs próprias;
- integração entre frontend e automação.

A evolução foi realizada de forma incremental, preservando as integrações já desenvolvidas e ampliando gradualmente a arquitetura.

---

## Estratégia de Integração

O projeto utiliza uma arquitetura híbrida no-code e low-code.

Cada ferramenta possui uma responsabilidade específica:

```text
Lovable
→ interface

Supabase
→ autenticação, banco, segurança e APIs

Make
→ orquestração e automação

Nominatim
→ geocodificação

Open-Meteo
→ previsão meteorológica

Airtable
→ auditoria operacional no-code

Gmail
→ alertas
```

Essa divisão permite manter:

- simplicidade;
- baixo custo;
- velocidade de desenvolvimento;
- clareza arquitetural;
- possibilidade de demonstração acadêmica;
- integração entre múltiplos serviços.

---

## Evidências

As evidências técnicas do projeto estão organizadas na pasta:

```text
screenshots/
```

Entre elas estão registros de:

- webhook;
- geocodificação;
- consulta à Open-Meteo;
- processamento das variáveis climáticas;
- classificação de risco;
- recomendação;
- persistência no Airtable;
- envio de alerta por e-mail;
- telas do Lovable;
- autenticação;
- confirmação de e-mail;
- Supabase Auth;
- tabela `profiles`;
- tabela `events`;
- políticas RLS;
- Supabase Edge Functions;
- execução das APIs REST próprias;
- inclusão do `event_id` no webhook;
- atualização do Supabase através do Make;
- evento atualizado com dados meteorológicos;
- execução completa do cenário;
- alerta enviado após a integração com o Supabase;
- criação real de evento pelo frontend;
- estado de processamento;
- exibição de dados reais;
- Dashboard real;
- correção de binding do card Atenção;
- tela de detalhes;
- tela de configurações final;
- correção da recomendação;
- teste da API com timezone dinâmico;
- teste do frontend no timezone do Japão;
- correção do fallback de próximo evento;
- teste do frontend simulando timezone do Brasil.

---

## Fluxo Ponta a Ponta Validado

O fluxo atual foi validado diretamente pelo frontend:

```text
Usuário
↓
Lovable
↓
validate-event
↓
Supabase events
↓
processing_status = pending
↓
event_id
↓
Make
↓
Nominatim
↓
Open-Meteo
↓
Tratamento de dados
↓
Classificação de risco
↓
Recomendação
↓
Airtable
↓
PATCH no Supabase
↓
processing_status = processed
↓
Filtro de alerta
↓
Gmail
↓
Lovable exibe resultado atualizado
```

Durante o teste, um evento criado diretamente pela interface foi:

- validado pela API própria;
- inserido no Supabase;
- enviado ao Make;
- geocodificado;
- processado com dados da Open-Meteo;
- classificado;
- registrado no Airtable;
- atualizado no Supabase;
- exibido na tela Meus Eventos;
- exibido na tela Detalhes do Evento;
- contabilizado no Dashboard;
- utilizado pela API `events-summary`;
- enviado por e-mail quando o risco exigiu alerta.

---

## Fluxo Temporal Validado

O fluxo de timezone também foi validado de ponta a ponta:

```text
Navegador
↓
Intl.DateTimeFormat().resolvedOptions().timeZone
↓
Lovable
↓
timezone na query string
↓
events-summary
↓
data e horário locais
↓
future_events
↓
next_event
↓
Dashboard
```

Esse fluxo foi testado com:

```text
Asia/Tokyo
America/Sao_Paulo
```

e apresentou resultados diferentes e corretos de acordo com a localização simulada do usuário.

---

## Testes e Ajustes Identificados

Durante os testes de integração foram encontrados e corrigidos problemas reais de integração.

### Binding do card Atenção

Problema:

```text
Dashboard mostrava Atenção = 00
```

Mesmo com dois eventos classificados como Atenção.

Diagnóstico:

```text
API retornava risk_summary.attention = 2
```

Causa:

```text
frontend buscava valor no nível raiz
```

Correção:

```text
summary.risk_summary.attention
```

### Eventos futuros

Problema inicial:

```text
evento já ocorrido no mesmo dia ainda aparecia como próximo evento
```

Causa:

```text
comparação considerava apenas a data
```

Correção:

```text
data + horário
```

### Timezone fixo

Problema:

```text
Asia/Tokyo estava definido diretamente na Edge Function
```

Impacto:

Um professor acessando o sistema no Brasil poderia receber uma contagem incorreta de eventos futuros.

Correção:

```text
timezone dinâmico recebido pela query string
+
validação do identificador IANA
+
fallback para UTC
```

### Timezone não integrado ao frontend

Problema:

A API já aceitava timezone dinâmico, mas o frontend ainda precisava enviar automaticamente o timezone do usuário.

Correção:

```text
Intl.DateTimeFormat().resolvedOptions().timeZone
↓
encodeURIComponent
↓
events-summary?timezone=...
```

### Fallback incorreto no próximo evento

Problema:

Quando `next_event` era `null`, o frontend exibia o primeiro evento da lista.

Correção:

```text
next_event = null
→ Nenhum evento futuro
```

### Recomendação com aspas duplicadas

Problema:

```text
recomendação aparecia com aspas extras
```

Causa:

```text
JSX adicionava aspas ao texto que já possuía aspas
```

Correção:

```text
renderização direta de recommendation
```

### Configurações sem persistência

Problema:

```text
botão Salvar preferências não possuía efeito real
```

Correção:

```text
remoção do botão
+
campos transformados em informação somente leitura
```

Esses ajustes foram mantidos no projeto porque representam melhorias de consistência, confiabilidade e alinhamento entre interface e backend.

---

## Próximos Passos

- Validar o fluxo com evento classificado como `Normal`
- Validar o comportamento do filtro de alerta para evento `Normal`
- Validar o fluxo com evento classificado como `Crítico`
- Validar o envio de alerta para evento `Crítico`
- Revisar documentação final
- Organizar evidências finais
- Preparar roteiro de apresentação
- Preparar vídeo acadêmico
- Realizar revisão final antes da entrega

---

## Status Atual

🚧 Em fase final de validação

Atualmente o projeto possui:

```text
Frontend real
+
Autenticação
+
Banco de dados
+
Segurança RLS
+
Automação Make
+
APIs externas
+
APIs REST próprias
+
Geocodificação
+
Previsão climática
+
Classificação de risco
+
Recomendações
+
Airtable
+
Atualização automática do Supabase
+
Dashboard real
+
Meus Eventos real
+
Detalhes do Evento real
+
Alertas por e-mail
+
Timezone dinâmico na API
+
Timezone automático no frontend
+
Validação Japão/Brasil
+
Fluxo ponta a ponta validado
```

O ClimaSeguro já possui um fluxo funcional completo entre frontend, APIs, banco de dados, automação e serviços externos.

A etapa atual é concluir os testes dos cenários climáticos restantes, revisar as evidências e preparar a documentação e apresentação acadêmica.