# Domínios do Projeto — Marketplace de Serviços de Beleza

**ID do Documento:** ARCH-DOM-001
**Versão:** 1.0
**Status:** Base de referência proposta para revisão
**Produto:** Marketplace para serviços de beleza e cuidados pessoais

## 1. Objetivo

Este documento define os domínios de negócio lógicos do projeto, seus limites de responsabilidade, entidades importantes, dependências e a sequência de implementação recomendada. Um domínio lógico não é, por si só, um bom candidato a microsserviço independente: a implementação autônoma deve ser justificada por uma responsabilidade de negócio coesa, propriedade independente dos dados e um custo de integração gerenciável.

A jornada principal do marketplace é:

**Encontrar um profissional → comparar uma oferta → escolher um horário → agendar → pagar → receber o serviço → avaliar.**

A lista atual contém 11 domínios lógicos. A revisão abaixo mantém todas as 11 capacidades, mas ajusta as responsabilidades e recomenda que nem todas se tornem microsserviços implantados separadamente na primeira versão.

## 2. Principais correções e decisões

1. **Agendamento e disponibilidade possuem uma forte relação de consistência.** Um horário exibido pode se tornar obsoleto, e o Kafka opera de forma assíncrona. Se a disponibilidade de agenda e os agendamentos confirmados forem gerenciados por serviços distintos desde o início, o sistema exigirá um protocolo de reserva/bloqueio bem estruturado para evitar agendamentos duplicados. Para o MVP, implemente os domínios 5 e 6 como dois módulos dentro de um único **Serviço de Agendamentos**. Mantenha os modelos de domínio separados, permitindo uma futura divisão, caso as necessidades operacionais justifiquem.
2. **O Domínio 11, Administração da Plataforma, é, essencialmente, uma aplicação ou capacidade administrativa, e não o proprietário natural de todos os dados que gerencia.** Os dados de usuário permanecem sob a responsabilidade de "Identidade"; o status de perfil, sob "Perfis"; categorias e ofertas, sob "Catálogo"; e casos de moderação, sob "Moderação". Não crie um microsserviço de administração que duplique as regras de negócio desses domínios. Comece com uma interface administrativa (Admin UI/BFF) ou uma camada de aplicação que invoque os serviços proprietários dos dados. Uma capacidade de auditoria centralizada pode ser adicionada, caso surja uma necessidade específica e bem definida.
3. **Descoberta (Discovery) e Análise de Dados (Analytics) são capacidades voltadas para a leitura.** Elas constroem visões pesquisáveis ​​ou consultáveis ​​a partir de dados que pertencem a outros domínios. Elas não devem se tornar fontes alternativas de verdade para registros de perfil, catálogo, agendamento, pagamento ou avaliação. Inicialmente, podem ser módulos ou simples modelos de leitura (*read models*), evoluindo para serviços independentes quando a escala, a latência ou a necessidade de autonomia de uma equipe assim o justificarem.
4. **O módulo de pagamentos não deve ser o responsável pelo estado do agendamento.** Um pagamento pode estar pendente ou ser estornado mesmo após o cancelamento de um agendamento. Suas máquinas de estado são distintas, mantendo a correlação por meio de identificadores (IDs).
5. **O módulo de moderação não detém a propriedade direta de contas ou avaliações.** Ele gerencia denúncias e casos de moderação; o serviço responsável pelo recurso em questão executa ações como suspensão, restrição de perfil ou ocultação de avaliações, por meio de uma operação autorizada.
6. **O Kafka destina-se a eventos de domínio assíncronos, não a consultas síncronas.** Processos como login, autorização imediata, validação final de disponibilidade e início de pagamento geralmente exigem um resultado síncrono. Eventos servem para projeções, notificações, auditoria e reações entre domínios. É fundamental planejar aspectos como idempotência no consumo, novas tentativas (*retries*), tratamento de mensagens não processáveis ​​(*dead-letter queues*), versionamento de eventos e identificadores de correlação.
7. **Não utilize a abordagem “MVC primeiro, arquitetura hexagonal depois” como justificativa para misturar regras de negócio com *controllers* ou classes do *framework*.** Ao implementar o padrão MVC, mantenha as regras de domínio em serviços de aplicação ou de domínio, garantindo a cobertura por testes unitários. Dessa forma, uma eventual migração futura será apenas uma refatoração de limites (*boundary refactoring*), e não uma reescrita completa do código.


## 3. Recommended first implementation set

### Recommended: four deployable services spanning five logical domains

| Initial service | Logical domains covered | Reason |
|---|---|---|
| 1. Identity & Access | Domain 1 | Provides account identity, authentication, roles, and authorization foundations. |
| 2. Profiles & Professionals | Domain 2 | Enables client and professional profiles and professional activation state. |
| 3. Catalog & Offers | Domain 3 | Lets professionals publish services, prices, and durations. |
| 4. Scheduling & Appointments | Domains 5 + 6 | Delivers the core booking journey while keeping slot availability and booking conflict rules transactionally close. |

This adjusts the proposed selection of **1, 2, 3, and 5**: include **domain 6 (Appointments)** inside the fourth service instead of treating domain 5 as an isolated service. This is the stronger option because a schedule without a booking owner cannot validate real booking conflicts end to end. The codebase can still maintain separate `availability` and `appointments` modules and separate use cases/aggregates.

### If the team insists on keeping domain 5 as its own microservice

That is possible, but its initial scope should be explicitly limited to working hours, exceptions, and blocks. Domain 5 must not claim that a slot is definitively bookable based only on an asynchronous projection of appointments. When domain 6 is introduced, domain 6 must perform an authoritative, atomic conflict check or implement a robust reservation protocol. The user-visible booking flow will remain incomplete until domain 6 exists.

### Important distinction: architecture practice versus product delivery

If the primary goal is learning microservices, Kafka, TDD, MVC, and later hexagonal architecture, the recommended four-service phase is a reasonable training sequence. If the primary goal is to launch the business quickly, 11 independently deployed microservices are likely unnecessary operational overhead; a modular monolith with these bounded contexts is a lower-risk starting point. The domain map remains useful in either architecture.

## 4. Domain catalogue

### Domain 1 — Identity & Access

**Purpose:** Own account identity, credentials, authentication, roles, and access policy.  
**Core entities:** `Account`, `Credential`, `RoleAssignment`, `ExternalIdentity`, `RecoveryChallenge` (names are illustrative).  
**Owns:** account ID, login identifiers, credential hashes, authentication lifecycle, roles/claims, account-level enabled/disabled status.  
**Does not own:** client profile details, professional public profile, services, appointments, payments, or moderation case content.

**Responsibilities**
- Register and authenticate users.
- Recover access and invalidate sessions/tokens where applicable.
- Support social login only if selected for the release.
- Represent the roles `CLIENT`, `PROFESSIONAL`, and `ADMIN` and enforce authorization policies.
- Publish minimal account lifecycle events for other domains.

**Inputs/dependencies:** identity credentials from the user; optional external identity provider.  
**Outputs/events:** `identity.account-created.v1`, `identity.account-status-changed.v1`, `identity.role-assigned.v1`.  
**Critical rules:** no plaintext passwords; unique normalized login identifier; server-side authorization; account ID is stable and never reused.

### Domínio 2 — Perfis e Profissionais

**Objetivo:** Gerenciar os dados de perfil do cliente, a apresentação pública do profissional e sua elegibilidade para listagem na plataforma.
**Entidades principais:** `ClientProfile`, `ProfessionalProfile`, `SocialLink`, `ProfessionalStatus`.
**Responsabilidades (escopo):** nome de exibição, campos de perfil/contato aprovados para o produto, descrição pública, área de atendimento/localização (com o nível de precisão definido), links para redes sociais externas e status do perfil público.
**Fora do escopo:** credenciais de autenticação, definições de categorias, ofertas de serviços individuais, agenda, agendamentos ou status de repasses financeiros.

**Responsabilidades**
- Criar/atualizar o perfil do cliente vinculado a um `accountId`.
- Criar/atualizar o perfil do profissional e suas informações públicas.
- Monitorar o status do perfil (rascunho, ativo, suspenso ou elegível para listagem).
- Validar campos obrigatórios do perfil antes da publicação.
- Armazenar links para perfis em redes sociais externas; não inclui galeria de trabalhos hospedada internamente (no MVP).

**Dependências:** O módulo de Identidade gerencia a identidade da conta e as funções (papéis); os módulos de Catálogo e Descoberta consomem os identificadores e o status dos profissionais.
**Saídas/eventos:** `profiles.client-profile-updated.v1`, `profiles.professional-profile-activated.v1`, `profiles.professional-profile-restricted.v1`.
**Regras críticas:** campos públicos devem ser claramente distinguidos de campos privados; a verificação de identidade é uma decisão de negócio, não um requisito pressuposto.

### Domínio 3 — Catálogo de Serviços e Ofertas

**Objetivo:** Gerenciar a taxonomia da plataforma e as ofertas que cada profissional disponibiliza para venda.
**Entidades principais:** `Category`, `ServiceDefinition`, `ServiceOffer` (e, em versões futuras, possivelmente `OfferOption`/`ComboDefinition`).
**Responsabilidades (escopo):** definições padrão de categorias/serviços, nome e descrição da oferta (específicos para cada profissional), preço, duração, status (ativo/inativo) e vínculo com o profissional.
**Fora do escopo:** perfil completo do profissional, agenda, agendamentos, pagamentos, classificação em buscas ou avaliações.

**Responsabilidades**
- Manter categorias como cabelo, barba, unhas, maquiagem, sobrancelhas e estética.
- Manter definições de serviços reutilizáveis, permitindo, ao mesmo tempo, ofertas específicas por profissional e diferentes níveis de detalhamento. - Permita que profissionais criem, editem, ativem e desativem suas próprias ofertas.
- Armazene o preço e a duração estimada de cada oferta.
- Mantenha registros históricos (snapshots) de ofertas/preços vinculados a agendamentos existentes.

**Distinção importante de modelo:** `ServiceDefinition` é o conceito genérico da plataforma; `ServiceOffer` é a oferta comercial específica de um profissional. Profissionais diferentes podem cobrar preços distintos e definir durações diferentes para serviços correlatos. A taxonomia não deve obrigar todos os profissionais a utilizar uma granularidade de serviço idêntica.

**Dependências:** ID/status do perfil do profissional; os agendamentos utilizam snapshots imutáveis ​​ou identificadores, além dos termos acordados.
**Saídas/eventos:** `catalog.category-changed.v1`, `catalog.service-offer-published.v1`, `catalog.service-offer-updated.v1`, `catalog.service-offer-deactivated.v1`.
**Regras críticas:** apenas profissionais elegíveis podem publicar ofertas; ofertas ativas exigem um preço válido e uma duração positiva; alterações não devem modificar os termos históricos de agendamentos já realizados.

### Domínio 4 — Descoberta e Busca

**Objetivo:** Ajudar clientes a encontrar e comparar profissionais ativos e ofertas.
**Componentes/dados principais:** consulta de busca, filtros, projeção de resultados, estratégia de ordenação.
**Responsabilidades:** modelos de busca/leitura, semântica de filtros e comportamento de ranqueamento/consulta — não inclui o perfil de origem, o serviço, a agenda ou os registros de avaliações.

**Responsabilidades**
- Busca por categoria, serviço ou nome do profissional.
- Filtragem por área de atendimento/localização, preço, avaliação e disponibilidade (quando houver dados confiáveis).
- Exibição de perfis públicos e resumos de ofertas a partir de modelos de leitura.
- Aplicação de uma ordenação padrão transparente; não se prevê posicionamento pago (anúncios) para o MVP.
- Tratamento de resultados vazios, com sugestão de ajustes nos filtros.

**Dependências:** Perfil, Catálogo, Disponibilidade/Agendamentos, Avaliações; consome eventos para atualizar projeções.
**Saídas:** resultados de busca e telemetria de consultas. Pode publicar o evento `discovery.index-updated.v1`, caso seja útil internamente.
**Observação sobre limites:** os resultados podem apresentar consistência eventual. Os resultados de busca são sugestões; o serviço de agendamento deve revalidar a disponibilidade no momento da reserva. Para um MVP inicial, isso pode ser implementado como um módulo/modelo de leitura, em vez de um microsserviço independente.

### Domínio 5 — Agenda e Disponibilidade

**Objetivo:** Representar os períodos em que um profissional pretende e está apto a prestar serviços.
**Entidades principais:** `WorkingHours` (Horário de Trabalho), `AvailabilityException` (Exceção de Disponibilidade), `TimeBlock` (Bloco de Tempo), `SchedulePolicy` (Política de Agendamento).
**Responsabilidades:** horários de trabalho recorrentes, fuso horário, exceções, intervalos e bloqueios manuais.
**Não inclui:** o ciclo de vida de agendamentos confirmados; isso pertence ao Domínio 6 (ou ao serviço combinado de Agendamentos, na implementação inicial).

**Responsabilidades**
- Configurar horários de trabalho recorrentes e exceções.
- Adicionar, editar e remover períodos bloqueados.
- Calcular intervalos disponíveis com base na duração do serviço e nas regras de agendamento.
- Disponibilizar consultas de disponibilidade para um determinado profissional, oferta e intervalo de datas.
- Rejeitar alterações na agenda que violem, de forma silenciosa, agendamentos já confirmados (a interação final é coordenada com o serviço de agendamentos).

**Dependências:** duração do serviço (proveniente do Catálogo); agendamentos confirmados (provenientes do Domínio 6 ou do módulo equivalente). **Saídas/eventos:** `availability.schedule-updated.v1`, `availability.time-block-changed.v1`.
**Regra crítica:** disponibilidade não equivale a uma reserva. Um intervalo exibido como livre deve ser validado de forma atômica no momento da confirmação; uma projeção assíncrona via Kafka, por si só, é insuficiente para evitar reservas duplicadas.

### Domínio 6 — Agendamentos e Comparecimento

**Objetivo:** Gerenciar o agendamento concreto e seu ciclo de vida.
**Entidades principais:** `Appointment` (Agendamento), `AppointmentStatusHistory` (Histórico de Status do Agendamento), `AttendanceRecord` (Registro de Comparecimento) — nomes ilustrativos.
**Responsabilidades (escopo):** Identificadores de cliente/profissional/oferta, intervalo de data/hora, registro (snapshot) de preço/duração, estado do agendamento, metadados de cancelamento, registro de comparecimento ou ausência (no-show).
**Fora do escopo:** Estado da transação de pagamento, definições genéricas de catálogo de serviços ou modelos de agenda.

**Responsabilidades**
- Criar um agendamento após validar a elegibilidade e a disponibilidade atual.
- Impedir agendamentos conflitantes para o mesmo profissional e intervalo de tempo.
- Preservar o serviço contratado, o preço e a duração definidos no momento da reserva.
- Suportar transições permitidas: confirmação, cancelamento, conclusão e registro de ausência (no-show).
- Manter o estado do agendamento independente do estado do pagamento.
- Publicar eventos do ciclo de vida para fluxos de pagamento, notificações, avaliações e análises (analytics).

**Dependências:** Identidade/Perfil, Catálogo, Disponibilidade; o sistema de Pagamentos integra-se via ID do agendamento.
**Saídas/eventos:** `appointments.appointment-created.v1`, `appointments.appointment-confirmed.v1`, `appointments.appointment-cancelled.v1`, `appointments.appointment-completed.v1`, `appointments.appointment-no-show.v1`.
**Regra crítica:** A prevenção de conflitos é uma invariante absoluta (autoritativa), e não apenas uma consulta de busca baseada em "melhor esforço". Se este domínio estiver separado do domínio de disponibilidade, utilize um protocolo de reserva e documente o comportamento de *timeout* (expiração) e liberação.

### Domínio 7 — Pagamentos e Repasses

**Objetivo:** Gerenciar transações financeiras, reembolsos, comissões e repasses aos profissionais.
**Entidades principais:** `Payment` (Pagamento), `Transaction` (Transação), `Refund` (Reembolso), `Payout` (Repasse), `PlatformFee` (Taxa da Plataforma).
**Responsabilidades (escopo):** Status financeiro, referência do provedor de pagamento, valor/moeda, dados para cálculo de comissões, registros de reembolso/repasse, identificadores para reconciliação.
**Fora do escopo:** Status do agendamento ou agenda de compromissos.

**Responsabilidades**
- Iniciar pagamentos por meio de um provedor autorizado.
- Processar *callbacks* ou *webhooks* do provedor de forma idempotente.
- Monitorar estados como: pendente, aprovado, rejeitado, cancelado, reembolsado e outros estados suportados pelo provedor. - Aplicar o modelo de comissão e repasse (payout) aprovado.
- Conciliar transações e reportar discrepâncias aos operadores autorizados.

**Dependências:** Ciclo de vida do agendamento e provedor de pagamentos.
**Saídas/eventos:** `payments.payment-status-changed.v1`, `payments.refund-status-changed.v1`, `payments.payout-status-changed.v1`.
**Decisões pendentes:** provedor, momento do pagamento, métodos aceitos, comissão da plataforma, cronograma de repasses, política de reembolso/cancelamento. Nenhuma dessas definições deve ser assumida sem uma decisão explícita.

**Domínio 8 — Avaliações e Reputação**

**Objetivo:** Gerenciar o feedback dos clientes vinculado a agendamentos concluídos e elegíveis.
**Entidades principais:** `Review` (Avaliação), `ReviewReply` (Resposta à avaliação — se aprovada), `ReputationSummary` (Resumo de reputação).
**Responsabilidades (escopo):** nota (classificação), comentário opcional, associação ao agendamento/cliente/profissional, estado de visibilidade da avaliação, nota pública agregada.
**Fora do escopo:** registros de agendamentos, processos de moderação ou ciclo de vida da conta do cliente.

**Responsabilidades**
- Validar se um cliente pode avaliar um agendamento elegível.
- Aplicar a política contra avaliações duplicadas.
- Armazenar a nota e o comentário opcional.
- Calcular/atualizar o resumo de reputação do profissional.
- Aplicar alterações de visibilidade decorrentes de decisões de moderação autorizadas.

**Dependências:** Evento de conclusão de agendamento; Moderação.
**Saídas/eventos:** `reviews.review-submitted.v1`, `reviews.review-published.v1`, `reviews.review-hidden.v1`.
**Regra crítica:** uma avaliação deve estar vinculada a um agendamento real e elegível, e não apenas a um perfil público de um profissional.

### Domínio 9 — Denúncias e Moderação

**Objetivo:** Receber denúncias e gerenciar casos de moderação envolvendo perfis, avaliações e atividades no marketplace.
**Entidades principais:** `Report` (Denúncia), `ModerationCase` (Caso de Moderação), `ModerationDecision` (Decisão de Moderação), `EvidenceReference` (Referência de Evidência).
**Responsabilidades (escopo próprio):** registro de denúncias, status de casos, decisões de moderadores, justificativas, registros temporais (timestamps) e histórico de casos.
**Não abrange:** a conta de usuário, o perfil profissional, a avaliação, o agendamento ou o pagamento subjacentes.

**Atribuições**
- Registrar denúncias relacionadas a um usuário, perfil profissional, avaliação ou agendamento.
- Realizar a triagem e a eliminação de duplicatas de denúncias, sempre que possível.
- Permitir que a equipe autorizada investigue e registre uma decisão.
- Solicitar ou acionar ações nos domínios responsáveis ​​(proprietários dos dados), como ocultar uma avaliação ou restringir um perfil.
- Manter um registro da decisão tomada e a correlação com o comando ou evento resultante.

**Dependências:** Identidade, Perfis, Avaliações, Agendamentos e operações administrativas.
**Saídas/eventos:** `moderation.report-created.v1`, `moderation.case-decided.v1`, `moderation.action-requested.v1`.
**Observação sobre limites:** o serviço de moderação registra a decisão; o serviço responsável pelo dado original é quem executa a alteração efetiva no estado da conta, perfil ou avaliação.

### Domínio 10 — Analytics e Indicadores

**Objetivo:** Fornecer indicadores operacionais/de negócio, sem atuar como a fonte primária da verdade operacional.
**Componentes principais:** `ProfessionalMetricsProjection` (Projeção de Métricas Profissionais), `PlatformMetricsProjection` (Projeção de Métricas da Plataforma), consultas para relatórios.
**Responsabilidades (escopo próprio):** projeções, agregações, definições de métricas e lógica de consultas para relatórios.
**Não abrange:** dados brutos de agendamentos, transações de pagamento, avaliações, perfis ou ofertas.

**Atribuições**
- Calcular o número de agendamentos, atendimentos concluídos, cancelamentos, médias de avaliações, tempo ocupado/disponível e volumes de transações geradas na plataforma (conforme a disponibilidade dos dados).
- Disponibilizar dashboards básicos para profissionais e para a plataforma.
- Documentar a definição de cada métrica, fuso horário, período de referência e o tratamento aplicado a cancelamentos/reembolsos.
- Consumir eventos e reconstruir projeções, caso necessário.
- Dependências: Eventos relacionados a Agendamentos, Pagamentos, Avaliações, Catálogo e Disponibilidade.
- Saídas: relatórios e métricas; não altera os registros de origem. **Aviso de delimitação:** os dados de análise (*analytics*) podem apresentar consistência eventual e devem exibir uma indicação clara de "atualizado em". Em uma versão inicial (MVP), isso pode ser implementado como um módulo ou um modelo de leitura (*read model*), em vez de um microsserviço independente.

**Domínio 11 — Administração de Plataforma e Operações de Auditoria**

**Objetivo:** Proporcionar uma experiência administrativa autorizada e fluxos de trabalho operacionais que abrangem múltiplos domínios, sem assumir a propriedade das entidades pertencentes a outros domínios.
**Formato recomendado:** Aplicação web administrativa/BFF (*Backend for Frontend*), incluindo casos de uso administrativos sujeitos a auditoria. Não duplique bases de dados referentes a usuários, categorias, agendamentos, pagamentos ou moderação.

**Responsabilidades**
- Disponibilizar fluxos de trabalho autorizados para o gerenciamento de usuários, profissionais, categorias, relatórios, agendamentos e investigações de transações.
- Invocar a API de cada domínio proprietário ou publicar um comando aprovado, seguindo o contrato definido por esse domínio.
- Controlar o acesso administrativo por meio de mecanismos de identidade e permissões validadas no lado do servidor (*server-side*).
- Registrar quem realizou ações administrativas relevantes, quando, por que e qual foi o resultado. O armazenamento de auditoria pode ser uma funcionalidade compartilhada da plataforma ou, se devidamente justificado, um serviço à parte.
- Evitar a criação de fluxos CRUD alternativos que contornem as regras de negócio estabelecidas pelos domínios.

**Sob sua responsabilidade:** a interface administrativa (UI)/orquestração via BFF e, caso seja intencionalmente isolado, o registro de auditoria administrativa.
**Não sob sua responsabilidade:** as entidades canônicas gerenciadas pelos domínios 1 a 10.
**Dependências:** todos os domínios que expõem operações administrativas; o sistema de Identidade (para autorização); e o sistema de Moderação (para decisões sobre casos específicos).
**Recomendação:** trate isso como uma funcionalidade da aplicação, e não como um décimo primeiro microsserviço de negócio na versão inicial. Caso um serviço de auditoria dedicado venha a ser criado, defina-o como "Trilha de Auditoria" (*Audit Trail*), com responsabilidades claras quanto à retenção de eventos e controle de acesso.

## 5. Matriz de propriedade de domínios

| Dado/capacidade | Proprietário (autoridade) | Outros domínios podem |
|---|---|---|
| ID da conta, credenciais, funções | 1. Identidade e Acesso | Consumir eventos básicos de identidade ou chamar APIs autorizadas |
| Perfil de cliente/profissional | 2. Perfis e Profissionais | Criar projeções públicas/de leitura |
| Categorias, definições de serviço, ofertas, preço/duração da oferta | 3. Catálogo e Ofertas | Ler/consultar projeções; agendamentos mantêm snapshots do momento da reserva |
| Índice de busca e ordenação de resultados | 4. Descoberta e Busca | Ler eventos do domínio de origem; nunca atualizar entidades de origem diretamente |
| Horário de trabalho, exceções, bloqueios manuais | 5. Agenda e Disponibilidade | Consultar disponibilidade |
| Reserva, ciclo de vida do agendamento, comparecimento | 6. Agendamentos | Consumir eventos do ciclo de vida |
| Pagamento, reembolso, comissão, repasse | 7. Pagamentos e Repasses | Consumir status de pagamento; não possuem autoridade sobre o status do agendamento |
| Avaliações, comentários, resumo de avaliações | 8. Avaliações e Reputação | A moderação pode solicitar alteração na visibilidade |
| Relatórios e decisões de moderação | 9. Relatórios e Moderação | Enviar solicitações de ação autorizadas aos domínios proprietários |
| Métricas/projeções agregadas | 10. Analytics e Indicadores | Ler dados derivados de eventos; sem atualizações na origem |
| Fluxos de trabalho administrativos/auditoria | 11. Aplicação/operações administrativas | Invocar contratos dos domínios proprietários; sem duplicação de banco de dados |

## 6. Regras de comunicação

### Use APIs síncronas quando
- O solicitante precisar de um resultado imediato de sucesso/falha (login, verificação de permissão atual, confirmação final de agendamento, início de pagamento).
- O solicitante precisar de um valor oficial (autoritativo) e dados defasados ​​representarem um risco.
- Um usuário estiver aguardando a conclusão de um comando.

### Use eventos via Kafka quando
- Um domínio tiver consolidado uma mudança de estado e outros domínios precisarem reagir.
- Projeções para busca/analytics precisarem ser atualizadas.
- Notificações, auditoria ou fluxos de trabalho subsequentes puderem ocorrer de forma assíncrona.

### Convenções mínimas para eventos
- Utilize nomes explícitos e versionados, como `appointments.appointment-confirmed.v1`. - Inclua o ID do evento, tipo/versão do evento, registro de data e hora da ocorrência, ID do agregado/entidade, ID de correlação e ID de causalidade, quando aplicável.
- Inclua apenas o mínimo de dados necessários; não publique senhas, tokens, credenciais de pagamento completas ou dados pessoais desnecessários.
- Os consumidores devem ser idempotentes. Considere a possibilidade de entrega "pelo menos uma vez" (at-least-once delivery).
- Defina uma política de retentativa e o tratamento de mensagens não processáveis ​​(dead-letter handling); monitore o atraso (lag) e as falhas no processamento de mensagens.
- Considere o padrão *transactional outbox* para evitar a confirmação (*commit*) de uma alteração no banco de dados sem a publicação do evento correspondente.
- Não utilize o Kafka como um mecanismo improvisado de RPC síncrono.

## 7. Sequência de construção recomendada

### Fase A — primeiros quatro serviços implementáveis
1. Identidade e Acesso (Identity & Access).
2. Perfis e Profissionais (Profiles & Professionals).
3. Catálogo e Ofertas (Catalog & Offers).
4. Agendamento e Compromissos (Scheduling & Appointments) — domínios 5 e 6 como módulos separados dentro do serviço.

Construa um fluxo *end-to-end* simplificado: cadastro/login → perfil profissional → oferta → agenda de trabalho → criação de agendamento com prevenção de conflitos. O pagamento pode ser representado por um *test double* (dublê de teste) apenas em testes locais; ele não deve ser representado como um pagamento concluído no produto final.

### Fase B — conclusão do núcleo do MVP
5. Pagamentos e Repasses (Payments & Payouts).
6. Avaliações e Reputação (Reviews & Reputation).
7. Relatórios e Moderação (Reports & Moderation).
8. Descoberta e Busca (Discovery & Search) — caso o módulo de consulta básico já não seja suficiente.

### Fase C — operações e escala
9. Analítica e Indicadores (Analytics & Indicators).
10. Fluxos de trabalho administrativos / refinamentos de auditoria.
11. Desmembre módulos — como busca ou disponibilidade — em serviços independentes apenas quando requisitos de escalabilidade, propriedade (*ownership*) ou confiabilidade justificarem o custo adicional de um sistema distribuído.

A numeração das fases refere-se à sequência de implementação, e não implica que os identificadores dos domínios lógicos tenham sido alterados.

## 8. MVC, TDD, Kafka e migração para arquitetura hexagonal

- É aceitável utilizar *controllers* no padrão MVC, mas mantenha as decisões de negócio fora deles.
- Realize testes unitários nas regras de domínio e nos serviços de aplicação antes de executar testes de integração com a infraestrutura.
- Defina interfaces de serviço explícitas para chamadas entre módulos e entre serviços; o código de domínio não deve depender de classes do Kafka ou de *controllers* web.
- Adicione testes de contrato/consumidor do Kafka para fluxos de produção e consumo de eventos.
- Antes da migração para a arquitetura hexagonal, estabeleça testes de caracterização e uma suíte de regressão que esteja passando (*green*).
- Mova dependências externas para trás de portas/adaptadores (*ports/adapters*) de forma gradual; mantenha os contratos da API pública e dos eventos estáveis ​​durante a refatoração.
- Não combine a migração arquitetural com uma reformulação abrangente das regras de negócio. Separe as alterações de refatoração das mudanças de comportamento.

## 9. Decisões ainda pendentes

- Provedor de pagamentos, métodos aceitos, momento da cobrança, comissionamento, repasses, reembolsos e política de cancelamento. - Se a verificação de identidade profissional é necessária e em qual etapa ela deve ocorrer.
- Estratégia de confirmação de agendamentos: aceitação automática ou manual (por parte do profissional).
- Fluxo de estados dos agendamentos e política para casos de não comparecimento (*no-show*).
- Precisão da localização e critérios de ranqueamento na busca/descoberta de profissionais.
- Canais de notificação para o MVP.
- Escala de avaliação, política de edição/resposta e diretrizes de moderação.
- Modelo de arquitetura: microsserviços (focados em aprendizado) versus um monólito modular de menor complexidade.

Nenhum item pendente deve ser considerado uma regra de negócio aprovada até que o *Product Owner* (responsável pelo produto) o confirme.