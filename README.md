# ClimaSeguro

Central Inteligente de Monitoramento de Risco Climático para Eventos Educacionais.

Projeto desenvolvido para a disciplina de **Integração de APIs** da UniFECAF.

## Objetivo

O ClimaSeguro é uma aplicação voltada a instituições de ensino que realizam eventos, aulas especiais, excursões e atividades presenciais que podem ser impactadas por condições climáticas.

A solução permite cadastrar eventos, consultar automaticamente informações meteorológicas para o local e horário planejados, classificar o nível de risco operacional e emitir alertas para os responsáveis quando forem identificadas condições que mereçam atenção.

O projeto utiliza integração entre APIs, automação, tratamento de dados, banco de dados no-code e interface web.

## Problema

Instituições educacionais frequentemente organizam atividades internas e externas sem possuir um processo centralizado para acompanhar as condições climáticas relacionadas a esses eventos.

Na prática, o responsável precisa consultar serviços meteorológicos manualmente, interpretar os dados e decidir se alguma ação preventiva é necessária.

Esse processo pode gerar:

* consultas repetitivas;
* falta de acompanhamento de mudanças na previsão;
* dificuldade de priorização;
* risco de decisões tardias;
* ausência de histórico das análises realizadas.

## Solução

O ClimaSeguro centraliza esse processo.

O usuário autenticado poderá cadastrar um evento informando:

* nome do evento;
* tipo de evento: interno ou externo;
* cidade;
* data;
* horário;
* e-mail do responsável.

A aplicação utilizará APIs externas para identificar a localização e consultar a previsão meteorológica relacionada ao evento.

Os dados serão tratados pelo Make, classificados segundo regras operacionais e armazenados no Airtable.

Quando forem identificadas condições de atenção ou risco crítico, o responsável poderá receber um alerta por e-mail.

## Arquitetura Prevista

Usuário autenticado
→ Lovable
→ Webhook do Make
→ Nominatim / OpenStreetMap
→ Open-Meteo Forecast
→ Tratamento e classificação de risco
→ Airtable
→ Alerta por e-mail
→ Dashboard no Lovable

## Tecnologias

* Lovable — interface web
* Supabase Auth — autenticação de usuários
* Make — automação e orquestração das integrações
* Airtable — banco de dados no-code
* Nominatim / OpenStreetMap — geocodificação
* Open-Meteo — dados e previsão meteorológica
* GitHub — versionamento e documentação

## Evolução do Projeto

### MVP Técnico Inicial

A primeira versão do projeto foi construída como uma Central Inteligente de Monitoramento Climático.

O objetivo inicial foi validar tecnicamente a arquitetura de integração:

Webhook
→ Nominatim
→ Open-Meteo
→ Tratamento
→ Airtable

Essa etapa comprovou que era possível:

* receber uma cidade por webhook;
* consultar uma API de geocodificação;
* utilizar as coordenadas em uma segunda API;
* manipular os dados recebidos;
* classificar as informações;
* armazenar o resultado em um banco no-code.

### Pivot de Produto

Após a validação técnica do MVP inicial, foi identificado que uma aplicação destinada apenas à consulta de condições climáticas apresentava baixo valor como produto.

A estratégia foi então reposicionada para resolver uma necessidade mais específica: apoiar instituições de ensino no monitoramento de riscos climáticos relacionados a eventos e atividades programadas.

O backend desenvolvido inicialmente foi preservado e passou a ser evoluído para trabalhar com:

* cadastro de eventos;
* previsão meteorológica;
* data e horário do evento;
* diferenciação entre atividades internas e externas;
* classificação de risco;
* recomendações operacionais;
* alertas automáticos por e-mail;
* monitoramento periódico de eventos futuros.

Esse pivot demonstra uma evolução orientada por valor: a tecnologia validada no primeiro protótipo foi reaproveitada para construir uma solução mais útil e contextualizada.

## Fluxo Técnico Validado

A primeira versão funcional já possui:

Webhook
→ Nominatim
→ Open-Meteo
→ Regra de classificação
→ Airtable

## Funcionalidades Já Implementadas

* recebimento de dados por webhook;
* geocodificação utilizando API externa;
* integração entre duas APIs distintas;
* consulta de dados meteorológicos;
* tratamento dos dados no Make;
* classificação operacional;
* persistência dos registros no Airtable;
* autenticação OAuth entre Make e Airtable.

## Próximas Funcionalidades

* cadastro estruturado de eventos;
* previsão meteorológica para data e horário;
* classificação de risco específica para eventos;
* diferenciação entre evento interno e externo;
* armazenamento dos eventos;
* alertas por e-mail;
* monitoramento periódico;
* autenticação de usuários;
* dashboard web;
* histórico de eventos e análises.

## Evidências do MVP Técnico

Os screenshots da primeira etapa estão disponíveis na pasta `screenshots/`.

### Dia 1 — Validação das Integrações

1. `01-webhook-success.png` — recebimento de dados pelo webhook
2. `02-nominatim-geocoding-success.png` — geocodificação utilizando Nominatim
3. `03-open-meteo-weather-success.png` — consulta à Open-Meteo
4. `04-status-variable-success.png` — tratamento e classificação dos dados
5. `05-make-airtable-module-success.png` — execução do módulo Airtable no Make
6. `06-airtable-table-record-success.png` — registro persistido no banco
7. `07-make-full-workflow.png` — cenário completo da primeira versão do backend

## Segurança

A arquitetura considera princípios de segurança e minimização de acesso.

* APIs públicas são utilizadas apenas para informações necessárias ao funcionamento da solução.
* A conexão entre Make e Airtable utiliza OAuth.
* O acesso do Make ao Airtable é limitado à base utilizada pelo projeto.
* URLs de webhook, tokens e credenciais não devem ser armazenados no repositório público.
* Dados pessoais desnecessários não são enviados às APIs meteorológicas.
* O e-mail do responsável será utilizado exclusivamente para o envio dos alertas relacionados aos eventos cadastrados.

## Privacidade e LGPD

O projeto segue o princípio de minimização de dados.

As APIs de localização e clima não precisam receber informações pessoais do usuário. Os dados de identificação utilizados pela aplicação deverão ser limitados ao necessário para autenticação e comunicação dos alertas.

Em uma implementação de produção, também seriam necessários mecanismos de:

* controle de acesso;
* política de retenção;
* exclusão de dados;
* gestão de consentimento;
* registro de operações;
* proteção de credenciais.

## Limitações

O ClimaSeguro é um protótipo acadêmico.

As classificações de risco utilizadas são regras operacionais definidas para demonstração e não substituem alertas meteorológicos oficiais ou orientações de autoridades competentes.

A disponibilidade de previsões também depende da janela temporal oferecida pela API meteorológica utilizada.

## Status

🚧 Em desenvolvimento — backend em evolução após validação do MVP técnico.
