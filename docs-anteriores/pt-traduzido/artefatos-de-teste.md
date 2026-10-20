# Artefatos de Teste — Marketplace de Serviços de Beleza

**ID do Documento:** TEST-ART-001
**Versão:** 1.0
**Status:** Base de testes proposta para TDD

## 1. Objetivo

Este documento define camadas de teste, artefatos e cenários em nível de domínio a serem utilizados durante a implementação do marketplace. Os testes devem ser escritos antes ou simultaneamente ao desenvolvimento do comportamento, seguindo o ciclo de Desenvolvimento Orientado a Testes (TDD): **Red (Falha) → Green (Sucesso) → Refactor (Refatoração)**. Um requisito não é considerado verificado apenas pela existência de um controlador ou de uma tela.

A recomendação inicial de implementação consiste em quatro serviços implantáveis: Identidade (Identity), Perfis (Profiles), Catálogo (Catalog) e um serviço de Agendamento e Compromissos (Scheduling & Appointments), este último contendo dois módulos internos. Caso a equipe opte por implantar o "Domínio 5" separadamente, os testes de conflito de autoridade devem ser mantidos no "Domínio 6", adicionando-se testes explícitos de contrato/reserva entre eles.

## 2. Estratégia de teste

### 2.1 Camadas de teste

| Camada | Objetivo | Dependências típicas | Sugestão de nomenclatura |
|---|---|---|---|
| Teste de unidade/domínio | Verificar rapidamente invariantes de negócio e transições de estado. | Sem servidor web, broker, banco de dados ou provedor real. | `...Test` ou `...Tests`, conforme a convenção do projeto |
| Teste de aplicação/serviço | Verificar orquestração, validação, verificações de permissão e o tratamento de resultados de colaboradores. | Fakes/mocks de portas ou interfaces de serviço. | `...ServiceTest` |
| Teste de repositório/integração | Verificar persistência, restrições, transações e mapeamento. | Banco de dados de teste/container. | `...RepositoryIT` |
| Teste de API/controlador | Verificar validação de requisições, status/formato HTTP, integração com autenticação e autorização. | Framework de teste web, autenticação para testes. | `...ControllerTest` |
| Teste de contrato Kafka | Verificar esquema de eventos, cabeçalhos, compatibilidade e comportamento do consumidor. | Kafka embarcado/em container ou ferramenta de contrato, conforme necessário. | `...EventContractTest` / `...ConsumerTest` |
| Teste ponta a ponta (E2E) | Verificar a jornada completa do usuário através do conjunto de serviços definido. | Ambiente de teste, dublês de teste (test doubles) ou provedor em ambiente sandbox. | `...E2ETest` |
| Teste não funcional | Verifique as metas de carga, resiliência, segurança, acessibilidade e observabilidade. | Ferramentas de staging, desempenho e segurança. | Plano/relatório de testes separado. |

Os nomes específicos do framework de testes Java e dos pacotes são decisões de implementação. Os nomes de testes apresentados abaixo são artefatos ilustrativos, não nomes de classes obrigatórios para o ambiente de produção.

### 2.2 Ciclo TDD

1. Selecione um requisito e um cenário concreto de falha ou caso limite (*edge case*).
2. Escreva um teste que falhe e que expresse o comportamento esperado.
3. Implemente a menor alteração necessária para que o teste passe.
4. Refatore mantendo a suíte de testes passando (verde).
5. Adicione testes de integração/contrato para limites que não podem ser verificados por testes unitários.
6. Mantenha um vínculo entre o teste e o ID do requisito correspondente (por exemplo, `FR-APT-003`).

### 2.3 Dados de teste e ambiente

- Utilize contas sintéticas e identificadores de profissionais/clientes; nunca utilize dados pessoais reais de produção em suítes de testes comuns.
- Congele ou injete fontes de tempo/relógio para testes que envolvam expiração, fusos horários, cancelamentos e períodos de relatório.
- Utilize um adaptador de pagamento determinístico para a maioria dos testes; recorra aos ambientes de *sandbox* dos provedores para testes de integração com *webhooks* e ciclo de vida.
- Teste consumidores Kafka utilizando eventos duplicados, atrasados, fora de ordem, malformados ou com versões não suportadas.
- Defina explicitamente horários de funcionamento, bloqueios, ofertas e agendamentos. Evite testes que dependam do horário atual real.
- Garanta que cada teste seja capaz de limpar seus próprios dados e, sempre que possível, execute-o de forma independente.

## 3. Domínio 1 — Identidade e Acesso

### Artefatos sugeridos
- `AccountRegistrationTest`
- `AuthenticationServiceTest`
- `RecoveryChallengeTest`
- `RoleAuthorizationTest`
- `AccountRepositoryIT`
- `IdentityControllerTest`
- `AccountCreatedEventContractTest`
- `AccountEventConsumerIdempotencyTest`
### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-IAM-001 | Cadastro com campos obrigatórios válidos. | A conta é criada e um ID de conta estável é retornado. |
| T-IAM-002 | Cadastro com e-mail/identificador de login normalizado duplicado. | A solicitação é rejeitada; nenhuma segunda conta é criada. |
| T-IAM-003 | Cadastro com campos obrigatórios ausentes ou inválidos. | Erros de validação são retornados; nenhuma conta inválida é persistida. |
| T-IAM-004 | Verificação dos dados de credenciais armazenados após o cadastro de senha. | A senha em texto simples não é persistida nem registrada em logs. |
| T-IAM-005 | Autenticação com credenciais corretas para uma conta ativa. | O resultado da autenticação contém apenas declarações (claims) de identidade/função aprovadas. |
| T-IAM-006 | Autenticação com credenciais incorretas. | O acesso é negado sem revelar detalhes sensíveis da conta. |
| T-IAM-007 | Autenticação de uma conta desativada/suspensa. | O acesso é negado conforme a política estabelecida. |
| T-IAM-008 | Recuperação de acesso com um desafio (challenge) expirado, reutilizado ou inválido. | A recuperação é rejeitada e nenhuma alteração nas credenciais é realizada. |
| T-IAM-009 | Tentativa de realizar uma operação administrativa sem a permissão de ADMIN. | O servidor nega a operação. |
| T-IAM-010 | Publicação/consumo de um evento de "conta criada" duplicado. | O consumidor não cria perfis ou projeções duplicadas nos sistemas subsequentes (downstream). |
| T-IAM-011 | Evento de alteração de função (role) contendo um campo de credencial/token. | A validação de contrato rejeita o esquema do evento; dados sensíveis nunca são emitidos. |

## 4. Domínio 2 — Perfis e Profissionais

### Artefatos sugeridos
- `ClientProfileServiceTest`
- `ProfessionalProfileLifecycleTest`
- `SocialLinkValidationTest`
- `ProfileVisibilityAuthorizationTest`
- `ProfileRepositoryIT`
- `ProfessionalProfileEventContractTest`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-PROF-001 | Criar um perfil de cliente vinculado a uma conta existente. | O perfil é criado com o ID de conta correto. |
| T-PROF-002 | Tentativa de criar um perfil associado a um ID de conta inexistente ou não autorizado. | A solicitação é rejeitada. |
| T-PROF-003 | Ativar um perfil profissional com todos os campos obrigatórios preenchidos. | A transição de estado é bem-sucedida e o evento de ativação é publicado uma única vez. |
| T-PROF-004 | Ativar um perfil com a ausência de um campo obrigatório. | A ativação é rejeitada e o perfil permanece não público. |
| T-PROF-005 | Adicionar uma URL de rede social malformada ou não suportada. | O link é rejeitado ou normalizado de acordo com a política estabelecida. |
| T-PROF-006 | Usuário anônimo solicita campos privados de um cliente/perfil. | Os campos privados não são retornados. |
| T-PROF-007 | Um profissional tenta editar o perfil de outro profissional. | A operação é negada. |
| T-PROF-008 | Um perfil suspenso tenta publicar ou atualizar seu status público para "ativo". | A transição inválida é negada; a restauração só é possível por meio de um fluxo de trabalho autorizado. |
| T-PROF-009 | Reprocessar o evento de ativação de perfil. | O consumidor de projeção permanece idempotente. |
| T-PROF-010 | A conta de identidade é desativada enquanto o perfil permanece ativo. | A política de integração definida impede que um perfil inelegível continue sendo descoberto após o processamento; qualquer atraso na propagação pode ser observado. |

## 5. Domínio 3 — Catálogo e Ofertas

### Artefatos sugeridos
- `CategoryServiceTest`
- `ServiceOfferValidationTest`
- `ProfessionalOfferOwnershipTest`
- `OfferPriceSnapshotTest`
- `CatalogRepositoryIT`
- `OfferEventContractTest`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-CAT-001 | Criação de uma categoria e definição de serviço válidas. | A definição é armazenada e disponibilizada para chamadores autorizados. |
| T-CAT-002 | Desativação de uma categoria/serviço que já possui histórico de uso. | Novos usos são bloqueados, mas os registros históricos permanecem acessíveis para consulta. |
| T-CAT-003 | Criação de uma oferta com preço e duração positivos para um profissional elegível. | A oferta torna-se ativa. |
| T-CAT-004 | Criação de uma oferta com duração zero/negativa ou preço inválido. | A validação rejeita a oferta. |
| T-CAT-005 | Tentativa de um profissional editar uma oferta que pertence a outro profissional. | A operação é negada. |
| T-CAT-006 | Dois profissionais oferecem o mesmo serviço genérico com preços/durações diferentes. | Ambas as ofertas coexistem com condições comerciais independentes. |
| T-CAT-007 | Profissional define um nome mais específico do que o serviço genérico da taxonomia. | Aceito, desde que a personalização de nomes de ofertas seja permitida pelas regras estabelecidas. |
| T-CAT-008 | Desativação de uma oferta após o resultado de uma busca ter sido armazenado em cache. | Uma nova tentativa de agendamento é rejeitada pela validação oficial. |
| T-CAT-009 | Alteração de preço após a criação de um agendamento. | O agendamento existente mantém o preço/duração registrados no momento da sua criação. |
| T-CAT-010 | Processamento de um evento de publicação de oferta duplicado. | A projeção de busca não duplica a oferta. | ## 6. Domínio 4 — Descoberta e Busca

### Artefatos sugeridos
- `ProfessionalSearchServiceTest`
- `SearchFilterTest`
- `PublicProfileProjectionTest`
- `SearchControllerTest`
- `SearchEventConsumerTest`
- `SearchProjectionRebuildIT`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-DSC-001 | Busca por uma categoria/serviço com ofertas ativas correspondentes. | Apenas registros elegíveis e correspondentes são retornados. |
| T-DSC-002 | Filtragem por localização, preço, avaliação ou disponibilidade (com dados presentes). | Os resultados atendem a todos os filtros aplicados. |
| T-DSC-003 | Envio de uma faixa de preço inválida ou de um filtro não suportado. | Uma mensagem de validação é retornada; nenhum resultado incorreto ou enganoso é apresentado. |
| T-DSC-004 | A busca não encontra resultados. | Um estado de "vazio" explícito é retornado. |
| T-DSC-005 | Perfil contém dados privados ou está em estado inativo. | Campos privados ou perfis inelegíveis são omitidos. |
| T-DSC-006 | Oferta/perfil é desativado, mas a projeção de busca ainda não processou o evento. | A busca pode apresentar dados temporariamente defasados, mas o serviço de agendamento rejeita a oferta; a atualização dos dados é observável. |
| T-DSC-007 | O mesmo evento de oferta é recebido duas vezes. | O índice de busca mantém apenas um registro lógico. |
| T-DSC-008 | Eventos chegam fora de ordem. | A projeção utiliza versões/carimbos de data/hora (timestamps) dos eventos ou uma estratégia de atualização confiável para evitar a reativação de dados obsoletos. |
| T-DSC-009 | O serviço/índice de busca está indisponível. | A API falha de forma controlada (graceful failure), registrando logs/alertas; não são criados dados de disponibilidade fictícios. |
| T-DSC-010 | Reconstrução da projeção a partir de um conjunto de dados (fixture) ou histórico de eventos controlado. | Os resultados reconstruídos correspondem ao estado esperado da fonte de dados. |

## 7. Domínio 5 — Agenda e Disponibilidade

### Artefatos sugeridos
- `WorkingHoursValidationTest`
- `AvailabilityCalculationTest`
- `TimeBlockServiceTest`
- `TimeZoneAndBoundaryTest`
- `ScheduleRepositoryIT`
- `AvailabilityEventContractTest`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-AVL-001 | Configurar uma agenda recorrente válida e um fuso horário. | A agenda é salva e interpretada conforme o fuso horário definido. |
| T-AVL-002 | Configurar um horário de término anterior ou igual ao horário de início, sem uma regra que permita a extensão pelo período noturno (virada de dia). | A agenda é rejeitada. |
| T-AVL-003 | Definir intervalos/blocos de trabalho incompatíveis que se sobrepõem. | A validação os rejeita ou normaliza, de acordo com uma política explícita. |
| T-AVL-004 | Consultar horários disponíveis para uma duração de serviço que se encaixa exatamente no limite da agenda. | O horário só aparece se a duração total estiver contida em um intervalo válido. |
| T-AVL-005 | Realizar uma consulta com uma duração maior do que qualquer intervalo livre existente. | Nenhum horário candidato é retornado. |
| T-AVL-006 | Adicionar um bloco que se sobrepõe a horários candidatos. | Os horários candidatos afetados deixam de existir. |
| T-AVL-007 | Tentar adicionar um bloco que se sobrepõe a um agendamento já confirmado. | A operação é rejeitada ou tratada por meio de um fluxo de trabalho aprovado explicitamente; não ocorre a exclusão silenciosa de agendamentos. |
| T-AVL-008 | Realizar uma consulta que abrange uma transição de horário de verão ou uma mudança de fuso horário, quando aplicável. | Os intervalos permanecem consistentes com as regras do fuso horário selecionado. |
| T-AVL-009 | Alterar a agenda enquanto um cliente possui uma lista antiga de horários candidatos. | A validação final do agendamento verifica novamente a disponibilidade e rejeita o horário caso ele não esteja mais disponível. |
| T-AVL-010 | Reprocessar um evento de atualização de agenda. | Os sistemas consumidores não duplicam blocos ou registros de agenda. | ## 8. Domínio 6 — Agendamentos e Atendimentos

### Artefatos sugeridos
- `AppointmentCreationTest`
- `AppointmentStateMachineTest`
- `AppointmentCancellationPolicyTest`
- `AppointmentConflictIntegrationTest`
- `AppointmentRepositoryIT`
- `AppointmentEventContractTest`
- `ConcurrentBookingE2ETest`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-APT-001 | Criar agendamento para um cliente válido, com perfil/oferta ativa e horário disponível. | O agendamento armazena as referências corretas, intervalo, estado e o registro das condições comerciais acordadas. |
| T-APT-002 | Criar agendamento para um perfil inativo ou oferta desativada. | O agendamento é rejeitado. |
| T-APT-003 | Criar agendamento quando o horário não está mais disponível. | É retornada uma resposta de conflito e nenhum agendamento confirmado é armazenado. |
| T-APT-004 | Duas solicitações simultâneas tentam agendar intervalos sobrepostos para o mesmo profissional. | No máximo um dos agendamentos conflitantes é efetivado como confirmado. |
| T-APT-005 | Dois agendamentos para o mesmo profissional possuem intervalos que não se sobrepõem. | Ambos podem ser concluídos com sucesso, caso as demais regras permitam. |
| T-APT-006 | A duração do serviço sobrepõe-se a um agendamento vizinho, mesmo que os horários de início sejam diferentes. | O agendamento conflitante é rejeitado. |
| T-APT-007 | Cliente tenta visualizar/cancelar o agendamento de outro cliente. | O acesso é negado. |
| T-APT-008 | Executar uma transição de estado inválida (por exemplo, de "cancelado" para "concluído", sem um fluxo permitido). | A transição é rejeitada e o estado permanece inalterado. |
| T-APT-009 | Cancelar um agendamento conforme a política permitida. | O cancelamento é registrado; o fluxo de pagamento subsequente é acionado separadamente, se aplicável. |
| T-APT-010 | Reembolso ainda pendente após o cancelamento do agendamento. | O agendamento consta como cancelado; o pagamento consta como aguardando reembolso. Não ocorre mesclagem indevida de estados. |
| T-APT-011 | Marcar um agendamento elegível como concluído. | O evento de conclusão é publicado, e a elegibilidade para avaliação pode ser determinada. |
| T-APT-012 | Registrar o não comparecimento (no-show) em um agendamento não elegível ou aplicar penalidade antes da aprovação da política. | A operação é rejeitada ou permanece desabilitada até que a política seja aprovada. |
| T-APT-013 | O mesmo comando de criação é reenviado utilizando uma chave de idempotência. | Apenas um agendamento lógico é criado; a nova tentativa retorna o resultado já existente. |
| T-APT-014 | A persistência no banco de dados é bem-sucedida, mas a publicação do evento falha. | Uma estratégia de *outbox*/reententativa (*retry*) acaba publicando o evento, sem duplicar o agendamento. |

**Invariante de prioridade máxima:** O caso `T-APT-004` deve ser um teste real de concorrência/integração, e não apenas um teste unitário com *mocks*.

## 9. Domínio 7 — Pagamentos e Repasses

### Artefatos sugeridos
- `PaymentInitiationServiceTest`
- `ProviderWebhookVerificationTest`
- `PaymentStateMachineTest`
- `RefundPolicyServiceTest`
- `PaymentRepositoryIT`
- `PaymentEventContractTest`
- `PaymentProviderSandboxIT`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-PAY-001 | Iniciar pagamento para um checkout elegível com valor válido. | Uma transação interna é criada e a solicitação ao provedor é correlacionada. |
| T-PAY-002 | Tentar novamente uma solicitação de início de pagamento usando a mesma chave de idempotência. | Nenhuma cobrança duplicada é criada. |
| T-PAY-003 | O provedor rejeita o pagamento. | O status do pagamento é definido como rejeitado/falho, conforme o mapeamento; nenhum sucesso indevido é indicado. |
| T-PAY-004 | A chamada ao provedor atinge o tempo limite (timeout) com resultado desconhecido. | O status permanece pendente/desconhecido até que seja reconciliado; o serviço não assume sucesso ou falha prematuramente. |
| T-PAY-005 | Recebimento de um callback válido do provedor. | O callback é autenticado e o status do pagamento é atualizado. |
| T-PAY-006 | Recebimento de um callback com assinatura inválida ou divergência no valor/moeda. | O callback é rejeitado e o status financeiro permanece inalterado. |
| T-PAY-007 | Recebimento do mesmo webhook duas vezes. | Apenas uma transição financeira lógica é aplicada. |
| T-PAY-008 | Recebimento de callbacks fora de ordem. | A máquina de estados impede regressões inválidas e segue a política de reconciliação do provedor. |
| T-PAY-009 | Cancelamento de um agendamento com pagamento aprovado, sujeito a reembolso. | A operação de reembolso é iniciada e rastreada de forma independente. |
| T-PAY-010 | O provedor confirma o reembolso apenas posteriormente. | O reembolso permanece pendente até que haja uma confirmação confiável. |
| T-PAY-011 | Consulta a registro financeiro por um usuário/profissional não autorizado. | Informações financeiras protegidas não são expostas. |
| T-PAY-012 | Verificação da persistência da transação. | As credenciais completas do cartão não são armazenadas; são armazenados a referência do provedor e os campos necessários para reconciliação. |

## 10. Domínio 8 — Avaliações e Reputação

### Artefatos sugeridos
- `ReviewEligibilityTest`
- `ReviewSubmissionServiceTest`
- `RatingAggregationTest`
- `ReviewRepositoryIT`
- `ReviewEventContractTest`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-REV-001 | Cliente avalia um agendamento concluído elegível. | A avaliação é vinculada ao agendamento, cliente e profissional corretos. |
| T-REV-002 | Cliente tenta avaliar um agendamento pertencente a outra conta. | A avaliação é rejeitada. |
| T-REV-003 | Cliente tenta avaliar um agendamento pendente/cancelado/não comparecido (no-show) quando não elegível. | A avaliação é rejeitada. |
| T-REV-004 | Cliente envia uma nota (rating) fora da faixa permitida. | A validação rejeita a nota. |
| T-REV-005 | Cliente envia uma avaliação duplicada para o mesmo agendamento. | A duplicata é rejeitada, caso a regra definida seja de apenas uma avaliação por agendamento. |
| T-REV-006 | Avaliação é ocultada por meio de uma ação de moderação válida. | A avaliação ocultada deixa de compor a média pública, conforme a fórmula estabelecida. |
| T-REV-007 | Recálculo da média após a adição/remoção de uma avaliação visível. | A contagem e a média correspondem a um valor de referência (fixture) e à política de arredondamento definida. |
| T-REV-008 | Processamento de um evento duplicado de "avaliação publicada". | A projeção para busca/análise contabiliza o evento apenas uma vez. |

## 11. Domínio 9 — Denúncias e Moderação

### Artefatos sugeridos
- `ReportCreationTest`
- `ModerationCaseStateMachineTest`
- `ModeratorAuthorizationTest`
- `ModerationDecisionAuditTest`
- `ModerationEventContractTest`
### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-MOD-001 | Enviar uma denúncia com um tipo de alvo e motivo válidos. | A denúncia recebe um ID de referência e um estado inicial. |
| T-MOD-002 | Enviar uma denúncia referente a um alvo inexistente ou não suportado. | A solicitação é rejeitada ou registrada como um alvo explicitamente não verificado, conforme a política. |
| T-MOD-003 | Usuário não autenticado ou que não seja moderador tenta abrir um caso de moderação. | O acesso é negado. |
| T-MOD-004 | Moderador registra uma decisão com o motivo obrigatório. | A decisão, o moderador, o registro de data/hora e a solicitação de ação resultante são registrados. |
| T-MOD-005 | Moderador tenta realizar uma transição de caso inválida. | A operação é rejeitada. |
| T-MOD-006 | Domínio proprietário rejeita uma solicitação de suspensão ou ocultação. | O caso registra a ação rejeitada/falha; o registro de origem não é exibido incorretamente como modificado. |
| T-MOD-007 | O mesmo evento de moderação é processado duas vezes. | Nenhum comando de suspensão ou ocultação é aplicado indevidamente mais de uma vez. |
| T-MOD-008 | Inspecionar logs/eventos em busca de dados pessoais ou credenciais desnecessários. | Não há presença de dados sensíveis. |

## 12. Domínio 10 — Análises e Indicadores

### Artefatos sugeridos
- `ProfessionalMetricsCalculationTest`
- `PlatformMetricsCalculationTest`
- `MetricsProjectionIdempotencyTest`
- `MetricsAuthorizationTest`
- `MetricsProjectionRebuildIT`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-ANL-001 | Agregar agendamentos para um período e evento/compromisso específicos. | A contagem corresponde ao tratamento documentado de estados/fuso horário. |
| T-ANL-002 | Agregar agendamentos concluídos, cancelados e de não comparecimento (*no-show*). | ​​Cada um é contabilizado de acordo com as definições de métricas aprovadas; as categorias não são confundidas ou misturadas. |
| T-ANL-003 | Calcular a quantidade e a média de avaliações. | O resultado está em conformidade com as regras de "avaliações públicas elegíveis" do domínio de Avaliações. |
| T-ANL-004 | Calcular o volume de pagamentos considerando reembolsos/cancelamentos. | As classificações de valores brutos/líquidos e o tratamento de reembolsos seguem as definições documentadas. |
| T-ANL-005 | Um profissional consulta os indicadores de outro profissional. | O acesso é negado. |
| T-ANL-006 | Processar o mesmo evento de agendamento/pagamento duas vezes. | Os agregados não são contabilizados em duplicidade. |
| T-ANL-007 | O consumidor de projeções apresenta atraso. | O atraso é observável e a defasagem ( freshness) é indicada onde necessário. |
| T-ANL-008 | Reconstruir a projeção a partir de um registro/histórico de eventos. | Os resultados reconstruídos correspondem aos agregados esperados. |

## 13. Domínio 11 — Administração de Plataforma e Operações de Auditoria

### Artefatos sugeridos
- `AdminPermissionPolicyTest`
- `AdminOwnerServiceDelegationTest`
- `AdministrativeAuditRecordTest`
- `AdminControllerTest`
- `AdminWorkflowE2ETest`

### Cenários

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-ADM-001 | Administrador autorizado invoca uma operação permitida em um domínio de proprietário (*owner domain*). | ​​O serviço do proprietário aplica sua própria validação e o resultado é exibido. |
| T-ADM-002 | Usuário não administrador tenta acessar a área administrativa. | O servidor rejeita o acesso. |
| T-ADM-003 | A interface administrativa tenta modificar o banco de dados diretamente ou contornar a API do proprietário. | A revisão de arquitetura/código e o teste de integração demonstram que tal caminho não existe. |
| T-ADM-004 | Um comando administrativo é enviado duas vezes devido a uma tentativa de reexecução (*retry*). | ​​A idempotência ou o tratamento de conflitos evita uma ação duplicada não intencional. |
| T-ADM-005 | Uma ação administrativa importante é concluída com sucesso. | O registro de auditoria inclui o autor, a ação, o alvo, o momento, o resultado e o motivo/ID de correlação exigido. |
| T-ADM-006 | Usuário não autorizado tenta acessar registros de auditoria. | O acesso é negado. |
| T-ADM-007 | Administrador investiga um agendamento cancelado com reembolso pendente. | A interface exibe separadamente os estados do agendamento e do pagamento/reembolso. |
| T-ADM-008 | O serviço do proprietário está temporariamente indisponível. | A interface administrativa informa a falha ou um status desconhecido, sem indicar sucesso. |

## 14. Testes entre domínios, Kafka e consistência

| ID | Cenário | Resultado esperado |
|---|---|---|
| T-INT-001 | O produtor emite um evento após uma alteração de domínio ser confirmada (*commit*). | ​​O evento torna-se disponível eventualmente; nenhum evento descreve um estado que foi revertido (*rollback*). |
| T-INT-002 | A confirmação (*commit*) no banco de dados ocorre com sucesso enquanto o *broker* está indisponível. | O mecanismo de *outbox* ou de reexecução (*retry*) publica o evento posteriormente. |
| T-INT-003 | O consumidor recebe um evento duplicado. | O processamento é idempotente. |
| T-INT-004 | O consumidor recebe um evento malformado ou com uma versão não suportada. | O evento é rejeitado ou colocado em quarentena de forma segura; a falha é observável. |
| T-INT-005 | Uma falha no consumidor resulta em tentativas de reprocessamento repetidas. | A política de retentativa é seguida; a mensagem problemática (*poison message*) é encaminhada para uma fila de mensagens mortas (*dead-letter queue*) configurada, evitando um loop infinito silencioso. |
| T-INT-006 | Rastreamento de um agendamento através dos consumidores de Catálogo, Disponibilidade/Agendamentos, Pagamento e Notificações. | IDs de correlação (*Correlation IDs*) permitem uma investigação de ponta a ponta. |
| T-INT-007 | A projeção de busca contém dados de horários defasados ​​(*stale data*). | ​​O fluxo de gravação de agendamentos ainda impede a criação de agendamentos conflitantes. |
| T-INT-008 | Um perfil ou oferta é desativado enquanto o cliente está finalizando o processo de contratação (*checkout*). | ​​A validação final do agendamento baseia-se no estado atual considerado como fonte da verdade (*authoritative state*). |
| T-INT-009 | O cancelamento de um agendamento e o processamento do reembolso são concluídos em momentos diferentes. | Cada estado é representado com precisão e reconciliado. |
| T-INT-010 | O evento contém uma senha, token ou credencial de cartão. | O teste de esquema ou de segurança falha, e o evento não é publicado. |

## 15. Cenários de aceitação ponta a ponta (E2E)

### E2E-001 — Publicação de profissional e oferta
1. Registrar uma conta de profissional.
2. Criar e ativar um perfil profissional.
3. Criar uma oferta de serviço com preço e duração.
4. Configurar horários de atendimento válidos.
5. Consultar a disponibilidade do profissional.
6. Verificar se o perfil/oferta pode ser encontrado por meio do fluxo de busca implementado.

**Resultado esperado:** cada etapa só é concluída com sucesso se os pré-requisitos forem válidos; os eventos emitidos atualizam as projeções sem criar registros duplicados.

### E2E-002 — Criação e conclusão de agendamento
1. Registrar/autenticar um cliente.
2. Localizar um profissional ativo e uma oferta.
3. Selecionar um horário disponível.
4. Criar um agendamento.
5. Tentar realizar um agendamento concorrente para um intervalo sobreposto.
6. Concluir o atendimento por meio de uma ação autorizada do profissional.
7. Enviar uma avaliação por parte do cliente.

**Resultado esperado:** no máximo um agendamento conflitante; transições válidas; avaliação permitida apenas após a conclusão elegível.

### E2E-003 — Falha no pagamento e nova tentativa
1. Iniciar o pagamento no ambiente de teste do provedor.
2. Simular uma rejeição ou *timeout* (tempo limite excedido).
3. Reexecutar o *callback*/evento.
4. Conciliar o estado final do pagamento com o estado do agendamento.

**Resultado esperado:** ausência de falsos sucessos, cobranças duplicadas ou confusão implícita entre os estados de agendamento e pagamento.

### E2E-004 — Cancelamento e reembolso
1. Criar um agendamento passível de cobrança.
2. Cancelar o agendamento conforme uma política de teste configurada.
3. Simular um atraso na confirmação do reembolso.
4. Consultar o agendamento e o pagamento separadamente.

**Resultado esperado:** um agendamento cancelado pode coexistir com um reembolso pendente; o estado final do reembolso só é alterado mediante uma confirmação confiável.

## 16. Artefatos de testes não funcionais

- **Segurança:** matriz de acesso baseada em funções (*role-based access*); testes de propriedade/IDOR (*Insecure Direct Object Reference*); revisão do tratamento de credenciais; varredura de dependências e segredos; testes de assinatura de *webhooks*.
- **Concorrência:** criação paralela de agendamentos para intervalos sobrepostos; bloqueio de agenda e atualização de agendamento simultâneos; inicialização de pagamento idempotente. - **Desempenho:** mensuração de requisições comuns (busca, perfil, disponibilidade) sob uma carga documentada; a proposta inicial é p95 ≤ 3 segundos, excluindo a latência de terceiros, sujeita a confirmação.
- **Acessibilidade/responsividade:** verificações manuais e automatizadas nos fluxos de cadastro, busca, perfil, reserva, pagamento e administração.
- **Resiliência:** cenários como falha no *broker*, consumidor lento, eventos duplicados ou fora de ordem, *timeout* de provedor de pagamento e indisponibilidade de um serviço de proprietário.
- **Observabilidade:** verificação de *logs* e métricas acionáveis, bem como de *IDs* de correlação, garantindo a exclusão de segredos e dados pessoais desnecessários.
- **Recuperação:** verificação dos procedimentos de *backup* e restauração de banco de dados, além da reconstrução de projeções, antes da entrada em produção.

## 17. Rede de segurança para migração de MVC para arquitetura hexagonal

Antes de refatorar cada serviço:

1. Congele os contratos de APIs externas e do Kafka durante a janela de alteração.
2. Garanta que as regras de domínio e de aplicação estejam cobertas por testes unitários ou de serviço.
3. Adicione testes de caracterização para comportamentos ainda não documentados.
4. Adicione testes de contrato para persistência, APIs e eventos (nos adaptadores).
5. Isole as dependências de infraestrutura/framework atrás de interfaces/adaptadores, realizando a mudança por meio de pequenos commits.
6. Execute todos os testes — unitários, de integração, de contrato e de ponta a ponta (E2E) — antes e depois de cada etapa.
7. Compare o comportamento, não a estrutura interna das classes; não reescreva asserções apenas para fazer com que a refatoração passe nos testes.
8. Realize o deploy de forma gradual, se o ambiente operacional permitir; monitore erros, atrasos no processamento de eventos, conflitos em reservas e a reconciliação de pagamentos.

## 18. Definição de "Concluído" (Definition of Done) para um requisito

- O ID do requisito está referenciado em um ou mais testes.
- O fluxo principal (*happy path*), entradas inválidas, autorização e casos relevantes de limites ou falhas estão cobertos.
- Regras de domínio críticas são testadas, sempre que possível, sem depender de infraestrutura.
- Contratos de persistência, APIs e Kafka são verificados, quando aplicável.
- Não há segredos ou dados pessoais desnecessários expostos em logs ou eventos.
- A documentação e os registros de decisões estão atualizados.
- O pipeline de CI é concluído com sucesso e o comportamento é demonstrado no ambiente de testes acordado.