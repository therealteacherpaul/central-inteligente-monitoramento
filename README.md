# ClimaSeguro

Projeto acadêmico desenvolvido para a disciplina de Integração de APIs da UniFECAF, no contexto do curso de IA e Automação Digital.

## Objetivo

O ClimaSeguro é uma aplicação voltada ao apoio à gestão de eventos institucionais, com foco na análise de riscos climáticos.

A plataforma permite cadastrar eventos, consultar condições meteorológicas previstas, classificar o risco climático, gerar recomendações e emitir alertas aos responsáveis.

O projeto combina autenticação, banco de dados, automações, APIs externas, APIs REST próprias e uma interface web desenvolvida com ferramentas no-code e low-code.

---

## Problema

Instituições de ensino realizam eventos internos e externos que podem ser impactados por chuva, calor excessivo, vento forte e outras condições climáticas.

O ClimaSeguro foi criado para reduzir esse risco operacional, concentrando em um único fluxo:

- cadastro do evento;
- consulta da previsão;
- análise de risco;
- recomendação;
- armazenamento dos resultados;
- atualização do banco principal;
- registro operacional complementar;
- alerta ao responsável.

---

## Arquitetura Atual

Fluxo da aplicação:

```text
Usuário
   ↓
Lovable
   ↓
Supabase Auth
   ↓
Supabase Database
   ↓
APIs REST próprias
```

Fluxo de processamento climático:

```text
Evento
   ↓
Webhook do Make
   ↓
OpenStreetMap Nominatim
   ↓
Open-Meteo
   ↓
Tratamento dos dados
   ↓
Classificação de risco
   ↓
Geração de recomendação
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
- Duas APIs REST próprias implementadas com Supabase Edge Functions
- API de validação de eventos
- API de resumo de eventos para o Dashboard
- Autenticação das APIs com JWT
- Integração das APIs com as políticas RLS do Supabase
- Fluxo ponta a ponta validado com atualização do Supabase e envio de alerta por e-mail

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

O fluxo do Make utiliza uma credencial server-side do Supabase para realizar atualizações de backend na tabela `events`.

Essa credencial é mantida exclusivamente no ambiente da automação e não é exposta no frontend, no GitHub ou na documentação pública.

---

## APIs Externas Utilizadas

### OpenStreetMap Nominatim

Utilizada para transformar o nome da cidade em coordenadas geográficas.

Fluxo:

```text
Cidade
↓
Nominatim
↓
Latitude + Longitude
```

Os dados retornados são utilizados posteriormente na consulta meteorológica.

### Open-Meteo

Utilizada para consultar a previsão climática correspondente à cidade, data e horário do evento.

Entre os dados utilizados estão:

- temperatura;
- sensação térmica;
- umidade;
- probabilidade de chuva;
- precipitação;
- velocidade do vento.

O processamento é realizado considerando o horário específico do evento.

---

## APIs REST Próprias

Além de consumir APIs externas, o ClimaSeguro também possui APIs REST próprias.

As APIs foram implementadas utilizando Supabase Edge Functions.

### Validate Event

Endpoint:

```http
POST /functions/v1/validate-event
```

Responsável pela validação dos dados de um evento.

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

Fluxo previsto:

```text
Formulário Novo Evento
↓
validate-event
↓
Dados válidos?
├── não → retorna erros
└── sim → continua cadastro
```

### Events Summary

Endpoint:

```http
GET /functions/v1/events-summary
```

Responsável por gerar um resumo dos eventos pertencentes ao usuário autenticado.

A resposta inclui:

- total de eventos;
- eventos futuros;
- eventos normais;
- eventos em atenção;
- eventos críticos;
- eventos pendentes;
- próximo evento.

Essa API foi criada para alimentar os indicadores do Dashboard.

Fluxo previsto:

```text
Dashboard
↓
events-summary
↓
Supabase events
↓
Indicadores consolidados
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
- justificativas técnicas.

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
├── sim → não envia alerta
└── não → Gmail
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
- Configurações

A aplicação já possui autenticação real integrada ao Supabase.

Os dados dos eventos ainda estão em processo de migração dos mocks do frontend para a tabela real `events`.

A próxima etapa da interface será conectar:

```text
Novo Evento
→ validate-event
→ Supabase events
→ Make
```

e:

```text
Dashboard
→ events-summary
→ dados reais
```

---

## Configurações da Aplicação

A interface atualmente possui uma tela de configurações com campos de limites climáticos.

Esses campos serão simplificados porque os limites ainda não são controlados dinamicamente pelo frontend.

A decisão é manter apenas configurações que possuam efeito real no backend, evitando que a interface apresente opções que não são utilizadas pela lógica do sistema.

Essa alteração será documentada como uma decisão de consistência funcional e redução consciente de escopo.

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
- alerta enviado após a integração com o Supabase.

---

## Fluxo Ponta a Ponta Validado

O backend atual já foi validado com o seguinte fluxo:

```text
Evento existente no Supabase
↓
event_id enviado ao webhook
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
Registro no Airtable
↓
PATCH no Supabase
↓
processing_status = processed
↓
Filtro de alerta
↓
E-mail enviado
```

Durante o teste, o evento foi atualizado corretamente no Supabase com:

- localização;
- dados meteorológicos;
- status climático;
- recomendação;
- status da previsão;
- status de processamento.

O alerta por e-mail também foi enviado com sucesso após a atualização do banco.

---

## Próximos Passos

- Integrar o formulário de Novo Evento à tabela `events`
- Utilizar a API `validate-event` no cadastro de eventos
- Fazer o frontend enviar o `event_id` ao webhook do Make
- Substituir os dados mockados da tela Meus Eventos
- Utilizar a API `events-summary` no Dashboard
- Exibir os resultados processados do Supabase na interface
- Simplificar a tela de Configurações
- Manter somente configurações realmente utilizadas pelo backend
- Realizar testes completos de ponta a ponta pelo frontend
- Validar o fluxo com múltiplos usuários
- Atualizar a documentação final
- Preparar o vídeo de apresentação acadêmica

---

## Status Atual

🚧 Em desenvolvimento

Atualmente o projeto já possui:

```text
Frontend
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
Alertas por e-mail
```

O backend do processamento climático já está integrado ao Supabase e foi validado ponta a ponta.

A próxima etapa principal é conectar os eventos reais do Supabase ao frontend do ClimaSeguro e substituir os dados mockados da interface.