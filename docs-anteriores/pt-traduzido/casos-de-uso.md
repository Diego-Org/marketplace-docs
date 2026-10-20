# Casos de Uso de Domínio — Marketplace de Serviços de Beleza

**ID do Documento:** UC-DOM-001
**Versão:** 1.0
**Status:** Base de referência proposta

## 1. Convenções

- **Ator principal:** inicia o caso de uso.
- **Pré-condições:** devem ser verdadeiras antes do início do fluxo.
- **Pós-condições:** descrevem o resultado após a conclusão bem-sucedida.
- **Eventos:** referem-se a eventos de integração assíncronos propostos; não substituem a validação síncrona quando uma resposta imediata é necessária.
- Nomes e estados exatos são provisórios nos casos em que uma decisão de negócio ainda não foi tomada.

A recomendação inicial de implementação consiste em quatro serviços implantáveis: Identidade, Perfis, Catálogo e um serviço combinado de Agendamento e Compromissos, abrangendo casos de uso dos Domínios 5 e 6.

---

## Domínio 1 — Identidade e Acesso

### UC-IAM-01 — Registrar conta
**Ator principal:** Cliente ou Profissional.
**Pré-condições:** O usuário ainda não está registrado com o identificador de login normalizado.

**Fluxo principal**
1. O usuário seleciona o tipo de conta e envia os dados de registro necessários.
2. O serviço valida o formato, os campos obrigatórios e a unicidade do identificador.
3. O serviço armazena a conta e a credencial processada de forma segura (caso o registro seja feito via senha).
4. O serviço atribui a função (role) inicial de acordo com o fluxo de cadastro.
5. O serviço retorna o identificador da conta e a próxima etapa necessária.
6. O serviço publica o evento `identity.account-created.v1` após a confirmação (commit) da criação da conta.

**Alternativas/erros:** Campos inválidos; identificador já em uso; falha em provedor de identidade externo; falha temporária de persistência. Não deve ser criada uma conta duplicada em caso de nova tentativa.

**Pós-condição:** Uma conta existe no estado inicial definido pelas políticas. A criação do perfil pode ocorrer em uma etapa ou serviço separado.

### UC-IAM-02 — Autenticar usuário
**Ator principal:** Usuário registrado.
**Pré-condições:** A conta existe e não está bloqueada para login.

**Fluxo principal:** O usuário envia suas credenciais ou utiliza uma identidade social aprovada; o serviço verifica a identidade, checa o estado da conta e emite o resultado de autenticação configurado, incluindo a função (role) e as reivindicações (claims) do usuário. **Alternativas/erros:** Credenciais inválidas; conta desativada/suspensa; limite de requisições atingido; provedor de identidade indisponível. As respostas não devem revelar se detalhes sensíveis da conta são válidos, em conformidade com a política de segurança.

**Pós-condição:** É emitida uma sessão/token válido ou a autenticação é rejeitada.

### UC-IAM-03 — Recuperar acesso
**Ator principal:** Titular da conta.
**Fluxo principal:** O usuário solicita a recuperação; o serviço responde sem revelar informações desnecessárias sobre a existência da conta; envia ou inicia um desafio de recuperação com validade limitada; valida o desafio; aceita uma nova credencial; invalida o desafio de recuperação ou o estado da sessão correspondente.

**Pós-condição:** O acesso é restaurado de forma segura ou a solicitação é rejeitada. Tokens de recuperação não devem ser registrados em logs nem publicados em eventos.

### UC-IAM-04 — Alterar status/papel de acesso
**Ator principal:** Administrador autorizado ou fluxo de trabalho interno aprovado.
**Fluxo principal:** O solicitante requer uma alteração permitida de papel ou status; o serviço verifica a autorização; valida a transição; registra a alteração; emite `identity.account-status-changed.v1` ou `identity.role-assigned.v1`.

**Alternativas:** O solicitante não possui permissão; a transição é inválida; a conta não foi encontrada. Toda ação deve ser passível de auditoria.

---

## Domínio 2 — Perfis e Profissionais

### UC-PROF-01 — Criar perfil de cliente
**Ator principal:** Cliente.
**Pré-condições:** Existe uma conta válida.

**Fluxo principal:** O cliente envia os campos do perfil; o serviço valida os campos e as regras de visibilidade; o perfil é vinculado ao `accountId`; o serviço retorna o estado do perfil e as próximas ações permitidas.

**Alternativas:** O perfil já existe; dados inválidos; tentativa não autorizada de vincular outro ID de conta.

**Pós-condição:** O perfil do cliente é criado, e os campos privados ficam acessíveis apenas a partes autorizadas.

### UC-PROF-02 — Criar ou atualizar perfil profissional
**Ator principal:** Profissional.
**Pré-condições:** A conta possui o papel de Profissional.

**Fluxo principal:** O profissional envia nome/detalhes de exibição, área de atuação, descrição e links para redes sociais (opcionais); o serviço valida o formato das URLs e os campos obrigatórios mínimos; salva como rascunho ou como perfil ativo, conforme as regras de publicação. publica `profiles.professional-profile-activated.v1` quando a ativação ocorre.

**Alternativas:** Dados obrigatórios ausentes; link malformado; conta não elegível; perfil suspenso que não pode ser ativado pelo usuário.

**Pós-condição:** O perfil é salvo em um estado explícito. A publicação não cria automaticamente ofertas de serviço; o Catálogo permanece como o responsável pelas ofertas.

**UC-PROF-03 — Restringir ou suspender perfil profissional**
**Ator principal:** Fluxo de trabalho administrativo/de moderação autorizado.
**Fluxo principal:** Uma solicitação autorizada identifica o perfil e o motivo; o sistema de Perfis valida a autoridade da solicitação e o estado atual; altera o estado do perfil; registra a referência ao ator/motivo; publica um evento de restrição.

**Pós-condição:** A descoberta pública e o comportamento de novas ofertas podem reagir ao estado do perfil; registros de agendamentos anteriores permanecem intactos.

---

## Domínio 3 — Catálogo de Serviços e Ofertas

### UC-CAT-01 — Manter taxonomia de categorias/serviços
**Ator principal:** Administrador autorizado.
**Fluxo principal:** O administrador cria ou atualiza uma definição de categoria/serviço; o sistema de catálogo valida o nome, o status e a hierarquia; a alteração é salva e um evento é emitido, caso seja necessário para projeções.

**Alternativas:** Entrada de taxonomia duplicada ou inválida; a exclusão é proibida, pois registros históricos fazem referência a ela. Deve-se priorizar a desativação ou o arquivamento em vez da exclusão definitiva.

**Pós-condição:** Uma definição canônica de categoria/serviço fica disponível para ofertas elegíveis.

### UC-CAT-02 — Publicar oferta profissional
**Ator principal:** Profissional.
**Pré-condições:** O perfil do profissional está elegível para publicação.

**Fluxo principal:** O profissional seleciona uma definição de categoria/serviço ou fornece um nome de oferta personalizado permitido; especifica preço, duração e descrição; o Catálogo valida o preço, a duração e a titularidade; a oferta é publicada; o evento `catalog.service-offer-published.v1` é emitido.

**Alternativas:** Perfil inativo; preço inválido; duração zero ou negativa; categoria indisponível; tentativa de gerenciar a oferta de outro profissional.

**Pós-condição:** A oferta fica ativa e elegível para busca/agendamento. O agendamento efetivo realiza uma nova validação do estado da oferta.

### UC-CAT-03 — Alterar ou desativar oferta
**Ator principal:** Profissional ou administrador autorizado.
**Fluxo principal:** O solicitante requisita uma alteração; o Catálogo valida a titularidade e os campos; armazena os novos termos ou desativa a oferta; publica o evento correspondente.

**Pós-condição:** Novos agendamentos utilizam os termos da oferta ativa no momento. Agendamentos existentes preservam os termos vigentes na data em que foram realizados.

---

## Domínio 4 — Descoberta e Busca

### UC-DSC-01 — Buscar profissionais/ofertas
**Ator principal:** Cliente.
**Pré-condições:** A busca pública está disponível.

**Fluxo principal:** O cliente envia uma categoria, serviço ou nome, além de filtros opcionais; o sistema de busca valida os filtros, consulta o modelo de leitura (*read model*) e retorna resumos de perfis e ofertas ativos e elegíveis, seguindo a ordem documentada. fornece orientações para o caso de não haver resultados (estado vazio).

**Alternativas:** Filtro não suportado; projeção defasada; índice dependente temporariamente indisponível. Uma resposta com restrições (degradada) não deve afirmar que um slot candidato está garantido.

**Pós-condição:** O cliente recebe resultados de descoberta equivalentes ou um estado explícito de "vazio" ou "erro".

### UC-DSC-02 — Visualizar detalhes de um profissional e de uma oferta
**Ator principal:** Cliente.
**Fluxo principal:** O cliente abre um resultado; a consulta ao perfil exibe campos públicos permitidos, ofertas ativas, avaliações publicadas e links para redes sociais; campos privados do perfil são omitidos.

**Pós-condição:** O cliente pode comparar o nome do serviço, preço, duração, localização/área de atendimento e detalhes de reputação antes de realizar o agendamento.

### UC-DSC-03 — Atualizar projeção de busca
**Ator principal:** Consumidor/sistema de busca.
**Gatilho:** Evento relacionado a perfil, oferta, avaliação ou outro evento assinado.

**Fluxo principal:** O consumidor valida o esquema/versão do evento; aplica a atualização da projeção de forma idempotente; registra o resultado do processamento e o status de atualização (frescor) da projeção.

**Alternativas:** Evento duplicado; versão não suportada; evento malformado; falha no processamento. Registros com falha seguem uma política de nova tentativa (retry) ou são encaminhados para uma fila de mensagens não processáveis ​​(*dead-letter queue*).

**Pós-condição:** O modelo de leitura reflete o evento após o processamento; o domínio de origem permanece inalterado.

---

## Domínio 5 — Agenda e Disponibilidade

### UC-AVL-01 — Configurar horário de trabalho
**Ator principal:** Profissional.
**Pré-condições:** A conta do profissional está ativa e autorizada.

**Main flow:** O profissional insere os dias/intervalos de horário recorrentes e o fuso horário; o serviço valida os limites dos intervalos e possíveis sobreposições; salva a agenda; publica o evento `availability.schedule-updated.v1`.

**Alternativas:** Intervalo inválido (término antes do início); fuso horário não suportado ou ausente; conflito da atualização da agenda com agendamentos existentes. O tratamento de conflitos deve ser explícito.

**Pós-condição:** A disponibilidade do profissional pode ser calculada a partir da agenda salva.

### UC-AVL-02 — Bloquear ou desbloquear um período
**Ator principal:** Profissional.
**Fluxo principal:** O profissional seleciona a data/hora e, opcionalmente, um motivo; o serviço valida o intervalo; verifica conflitos com agendamentos existentes por meio do mesmo serviço/módulo ou de um fluxo de coordenação centralizado. registra o bloqueio ou a recusa e explica o conflito.

**Pós-condição:** Os novos horários sugeridos excluem o período bloqueado; agendamentos já existentes não são removidos silenciosamente.

**UC-AVL-03 — Consultar disponibilidade**
**Ator principal:** Cliente ou módulo de Agendamentos.
**Pré-condições:** Profissional, oferta, intervalo de datas e duração são válidos.

**Fluxo principal:** O serviço combina horários recorrentes, exceções, bloqueios, regras de fuso horário, duração e intervalos de agendamento já ocupados; retorna os horários disponíveis e as restrições relevantes.

**Importante:** Este resultado constitui uma lista de opções. A confirmação final do agendamento deve verificar, de forma atômica, se o intervalo permanece livre.


**Pós-condição:** O solicitante recebe intervalos candidatos (podendo não haver nenhum).

---

## Domínio 6 — Agendamentos e Atendimentos

### UC-APT-01 — Criar agendamento
**Ator principal:** Cliente.
**Atores de apoio:** Catálogo, Disponibilidade, Pagamento (dependendo do fluxo aprovado).
**Pré-condições:** O cliente está autorizado; o profissional/perfil e a oferta são elegíveis; um intervalo de tempo candidato foi selecionado.

**Fluxo principal**
1. O cliente revisa o profissional, a oferta, o preço, a duração, a data/hora e as condições de cancelamento.
2. O serviço de agendamento revalida a elegibilidade da oferta e as restrições de agenda.
3. O serviço realiza uma verificação definitiva de conflitos no ponto de registro (escrita).
4. O serviço armazena o agendamento, incluindo um registro (snapshot) do preço/duração acordados e o estado inicial definido pelo fluxo aprovado.
5. O serviço emite o evento `appointments.appointment-created.v1` e, conforme apropriado, o evento de estado subsequente.
6. O cliente e o profissional são informados sobre o estado resultante.

**Alternativas:** O intervalo foi ocupado por uma solicitação concorrente; a oferta foi desativada; o perfil tornou-se inelegível; a solicitação foi reenviada; o pagamento está pendente ou falhou. Os estados do agendamento e do pagamento devem permanecer coerentes, porém distintos.

**Pós-condição:** É confirmado, no máximo, um agendamento incompatível para o mesmo profissional/intervalo de tempo.

### UC-APT-02 — Cancelar agendamento
**Ator principal:** Cliente ou profissional autorizado, conforme a política estabelecida.
**Fluxo principal:** O solicitante requisita o cancelamento; o serviço verifica a titularidade, o estado atual, o momento da solicitação e a política aplicável; apresenta ou valida os efeitos; registra o cancelamento; atualiza a disponibilidade; emite o evento `appointments.appointment-cancelled.v1`; o domínio de pagamento processa, de forma independente, qualquer ação de reembolso.

**Alternativas:** O cancelamento não é permitido naquela etapa; outra ação alterou o estado simultaneamente; o provedor de reembolso falha. O cancelamento de um agendamento não deve implicar, erroneamente, que um reembolso foi concluído.

**Pós-condição:** O status do agendamento e o status financeiro podem ser consultados e auditados individualmente.

### UC-APT-03 — Concluir atendimento ou registrar ausência (no-show)
**Ator principal:** Profissional ou fluxo operacional autorizado.
**Fluxo principal:** O ator acessa um agendamento elegível; seleciona a conclusão ou o registro de ausência (no-show), conforme permitido pela política; O serviço valida a transição e a autorização; registra o evento e publica o evento de ciclo de vida correspondente.

**Alternativas:** Transição de estado inválida; o ator não possui acesso; agendamento já finalizado. Penalidades por não comparecimento (*no-show*) não são aplicadas até que a política seja aprovada.

**Pós-condição:** Processos de análise de elegibilidade e de *analytics* podem reagir ao estado final de comparecimento.

### UC-APT-04 — Resolver tentativas de agendamento concorrentes
**Ator principal:** Sistema.
**Gatilho:** Duas ou mais solicitações de agendamento simultâneas visam intervalos sobrepostos.

**Fluxo principal:** Cada solicitação realiza uma validação final na fronteira de autoridade; regras de transação/bloqueio/restrição permitem a confirmação apenas de um agendamento válido e sem sobreposição; a solicitação que não obteve sucesso recebe uma resposta explícita de conflito.

**Pós-condição:** Não existem dois agendamentos confirmados incompatíveis para o mesmo profissional. A consistência eventual do Kafka, por si só, não atende a esse requisito.

---

## Domínio 7 — Pagamentos e Repasses

### UC-PAY-01 — Iniciar pagamento de agendamento
**Ator principal:** Cliente.
**Atores de apoio:** Provedor de pagamento, serviço de agendamento.
**Pré-condições:** O estado do agendamento/checkout permite o pagamento conforme a política aprovada.

**Fluxo principal:** O serviço de pagamento cria um registro interno de pagamento e uma chave de idempotência; solicita o pagamento ao provedor; retorna um resultado (pendente/ação necessária/falha); armazena referências do provedor; informa o status ao serviço de orquestração de agendamentos.

**Alternativas:** Provedor indisponível; solicitação de início duplicada; pagamento rejeitado; *timeout* com resultado desconhecido. O serviço deve reconciliar resultados desconhecidos em vez de presumir falha ou sucesso.

**Pós-condição:** O estado do pagamento é armazenado separadamente e correlacionado ao ID do agendamento.

### UC-PAY-02 — Processar *callback*/*webhook* do provedor
**Ator principal:** Provedor de pagamento.
**Fluxo principal:** O provedor envia um *callback*; o serviço verifica a autenticidade, remove duplicatas do evento, valida o valor/moeda/referência, atualiza o estado do pagamento e publica o evento `payments.payment-status-changed.v1`.

**Alternativas:** Assinatura inválida; transação desconhecida; evento repetido; status fora de ordem. *Callbacks* inválidos não alteram o estado financeiro.

**Pós-condição:** O histórico de transações reflete os resultados verificados pelo provedor. ### UC-PAY-03 — Processar reembolso/pagamento
**Ator principal:** Fluxo de trabalho de negócios aprovado ou operador autorizado.
**Fluxo principal:** O serviço valida a elegibilidade conforme a política e o registro financeiro; solicita a operação ao provedor; monitora o status (pendente/concluído/falha); publica atualizações de status; registra detalhes de reconciliação.

**Pós-condição:** O reembolso/pagamento não é considerado concluído até que seja confirmado por um status verificado do provedor.

---

## Domínio 8 — Avaliações e Reputação

### UC-REV-01 — Enviar avaliação
**Ator principal:** Cliente.
**Pré-condições:** O cliente possui um agendamento concluído elegível e ainda não o avaliou, conforme a política estabelecida.

**Fluxo principal:** O cliente envia uma nota e um comentário opcional; o serviço verifica a elegibilidade do agendamento e a identidade/titularidade; valida a nota/comentário; armazena a avaliação; torna-a visível ou a coloca em moderação (conforme a política); e emite um evento de avaliação.

**Alternativas:** Agendamento incompleto/inelegível; avaliação duplicada; nota inválida; conteúdo bloqueado pela política.

**Pós-condição:** A avaliação fica vinculada a um agendamento específico e contribui para a reputação apenas quando estiver visível/elegível.

### UC-REV-02 — Atualizar resumo de reputação
**Ator principal:** Processo de projeção/agregação de avaliações.
**Gatilho:** Avaliação publicada, ocultada ou editada (conforme a política).

**Fluxo principal:** O serviço recalcula a contagem e a média utilizando as avaliações visíveis e elegíveis; armazena o resumo; e emite uma atualização de projeção, quando necessário.

**Pós-condição:** O resumo público segue uma fórmula documentada e não inclui avaliações ocultadas ou inelegíveis.

---

## Domínio 9 — Denúncias e Moderação

### UC-MOD-01 — Enviar denúncia
**Ator principal:** Cliente ou Profissional.
**Fluxo principal:** O denunciante seleciona o tipo de alvo e o motivo; fornece detalhes opcionais; o sistema valida os campos permitidos e armazena a denúncia; retorna um ID de referência; e emite o evento `moderation.report-created.v1`.

**Alternativas:** Alvo inválido; taxa de solicitações abusiva; denúncia duplicada ou já existente (conforme a política de desduplicação).

**Pós-condição:** A denúncia fica disponível para moderadores autorizados, sem ser exposta publicamente.

### UC-MOD-02 — Analisar e decidir sobre caso de moderação
**Ator principal:** Moderador/Administrador autorizado.
**Pré-condições:** O membro da equipe possui permissão de moderação.

**Fluxo principal:** O moderador abre o caso e acessa as evidências permitidas; analisa o contexto relevante proveniente dos domínios proprietários; registra a decisão, o motivo e a ação solicitada; o histórico do caso é preservado; e um evento é publicado.

**Alternativas:** Evidências insuficientes; conflito de acesso; o alvo não existe mais; solicitação de ação rejeitada pelo domínio proprietário. O estado do caso registra o status resultante.

**Pós-condição:** A decisão é rastreável; o domínio proprietário é responsável pela efetiva alteração do estado de destino.

---

## Domínio 10 — Análises e Indicadores

### UC-ANL-01 — Visualizar indicadores do profissional
**Ator principal:** Profissional.
**Pré-condições:** O profissional está autenticado e autorizado a acessar as métricas daquele perfil.

**Fluxo principal:** O profissional seleciona um período; o serviço consulta as projeções; apresenta a contagem de agendamentos, atendimentos concluídos/cancelados, resumo de avaliações e indicadores de agenda com base nos dados disponíveis; exibe o período e a data da última atualização, quando aplicável.

**Alternativas:** Dados insuficientes; projeção com atraso; período não suportado. A interface do usuário (UI) não deve sugerir precisão em tempo real para dados que possam estar sujeitos a latência.

**Pós-condição:** O profissional visualiza os indicadores autorizados e seu respectivo recorte temporal.

### UC-ANL-02 — Atualizar projeção de análises
**Ator principal:** Consumidor de análises (Analytics).
**Gatilho:** Evento relacionado a agendamento, pagamento, avaliação, catálogo ou disponibilidade.

**Fluxo principal:** Valida o evento; atualiza as métricas de forma idempotente; trata correções ou eventos duplicados; registra o atraso no processamento; permite a reconstrução da projeção.

**Pós-condição:** As análises são derivadas de eventos do domínio proprietário e não alteram os registros operacionais.

---

## Domínio 11 — Administração de Plataforma e Operações de Auditoria

Este componente é tratado como uma aplicação/funcionalidade administrativa, e não como um proprietário independente de todas as entidades do domínio.

### UC-ADM-01 — Administrar um usuário/perfil/categoria
**Ator principal:** Administrador autorizado.
**Fluxo principal:** O administrador acessa a tela administrativa correspondente; o sistema verifica a permissão; a operação é enviada ao serviço proprietário; o serviço proprietário valida a regra de negócio e efetiva a alteração; a resposta é exibida; a ação administrativa é registrada (log).

**Alternativas:** Permissão insuficiente; transição inválida; serviço proprietário indisponível; envio duplicado. A aplicação administrativa não deve realizar gravações diretamente no banco de dados do serviço proprietário.

**Pós-condição:** O serviço proprietário permanece como a fonte da verdade (*source of truth*) e a ação torna-se rastreável.

### UC-ADM-02 — Investigar problemas com agendamentos/pagamentos
**Ator principal:** Administrador de suporte autorizado.
**Fluxo principal:** O administrador realiza uma busca utilizando identificadores permitidos; o sistema recupera, a partir dos serviços proprietários, as visualizações de agendamento e pagamento adequadas ao perfil do administrador; os estados relevantes são exibidos separadamente; o administrador executa apenas ações compensatórias ou operacionais previamente aprovadas; a ação é registrada (log).

**Alternativas:** Registro inexistente; acesso a detalhes financeiros negado; status do prestador pendente; divergência temporária entre serviços. Ferramentas de suporte não devem marcar, manualmente, um reembolso ou transação como concluídos sem evidências que comprovem a legitimidade da operação.

**Pós-condição:** O resultado da investigação é registrado sem violar as invariantes de pagamento ou agendamento.

### UC-ADM-03 — Auditar operação administrativa
**Ator principal:** Sistema.
**Gatilho:** Ação administrativa relevante.

**Fluxo principal:** Registra-se o ator, a ação, a referência do alvo, o registro de data/hora (*timestamp*), o resultado, o ID de correlação e a justificativa necessária; o acesso de leitura aos registros de auditoria é restrito; os dados são retidos conforme uma política aprovada.

**Pós-condição:** A ação pode ser investigada sem que haja o armazenamento de credenciais, tokens ou dados pessoais desnecessários.

---

## 2. Cenários entre domínios (*Cross-domain*)

### UC-X-01 — Agendamento de ponta a ponta (*MVP* central)
1. O cliente realiza a autenticação (Domínio 1).
2. O profissional possui um perfil ativo (Domínio 2) e uma oferta ativa (Domínio 3).
3. O cliente localiza a oferta (Domínio 4 ou um módulo de consulta inicial). 4. A verificação de disponibilidade retorna os intervalos elegíveis (Domínio 5).
5. O fluxo de agendamento valida, de forma atômica, o intervalo selecionado e registra a reserva (Domínio 6, ou serviço combinado).
6. O pagamento é iniciado e acompanhado separadamente, caso o fluxo escolhido exija (Domínio 7).
7. Notificações sobre o agendamento/pagamento e as projeções de leitura são atualizadas de forma assíncrona.
8. Após a conclusão bem-sucedida, o cliente pode revisar os detalhes (Domínio 8).

**Casos de falha crítica:** oferta desativada entre a busca e a finalização (checkout); horário ocupado simultaneamente por outro usuário; *timeout* (tempo limite excedido) no provedor de pagamento; *webhook* duplicado; cancelamento solicitado antes da efetivação do reembolso; falha no consumidor de eventos. Cada caso deve resultar em um estado explícito que permita uma nova tentativa ou reconciliação.

**UC-X-02 — Perfil profissional torna-se pesquisável**
1. O sistema de identidade cria a conta e emite um evento de criação de conta.
2. O profissional completa seu perfil e torna-se ativo.
3. O profissional publica uma ou mais ofertas.
4. A projeção de busca consome os eventos de perfil/oferta e indexa os registros ativos.
5. A busca realizada pelo cliente retorna o perfil apenas se tanto o perfil quanto a oferta forem elegíveis, de acordo com as regras da projeção.

**Importante:** a indexação assíncrona significa que o perfil pode não aparecer imediatamente. A interface do usuário (UI) deve exibir claramente o status da publicação; o agendamento direto, por sua vez, sempre realiza a validação com base na fonte da verdade (*source-of-truth*) atual.

**UC-X-03 — Cancelamento de agendamento com pagamento/reembolso**
1. O cliente solicita o cancelamento.
2. O serviço de agendamento avalia a política de cancelamento e registra a transição de estado permitida.
3. O serviço de pagamento recebe, de forma independente, uma solicitação de reembolso autorizada, caso a política exija.
4. O serviço de pagamento processa o status junto ao provedor e emite eventos referentes ao reembolso/pagamento.
5. O cliente visualiza o agendamento como cancelado, enquanto o reembolso pode estar pendente, concluído ou ter falhado.
6. A equipe de suporte/administração pode investigar a situação, sem equiparar o cancelamento à conclusão do reembolso.

---

## 3. Decisões em aberto que afetam estes casos de uso

- Se o agendamento é confirmado automaticamente ou se depende da aceitação por parte do profissional.
- Se o pagamento ocorre antes da confirmação do agendamento, após a confirmação ou por meio de uma alternativa compatível.
- Estados específicos do agendamento e as transições permitidas.
- Prazo para cancelamento, penalidades, procedimentos de reembolso e política para casos de não comparecimento (*no-show*).
- Campos obrigatórios para os profissionais e requisitos de verificação.
- Escala de avaliação, política para duplicatas/edições/respostas e regras de moderação.
- Critérios de classificação em buscas, nível de detalhamento da localização e canais de notificação.