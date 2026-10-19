# Especificação de Requisitos — Marketplace de Serviços de Beleza

**ID do Documento:** REQ-SPEC-001
**Versão:** 1.0
**Status:** Base de referência proposta — decisões de negócio ainda em aberto
**Escopo:** Primeira versão (MVP), organizada em 11 domínios lógicos

## 1. Propósito e terminologia

Este documento especifica requisitos de negócio, requisitos funcionais, requisitos de qualidade e contratos entre domínios para o marketplace. Ele descreve o comportamento esperado, sem garantir que cada domínio já esteja implementado como um microsserviço com implantação independente.

**Atores:** Cliente, Profissional, Administrador, Provedor de Identidade externo (opcional), Provedor de Pagamento e integrações de sistema.
**Prioridades:** `MUST` (obrigatório) = necessário para o escopo definido do MVP; `SHOULD` (deveria) = importante, mas passível de negociação; `COULD` (poderia) = desejável; `PENDING` (pendente) = requer uma decisão de negócio antes da implementação.
**IDs de Requisitos:** `BR` = requisito de negócio, `FR` = requisito funcional, `NFR` = requisito não funcional, `INT` = requisito de integração/evento.

As definições oficiais dos limites de domínio e as regras de propriedade estão em `project_domains.md`. Os fluxos de casos de uso estão em `domain_use_cases.md`; os cenários de teste estão em `test_artifacts.md`.

## 2. Objetivo do produto e jornada do MVP

O produto deve conectar clientes a profissionais de beleza e cuidados pessoais, permitindo que o cliente encontre uma oferta, verifique seu preço e duração, selecione um horário disponível, realize o agendamento, efetue o pagamento conforme o fluxo aprovado, receba o serviço e forneça uma avaliação. Os profissionais devem ser capazes de apresentar seus perfis e ofertas, manter suas agendas e gerenciar agendamentos. Os administradores devem poder tratar questões operacionais sem contornar o domínio responsável pelo registro afetado.

Jornada principal:

1. Criação de conta/autenticação.
2. Publicação do perfil do profissional e da oferta de serviço.
3. O cliente busca ou navega pelos profissionais/ofertas disponíveis.
4. O cliente analisa os detalhes da oferta e o horário pretendido.
5. O cliente realiza um agendamento; o sistema executa uma verificação oficial de conflitos. 6. O pagamento é processado de acordo com a política de pagamentos aprovada.
7. O agendamento é concluído ou transita para um estado final permitido.
8. O cliente pode enviar uma avaliação, caso seja elegível.

## 3. Requisitos de negócio

| ID | Requisito | Prioridade |
|---|---|---|
| BR-001 | O usuário pode se cadastrar e acessar a plataforma conforme o seu perfil (papel) atribuído. | OBRIGATÓRIO |
| BR-002 | Os clientes podem manter as informações de perfil necessárias para o uso do marketplace. | OBRIGATÓRIO |
| BR-003 | Os profissionais podem criar um perfil público e tornar-se visíveis após atenderem aos requisitos de publicação. | OBRIGATÓRIO |
| BR-004 | A plataforma disponibiliza categorias e definições genéricas de serviços, permitindo, ao mesmo tempo, ofertas, preços e durações específicos de cada profissional. | OBRIGATÓRIO |
| BR-005 | Os clientes podem pesquisar e comparar profissionais ativos e ofertas utilizando os filtros disponíveis. | OBRIGATÓRIO; uma busca simples pode preceder um serviço de busca dedicado |
| BR-006 | Os profissionais podem configurar horários de trabalho, exceções e períodos de indisponibilidade. | OBRIGATÓRIO |
| BR-007 | Os clientes podem criar, consultar e cancelar agendamentos de acordo com as regras definidas. | OBRIGATÓRIO |
| BR-008 | Agendamentos conflitantes para o mesmo profissional e mesmo intervalo de tempo não podem ser confirmados simultaneamente. | OBRIGATÓRIO |
| BR-009 | A primeira versão inclui funcionalidade de pagamento online, utilizando um provedor selecionado e uma política financeira aprovada. | OBRIGATÓRIO; fluxo detalhado PENDENTE |
| BR-010 | Clientes elegíveis podem avaliar agendamentos concluídos/elegíveis. | OBRIGATÓRIO |
| BR-011 | Usuários podem relatar problemas, e moderadores autorizados podem analisar esses relatos. | RECOMENDÁVEL |
| BR-012 | Profissionais podem visualizar indicadores operacionais básicos derivados dos registros da plataforma. | RECOMENDÁVEL; projeções básicas podem ser implementadas em etapas |
| BR-013 | Administradores podem realizar operações permitidas, conforme as regras dos domínios sob sua responsabilidade. | OBRIGATÓRIO |
| BR-014 | Dados pessoais, registros financeiros e ações administrativas são protegidos e rastreáveis. | OBRIGATÓRIO |

## 4. Requisitos funcionais por domínio

### 4.1 Domínio 1 — Identidade e Acesso

| ID | Requisito | Prioridade |
|---|---|---|
| FR-IAM-001 | Registrar contas com os campos obrigatórios e identificadores de login normalizados. | OBRIGATÓRIO |
| FR-IAM-002 | Autenticar um usuário com credenciais válidas e retornar o resultado da sessão/token configurado. | OBRIGATÓRIO |
| FR-IAM-003 | Proporcionar recuperação de conta de forma segura. | OBRIGATÓRIO |
| FR-IAM-004 | Suportar autenticação via redes sociais apenas se o provedor e o fluxo forem aprovados. | RECOMENDADO/PENDENTE |
| FR-IAM-005 | Manter os papéis (roles) de Cliente, Profissional e Administrador, e avaliar as permissões no lado do servidor (server-side). | OBRIGATÓRIO |
| FR-IAM-006 | Habilitar/desabilitar uma conta conforme a política autorizada, revogando ou expirando o acesso, conforme apropriado. | OBRIGATÓRIO |
| FR-IAM-007 | Publicar eventos básicos do ciclo de vida da conta, sem incluir credenciais ou tokens. | OBRIGATÓRIO (para integração de eventos) |

### 4.2 Domínio 2 — Perfis e Profissionais

| ID | Requisito | Prioridade |
|---|---|---|
| FR-PROF-001 | Criar e atualizar o perfil de cliente vinculado a um ID de conta. | OBRIGATÓRIO |
| FR-PROF-002 | Criar e atualizar um perfil profissional vinculado a um ID de conta. | OBRIGATÓRIO |
| FR-PROF-003 | Manter os estados do perfil (por exemplo: rascunho, ativo, suspenso) com transições válidas. | OBRIGATÓRIO |
| FR-PROF-004 | Validar os campos mínimos necessários antes que um perfil possa ser publicado. | OBRIGATÓRIO; lista de campos a ser aprovada |
| FR-PROF-005 | Armazenar links de redes sociais externos associados ao perfil profissional. | RECOMENDADO |
| FR-PROF-006 | Apresentar campos públicos separadamente dos campos privados e restringir o acesso aos campos privados. | OBRIGATÓRIO |
| FR-PROF-007 | Publicar alterações no perfil necessárias para mecanismos de busca e projeções de dados (downstream projections). | OBRIGATÓRIO |
| FR-PROF-008 | Não exigir verificação de identidade profissional, a menos que o proprietário do produto aprove uma política de verificação. | Decisão PENDENTE; o comportamento deve permanecer configurável |

### 4.3 Domínio 3 — Catálogo de Serviços e Ofertas

| ID | Requisito | Prioridade |
|---|---|---|
| FR-CAT-001 | Manter categorias da plataforma e definições genéricas de serviços. | OBRIGATÓRIO |
| FR-CAT-002 | Permitir que um profissional elegível crie, edite, liste, ative e desative suas ofertas. | OBRIGATÓRIO |
| FR-CAT-003 | Registrar, para cada oferta: ID do profissional, associação a categoria/serviço, nome, preço, duração estimada e status (ativo/inativo). | OBRIGATÓRIO |
| FR-CAT-004 | Permitir que diferentes profissionais definam preços e durações distintos para serviços semelhantes. | OBRIGATÓRIO |
| FR-CAT-005 | Permitir que as ofertas dos profissionais utilizem um nível de detalhamento mais específico do que a taxonomia genérica de serviços, quando apropriado. | OBRIGATÓRIO |
| FR-CAT-006 | Preservar registros (snapshots) do preço/duração acordados para agendamentos existentes, caso a oferta seja alterada posteriormente. | OBRIGATÓRIO |
| FR-CAT-007 | Impedir que ofertas inativas sejam selecionadas para novos agendamentos. | OBRIGATÓRIO |
| FR-CAT-008 | Publicar eventos do ciclo de vida do catálogo/oferta para projeções dependentes. | OBRIGATÓRIO (para integração de eventos) |

### 4.4 Domínio 4 — Descoberta e Busca

| ID | Requisito | Prioridade |
|---|---|---|
| FR-DSC-001 | Busca por categoria, serviço ou nome do profissional. | OBRIGATÓRIO |
| FR-DSC-002 | Filtro por área de atendimento/localização, preço, avaliação e disponibilidade (quando houver dados confiáveis). | OBRIGATÓRIO (com comportamento claro para casos em que o filtro não pode ser aplicado) |
| FR-DSC-003 | Exibição de resumos de perfis públicos ativos e ofertas, sem expor campos privados. | OBRIGATÓRIO |
| FR-DSC-004 | Tratamento adequado de resultados vazios, filtros inválidos e dados de projeção defasados ​​ou indisponíveis. | OBRIGATÓRIO |
| FR-DSC-005 | Aplicação de uma ordenação padrão documentada; o posicionamento pago não faz parte do MVP, a menos que aprovado. | OBRIGATÓRIO / PENDENTE (detalhes de ranqueamento) |
| FR-DSC-006 | A disponibilidade na busca deve ser considerada apenas indicativa; a confirmação do agendamento deve ser revalidada junto ao responsável pelo serviço. | OBRIGATÓRIO |
| FR-DSC-007 | Atualização de modelos de leitura (read models) a partir de eventos versionados e exposição da atualidade (frescor) dos dados projetados para fins operacionais. | RECOMENDÁVEL |

### 4.5 Domínio 5 — Agenda e Disponibilidade

| ID | Requisito | Prioridade |
|---|---|---|
| FR-AVL-001 | Configuração de horários de trabalho recorrentes com um fuso horário definido. | OBRIGATÓRIO |
| FR-AVL-002 | Criação, edição e remoção de exceções para datas específicas e bloqueios manuais de horários. | OBRIGATÓRIO |
| FR-AVL-003 | Cálculo de intervalos disponíveis com base nos horários configurados, bloqueios e na duração da oferta selecionada. | OBRIGATÓRIO |
| FR-AVL-004 | Retorno da disponibilidade para um profissional, oferta e intervalo de datas. | OBRIGATÓRIO |
| FR-AVL-005 | Rejeição de intervalos de agenda inválidos e bloqueios incompatíveis que se sobreponham, conforme as regras definidas. | OBRIGATÓRIO |
| FR-AVL-006 | Emissão de avisos ou rejeição de alterações na agenda que entrem em conflito com agendamentos existentes; não invalidar agendamentos de forma silenciosa. | OBRIGATÓRIO |
| FR-AVL-007 | Publicar a agenda e bloquear alterações para os consumidores relevantes. | DEVERIA |
| FR-AVL-008 | Não tratar uma resposta de disponibilidade como uma reserva definitiva (atômica). | DEVE |

### 4.6 Domínio 6 — Agendamentos e Comparecimento

| ID | Requisito | Prioridade |
|---|---|---|
| FR-APT-001 | Criar um agendamento vinculado a um cliente, profissional, oferta, intervalo de tempo e às condições comerciais acordadas. | DEVE |
| FR-APT-002 | Validar a elegibilidade da oferta e a disponibilidade atual antes de confirmar um agendamento. | DEVE |
| FR-APT-003 | Impedir que solicitações simultâneas confirmem intervalos incompatíveis para um mesmo profissional. | DEVE |
| FR-APT-004 | Manter uma máquina de estados documentada para as situações: pendente, confirmado, cancelado, concluído e ausência (no-show), conforme aprovado. | DEVE; estados exatos: PENDENTE |
| FR-APT-005 | Garantir que apenas transições de estado válidas sejam permitidas e registrar alterações importantes. | DEVE |
| FR-APT-006 | Permitir o cancelamento pelo cliente, seguindo a política aprovada, e registrar o motivo/efeitos, quando aplicável. | DEVE |
| FR-APT-007 | Permitir que um profissional autorizado registre o comparecimento (concluído) ou a ausência (no-show), conforme o fluxo de trabalho aprovado. | DEVERIA; política de ausência (no-show) pendente |
| FR-APT-008 | Publicar eventos do ciclo de vida do agendamento para fins de pagamento, avaliações, notificações e análise de dados (analytics). | DEVE |
| FR-APT-009 | Manter o estado do agendamento distinto do estado do pagamento. | DEVE |

### 4.7 Domínio 7 — Pagamentos e Repasses

| ID | Requisito | Prioridade |
|---|---|---|
| FR-PAY-001 | Iniciar o pagamento online por meio de um provedor aprovado para o agendamento correspondente. | OBRIGATÓRIO |
| FR-PAY-002 | Armazenar a referência da transação do provedor, valor, moeda, status e registros de data/hora relevantes. | OBRIGATÓRIO |
| FR-PAY-003 | Processar webhooks/callbacks verificados de forma idempotente. | OBRIGATÓRIO |
| FR-PAY-004 | Distinguir o status do pagamento do status do agendamento. | OBRIGATÓRIO |
| FR-PAY-005 | Implementar reembolsos, comissões e repasses de acordo com uma política aprovada e as funcionalidades do provedor. | OBRIGATÓRIO (escopo/decisões de negócio pendentes) |
| FR-PAY-006 | Disponibilizar visualizações autorizadas dos registros de transações e repasses para fins de suporte/conciliação. | OBRIGATÓRIO |
| FR-PAY-007 | Nunca indicar um pagamento como aprovado sem um resultado confirmado pelo provedor. | OBRIGATÓRIO |
| FR-PAY-008 | Evitar o armazenamento de dados completos de cartões de pagamento, mantendo-os sob responsabilidade do provedor. | OBRIGATÓRIO |

### 4.8 Domínio 8 — Avaliações e Reputação

| ID | Requisito | Prioridade |
|---|---|---|
| FR-REV-001 | Aceitar uma avaliação apenas de um cliente associado a um agendamento elegível. | OBRIGATÓRIO |
| FR-REV-002 | Armazenar a nota, comentário (opcional), autor da avaliação, profissional, agendamento e registros de data/hora. | OBRIGATÓRIO |
| FR-REV-003 | Aplicar a escala de avaliação aprovada e a política de "uma avaliação por agendamento", caso esta seja adotada. | OBRIGATÓRIO (escala/política a confirmar) |
| FR-REV-004 | Calcular e exibir um resumo da reputação com base nas avaliações elegíveis e visíveis. | RECOMENDÁVEL |
| FR-REV-005 | Aplicar alterações autorizadas de visibilidade/moderação, mantendo o histórico de auditoria relevante. | OBRIGATÓRIO |
| FR-REV-006 | Publicar eventos do ciclo de vida das avaliações para fins de busca e análises (analytics). | RECOMENDÁVEL | ### 4.9 Domínio 9 — Denúncias e Moderação

| ID | Requisito | Prioridade |
|---|---|---|
| FR-MOD-001 | Criar denúncias para tipos de alvo definidos, como usuário, perfil, avaliação ou agendamento. | DEVERIA |
| FR-MOD-002 | Acompanhar cada denúncia/caso por meio de status documentados e armazenar seu histórico. | DEVERIA |
| FR-MOD-003 | Restringir o acesso aos detalhes de casos e às ações de moderação apenas à equipe autorizada. | DEVE |
| FR-MOD-004 | Registrar a decisão, o motivo, o moderador, o registro de data/hora e qualquer solicitação de ação decorrente. | DEVE |
| FR-MOD-005 | Solicitar ações sobre perfis/contas/avaliações ao domínio responsável por eles, em vez de modificar seus dados diretamente. | DEVE |
| FR-MOD-006 | Notificar as partes operacionais pertinentes, conforme uma política de moderação publicada. | PENDENTE (detalhes sobre canais/política) |

### 4.10 Domínio 10 — Análises e Indicadores

| ID | Requisito | Prioridade |
|---|---|---|
| FR-ANL-001 | Calcular contagens básicas e status de agendamentos para períodos selecionados. | DEVERIA |
| FR-ANL-002 | Calcular cancelamentos e agendamentos concluídos utilizando definições documentadas. | DEVERIA |
| FR-ANL-003 | Exibir a contagem de avaliações elegíveis e a nota média. | DEVERIA |
| FR-ANL-004 | Exibir indicadores de horários ocupados e disponíveis, sempre que os dados de origem permitirem um cálculo confiável. | DEVERIA |
| FR-ANL-005 | Apresentar indicadores relevantes de pagamento/volume, sem expor detalhes financeiros a pessoas não autorizadas. | DEVERIA |
| FR-ANL-006 | Exibir o período de medição, o fuso horário e a data da última atualização das projeções, quando pertinente. | DEVERIA |
| FR-ANL-007 | Consumir eventos do domínio de origem e reconstruir projeções sem alterar os registros originais. | DEVERIA |

### 4.11 Domínio 11 — Administração de Plataforma e Operações de Auditoria

Trata-se, inicialmente, de uma aplicação administrativa/funcionalidade do tipo BFF (Backend-for-Frontend), e não de um microsserviço de negócio independente e obrigatório.

| ID | Requisito | Prioridade |
|---|---|---|
| FR-ADM-001 | Fornecer uma interface administrativa protegida por verificações de funções/permissões no lado do servidor. | OBRIGATÓRIO |
| FR-ADM-002 | Permitir que a equipe autorizada visualize e gerencie usuários por meio das operações suportadas pelo domínio de Identidade (Identity). | OBRIGATÓRIO |
| FR-ADM-003 | Permitir que a equipe autorizada analise perfis profissionais por meio das operações suportadas pelo domínio de Perfis (Profiles). | OBRIGATÓRIO |
| FR-ADM-004 | Permitir que a equipe autorizada gerencie categorias por meio das operações suportadas pelo domínio de Catálogo (Catalog). | OBRIGATÓRIO |
| FR-ADM-005 | Permitir que a equipe autorizada investigue problemas relacionados a agendamentos/pagamentos, utilizando APIs do domínio proprietário e as devidas permissões. | OBRIGATÓRIO |
| FR-ADM-006 | Vincular moderadores ao fluxo de trabalho de casos do domínio de Moderação, em vez de duplicar esse fluxo. | OBRIGATÓRIO |
| FR-ADM-007 | Registrar ações administrativas importantes, incluindo o autor, a operação, o registro de data/hora e o resultado; capturar o motivo sempre que a política exigir. | RECOMendado |
| FR-ADM-008 | Evitar gravações diretas em bancos de dados de outros domínios. | OBRIGATÓRIO |

## 5. Requisitos de integração entre domínios

| ID | Requisito | Prioridade |
|---|---|---|
| INT-001 | Cada domínio deve permanecer como o proprietário oficial (autoridade) de seus próprios dados; outros domínios não devem gravar diretamente em seu banco de dados. | OBRIGATÓRIO |
| INT-002 | Os nomes e payloads (conteúdos) dos eventos no Kafka devem ser versionados e documentados. | OBRIGATÓRIO |
| INT-003 | Todo evento deve conter um ID de evento, tipo/versão do evento, data/hora da ocorrência, ID da entidade e ID de correlação; um ID de causalidade deve ser fornecido quando relevante. | OBRIGATÓRIO |
| INT-004 | Os consumidores (consumers) devem ser idempotentes, uma vez que a entrega duplicada é possível. | OBRIGATÓRIO |
| INT-005 | O tratamento de eventos com falha deve contar com procedimentos documentados de nova tentativa (*retry*) e de *dead-letter* (fila de mensagens não processadas), além de monitoramento. | OBRIGATÓRIO antes da entrada em produção |
| INT-006 | Eventos não devem conter senhas, *tokens* de acesso, credenciais de cartões de pagamento ou dados pessoais desnecessários. | OBRIGATÓRIO |
| INT-007 | Uma operação no banco de dados que altera o estado e o evento gerado a partir dela não devem divergir silenciosamente; avalie o uso de *transactional outbox* ou um padrão de confiabilidade equivalente. | OBRIGATÓRIO antes da entrada em produção |
| INT-008 | Chamadas síncronas são necessárias quando uma resposta autoritativa atual é exigida; o Kafka não deve ser utilizado como um mecanismo de RPC síncrono *ad hoc*. | OBRIGATÓRIO |
| INT-009 | Fluxos de trabalho que abrangem múltiplos domínios devem documentar falhas parciais e comportamentos de compensação, especialmente em processos de agendamento, pagamento ou reembolso. | OBRIGATÓRIO antes do lançamento de pagamentos |

### Proposta inicial de contratos de eventos (nomes provisórios)

- `identity.account-created.v1`
- `identity.account-status-changed.v1`
- `profiles.professional-profile-activated.v1`
- `profiles.professional-profile-restricted.v1`
- `catalog.service-offer-published.v1`
- `catalog.service-offer-updated.v1`
- `availability.schedule-updated.v1`
- `availability.time-block-changed.v1`
- `appointments.appointment-created.v1`
- `appointments.appointment-confirmed.v1`
- `appointments.appointment-cancelled.v1`
- `appointments.appointment-completed.v1`
- `payments.payment-status-changed.v1`
- `reviews.review-published.v1`
- `moderation.case-decided.v1`

Estes nomes constituem um vocabulário proposto, e não o esquema final dos eventos. Para cada evento, antes da implementação, é necessário definir um esquema, um produtor, um consumidor, uma política de versionamento e uma estratégia para o tratamento de falhas.

## 6. Requisitos não funcionais

| ID | Categoria | Requisito | Abordagem de validação |
|---|---|---|---|
| NFR-SEC-001 | Segurança | Senhas nunca devem ser armazenadas em texto simples; utilize um mecanismo de hash de senhas que receba manutenção. | Revisão de implementação e testes de autenticação. |
| NFR-SEC-002 | Autorização | Aplique permissões nos endpoints/casos de uso do lado do servidor, e não apenas na interface do usuário (UI). | Testes negativos de permissões (papéis) e de propriedade de dados. |
| NFR-SEC-003 | Segredos | Nunca inclua senhas, chaves de assinatura, credenciais de provedores ou tokens de produção no controle de versão (commit). | Varredura de segredos e revisão de configuração. |
| NFR-PRV-001 | Privacidade | Colete apenas os dados necessários para os fins definidos e restrinja o acesso a dados privados de perfil ou financeiros. | Revisão do fluxo de dados e testes de acesso. |
| NFR-PRV-002 | LGPD | Forneça informações transparentes sobre privacidade e um processo documentado para atender às solicitações dos titulares dos dados. | Revisão jurídica/do produto antes do lançamento. |
| NFR-DATA-001 | Integridade | A prevenção de conflitos em reservas deve ser garantida no ponto de escrita principal (fonte da verdade), incluindo o tratamento de requisições simultâneas. | Teste de integração com concorrência. |
| NFR-DATA-002 | Integridade | Callbacks de pagamento e eventos via Kafka devem ser processáveis ​​de forma segura, mesmo que ocorram múltiplas vezes. | Testes de duplicidade/repetição (replay). |
| NFR-PERF-001 | Desempenho | Meta inicial: 95% das requisições internas comuns da aplicação devem ser concluídas em até 3 segundos sob uma carga definida, excluindo a latência de serviços de terceiros. | Teste de carga com ambiente e carga de trabalho documentados. |
| NFR-UX-001 | Usabilidade | Tarefas principais devem apresentar estados claros de carregamento, sucesso, ausência de dados (vazio), validação e erro. | Testes de usabilidade/aceitação. |
| NFR-RESP-001 | Responsividade | As jornadas principais devem ser operáveis ​​nos dispositivos móveis e desktops suportados. | Testes de interface responsiva. |
| NFR-ACC-001 | Acessibilidade | Adotar o padrão WCAG 2.2 nível AA para as jornadas principais, sempre que possível dentro do escopo acordado. | Verificações de acessibilidade automatizadas e manuais. |
| NFR-OBS-001 | Observabilidade | Falhas críticas, falhas no processamento de eventos e erros de integração externa devem ser registrados/monitorados, sem incluir dados pessoais desnecessários. | Monitoramento e inspeção de logs. |
| NFR-MAINT-001 | Manutenibilidade | As regras de domínio devem ser testáveis ​​sem a necessidade de um servidor web, broker Kafka ou provedor de pagamento externo. | Estrutura de testes unitários/revisão de código. |
| NFR-TEST-001 | Testabilidade | Regras críticas relacionadas a permissões, ofertas, agendamentos, pagamentos e elegibilidade para avaliações devem possuir testes automatizados. | Relatório de testes em CI (Integração Contínua). |
| NFR-COMP-001 | Compatibilidade | As versões de navegadores suportadas e a política de compatibilidade de API devem estar documentadas. | Matriz de compatibilidade de navegadores e testes de contrato. |
| NFR-AVAIL-001 | Disponibilidade | Uma meta de disponibilidade em produção e objetivos de recuperação devem ser definidos antes do lançamento público; não prometa um SLA sem a devida sustentação operacional. | Revisão de prontidão operacional. |

## 7. Exclusões do MVP

A primeira versão não inclui portfólio interno de fotos, recomendações/correspondências via IA, previsão de demanda, programas de fidelidade avançados, administração complexa de múltiplos estabelecimentos/franquias, automação de marketing ou gestão financeira profissional avançada. Links para perfis em redes sociais externas são permitidos. Estes itens poderão ser reconsiderados após a análise de evidências de uso.

## 8. Decisões de negócio pendentes

| ID | Decisão | Requisitos afetados |
|---|---|---|
| DEC-01 | Momento do pagamento e métodos aceitos | FR-PAY-001, FR-APT-002, fluxo de agendamento |
| DEC-02 | Provedor de pagamento e taxas | FR-PAY-001–008 |
| DEC-03 | Comissões e cronograma de repasses | FR-PAY-005, FR-ADM-005 |
| DEC-04 | Prazo para cancelamento, penalidades, política de reembolso e tratamento de não comparecimento (*no-show*) | FR-APT-004–007, compensação de pagamentos | | DEC-05 | Confirmação automática de agendamento versus aceitação pelo profissional | FR-APT-001–005 |
| DEC-06 | Verificação de identidade profissional | FR-PROF-008 |
| DEC-07 | Campos obrigatórios no perfil e precisão da localização pública | FR-PROF-004, FR-DSC-002 |
| DEC-08 | Classificação em buscas e definição de “disponível agora” | FR-DSC-002, FR-DSC-005 |
| DEC-09 | Canais de notificação | integrações entre domínios e UX |
| DEC-10 | Escala de avaliação, política de uma única avaliação por usuário, edição/respostas | FR-REV-001–006 |
| DEC-11 | Estratégia de implementação: microsserviços independentes versus monólito modular para um MVP inicial | Arquitetura e operações |

## 9. Aprovação e controle de alterações

Um requisito só é considerado aprovado após o *product owner* confirmar seu comportamento e as decisões pendentes relevantes serem resolvidas. Alterações devem, sempre que possível, manter os IDs originais, documentar o motivo da mudança e atualizar os casos de uso, contratos de eventos, testes e a rastreabilidade relacionados. Requisitos não constituem prova de implementação; a conclusão é demonstrada pela aprovação nos testes de aceitação e testes automatizados previamente acordados.