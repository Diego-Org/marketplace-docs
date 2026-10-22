# **Cenários de teste — Marketplace de serviços de beleza**

Este documento transforma as histórias de usuário em cenários de teste que podem orientar a implementação com TDD em Java e Spring Boot.

Os cenários estão identificados individualmente para facilitar sua rastreabilidade e eles descrevem o comportamento esperado, sem impor uma implementação específica.

## **Convenções utilizadas**

* CT-IDA — Identidade e Acesso.  
* CT-PRO — Perfis e Profissionais.  
* CT-CAT — Catálogo de Serviços.  
* CT-AGE — Disponibilidade e Agendamentos.

Cada cenário contém um identificador, uma descrição, as condições necessárias, a ação e o resultado esperado. Os cenários negativos verificam se operações inválidas são recusadas; os cenários positivos verificam se operações válidas produzem o resultado esperado.

### **Como aplicar os cenários no TDD**

1. Escolha um cenário pequeno e independente.  
2. Escreva o teste automatizado e confirme que ele falha pelo motivo esperado (RED).  
3. Implemente apenas o necessário para o teste passar (GREEN).  
4. Refatore e execute novamente os testes existentes (REFACTOR).

Os códigos HTTP mencionados adiante são sugestões para testes de API REST. Os códigos exatos devem seguir o contrato de API adotado pelo projeto.

## Domínio: Disponibilidade e Agendamentos

Os cenários estão agrupados por história de usuário e, dentro de cada história, separados em **cenários de sucesso** e **cenários de insucesso**. Os IDs e as descrições originais foram preservados.

**Como interpretar:** “sucesso” significa que o sistema deve se comportar corretamente diante daquela situação — inclusive retornar lista vazia ou manter dados consistentes. “Insucesso” significa que a solicitação é inválida, não autorizada ou conflitante e deve ser recusada ou tratada sem violar as regras. O teste de insucesso também deve terminar aprovado quando a rejeição esperada ocorre.

### US-AGE-001 — Definir horários de atendimento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-001-01 | Cadastrar período válido: o período é persistido para o profissional correto. |
| CT-AGE-001-06 | Cadastrar períodos distintos no mesmo dia: ambos são persistidos quando válidos. |
| CT-AGE-001-08 | Consultar os horários após o cadastro: os períodos correspondem aos dados persistidos. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-001-02 | Informar início igual ao término: o período é recusado. |
| CT-AGE-001-03 | Informar início posterior ao término: o período é recusado, salvo regra explícita para períodos que cruzam a meia-noite. |
| CT-AGE-001-04 | Cadastrar horário sem dia obrigatório: a operação é recusada. |
| CT-AGE-001-05 | Cadastrar períodos sobrepostos para o mesmo dia: a aplicação impede a sobreposição ou aplica a regra definida. |
| CT-AGE-001-07 | Tentar configurar a agenda de outro profissional: a operação é recusada. |

### US-AGE-002 — Atualizar horários de atendimento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-002-01 | Atualizar períodos com dados válidos: a nova disponibilidade é persistida. |
| CT-AGE-002-04 | Alterar disponibilidade com agendamentos futuros afetados: o sistema aplica a política definida sem cancelar compromissos silenciosamente. |
| CT-AGE-002-06 | Atualizar períodos de um profissional sem agendamentos: os novos períodos ficam disponíveis para consultas posteriores. |
| CT-AGE-002-07 | Consultar agendamentos históricos após atualização: os registros anteriores são preservados. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-002-02 | Atualizar com horários inválidos: a operação é recusada. |
| CT-AGE-002-03 | Atualizar períodos que se sobrepõem: o resultado respeita a regra de validação definida. |
| CT-AGE-002-05 | Atualizar agenda de outro profissional: a operação é recusada. |

### US-AGE-003 — Bloquear períodos da agenda

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-003-01 | Criar bloqueio com intervalo válido: o bloqueio é persistido. |
| CT-AGE-003-03 | Consultar disponibilidade durante o bloqueio: os horários afetados não são oferecidos. |
| CT-AGE-003-04 | Criar bloqueio parcialmente sobreposto a um horário de atendimento: o intervalo bloqueado é excluído da disponibilidade. |
| CT-AGE-003-07 | Criar bloqueio fora dos horários regulares de atendimento: o sistema trata a solicitação de acordo com a política definida. |
| CT-AGE-003-08 | Consultar o bloqueio após criá-lo: os dados persistidos correspondem ao intervalo solicitado. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-003-02 | Criar bloqueio com início igual ou posterior ao término: a operação é recusada. |
| CT-AGE-003-05 | Criar bloqueio que coincide com agendamento existente: a operação segue a política de conflitos definida. |
| CT-AGE-003-06 | Tentar bloquear agenda de outro profissional: a operação é recusada. |

### US-AGE-004 — Remover um bloqueio da agenda

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-004-01 | Remover bloqueio próprio existente: o bloqueio deixa de restringir a disponibilidade. |
| CT-AGE-004-04 | Remover bloqueio parcialmente sobreposto a um período de atendimento: apenas a restrição correspondente ao bloqueio é removida. |
| CT-AGE-004-05 | Remover bloqueio quando existe outro bloqueio ou agendamento no mesmo intervalo: o horário só volta a ficar disponível se todas as condições permitirem. |
| CT-AGE-004-06 | Repetir a remoção: o sistema mantém um estado consistente. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-004-02 | Remover bloqueio inexistente: o sistema retorna o resultado previsto. |
| CT-AGE-004-03 | Tentar remover bloqueio de outro profissional: a operação é recusada. |

### US-AGE-005 — Consultar horários disponíveis

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-005-01 | Consultar período com horários livres: são retornados intervalos compatíveis com a duração do serviço. |
| CT-AGE-005-02 | Consultar período sem horários livres: é retornada uma lista vazia. |
| CT-AGE-005-03 | Consultar intervalo fora do horário de atendimento: nenhum horário incompatível é oferecido. |
| CT-AGE-005-04 | Consultar intervalo ocupado por agendamento ativo: os horários conflitantes são excluídos. |
| CT-AGE-005-05 | Consultar intervalo parcialmente bloqueado: os horários que se sobrepõem ao bloqueio são excluídos. |
| CT-AGE-005-10 | Consultar disponibilidade: nenhum agendamento é criado nem horário é reservado pela simples consulta. |
| CT-AGE-005-11 | Consultar próximo de um limite de atendimento: o resultado respeita a duração completa e os limites definidos. |
| CT-AGE-005-12 | Consultar em fuso horário ou transição de horário de verão, se aplicável: os horários retornados correspondem à política temporal do sistema. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-005-06 | Serviço tem duração maior que o tempo livre restante: o intervalo não é oferecido. |
| CT-AGE-005-07 | Consultar horário no passado: o horário não é oferecido para nova contratação. |
| CT-AGE-005-08 | Consultar com serviço inexistente ou inativo: a solicitação é recusada. |
| CT-AGE-005-09 | Consultar com profissional inexistente ou indisponível para contratação: a solicitação é tratada conforme o contrato definido. |

### US-AGE-006 — Criar um agendamento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-006-01 | Criar agendamento com cliente, profissional, serviço e horário válidos: o agendamento é persistido. |
| CT-AGE-006-11 | Criar agendamento válido: preço, duração e demais condições contratadas ficam registrados como histórico. |
| CT-AGE-006-12 | Alterar o preço do serviço depois do agendamento: o preço histórico permanece inalterado. |
| CT-AGE-006-14 | Enviar duas requisições concorrentes para o mesmo horário: no máximo uma reserva incompatível é confirmada. |
| CT-AGE-006-15 | Repetir a mesma requisição por falha de comunicação: o sistema segue a política de idempotência adotada e não cria duplicidade indevida. |
| CT-AGE-006-16 | Criar agendamento cujo término coincide exatamente com o início do próximo: o resultado segue a regra de intervalos adotada. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-006-02 | Tentar criar agendamento sem autenticação: a operação é recusada. |
| CT-AGE-006-03 | Informar cliente diferente do usuário autenticado: a operação é recusada, salvo fluxo autorizado específico. |
| CT-AGE-006-04 | Informar profissional inexistente ou não elegível para contratação: a operação é recusada. |
| CT-AGE-006-05 | Informar serviço inexistente, inativo ou não oferecido pelo profissional: a operação é recusada. |
| CT-AGE-006-06 | Solicitar horário no passado: a operação é recusada. |
| CT-AGE-006-07 | Solicitar horário fora da disponibilidade: a operação é recusada. |
| CT-AGE-006-08 | Solicitar horário conflitante com outro agendamento ativo: a operação é recusada. |
| CT-AGE-006-09 | Solicitar horário conflitante com bloqueio: a operação é recusada. |
| CT-AGE-006-10 | Enviar duração inválida ou incompatível com o serviço: a operação é recusada. |
| CT-AGE-006-13 | Falhar a persistência durante a criação: não fica um agendamento parcialmente gravado. |

### US-AGE-007 — Consultar os detalhes de um agendamento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-007-01 | Cliente consulta o próprio agendamento: os detalhes são retornados. |
| CT-AGE-007-02 | Profissional consulta agendamento da própria agenda: os detalhes são retornados. |
| CT-AGE-007-06 | Consultar agendamento cancelado ou concluído: o registro é retornado com o estado correto. |
| CT-AGE-007-07 | Consultar detalhes: preço e duração históricos correspondem aos valores registrados no agendamento. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-007-03 | Cliente tenta consultar agendamento de outra pessoa: o acesso é recusado. |
| CT-AGE-007-04 | Profissional tenta consultar agendamento de outra agenda: o acesso é recusado. |
| CT-AGE-007-05 | Consultar agendamento inexistente: o sistema retorna o erro previsto. |
| CT-AGE-007-08 | Consultar detalhes sem autenticação: o acesso é recusado. |

### US-AGE-008 — Listar meus agendamentos

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-008-01 | Cliente possui agendamentos: são retornados apenas os registros associados a ele. |
| CT-AGE-008-02 | Cliente não possui agendamentos: é retornada uma lista vazia. |
| CT-AGE-008-03 | Cliente solicita apenas agendamentos futuros: o resultado respeita o filtro temporal definido. |
| CT-AGE-008-04 | Cliente solicita histórico: os registros históricos são incluídos conforme os filtros disponíveis. |
| CT-AGE-008-06 | Listagem contém agendamentos com diferentes estados: os estados atuais são apresentados corretamente. |
| CT-AGE-008-07 | Listar agendamentos com parâmetros de paginação válidos, se suportados: os resultados são consistentes entre páginas. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-008-05 | Cliente tenta informar outro identificador para consultar agenda alheia: a autorização é aplicada no servidor. |

### US-AGE-009 — Consultar a agenda profissional

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-009-01 | Profissional consulta agenda com atendimentos: são retornados os registros associados à sua agenda. |
| CT-AGE-009-02 | Profissional sem agendamentos consulta a agenda: é retornada uma lista vazia. |
| CT-AGE-009-03 | Filtrar agenda por período: são retornados apenas os registros compatíveis. |
| CT-AGE-009-04 | Filtrar agenda por estado: os resultados respeitam o filtro solicitado. |
| CT-AGE-009-06 | Consultar agenda com agendamentos cancelados e concluídos: os estados são distinguidos corretamente. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-009-05 | Profissional tenta consultar agenda de outro profissional: o acesso é recusado. |
| CT-AGE-009-07 | Consultar agenda sem autenticação: o acesso é recusado. |

### US-AGE-010 — Cancelar um agendamento como cliente

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-010-01 | Cliente cancela próprio agendamento cancelável: o estado muda para cancelado. |
| CT-AGE-010-05 | Tentar cancelar agendamento já cancelado: o resultado segue a política de idempotência definida. |
| CT-AGE-010-06 | Cancelar dentro do prazo permitido: o cancelamento é aceito. |
| CT-AGE-010-08 | Cancelar agendamento ativo: o registro histórico é preservado. |
| CT-AGE-010-09 | Cancelar agendamento: o horário pode voltar a ficar disponível, desde que nenhuma outra restrição o impeça. |
| CT-AGE-010-10 | Cancelamento com pagamento associado: o resultado financeiro segue o fluxo de pagamento e reembolso definido. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-010-02 | Cliente tenta cancelar agendamento de outra pessoa: a operação é recusada. |
| CT-AGE-010-03 | Tentar cancelar agendamento inexistente: o sistema retorna o erro previsto. |
| CT-AGE-010-04 | Tentar cancelar agendamento concluído: a operação é recusada. |
| CT-AGE-010-07 | Cancelar fora do prazo permitido: o sistema aplica a política de cancelamento definida. |

### US-AGE-011 — Cancelar um agendamento como profissional

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-011-01 | Profissional cancela agendamento cancelável da própria agenda: o estado é atualizado. |
| CT-AGE-011-05 | Cancelar agendamento já cancelado: o sistema mantém um estado consistente. |
| CT-AGE-011-06 | Cancelar agendamento com pagamento associado: os efeitos financeiros seguem a política definida. |
| CT-AGE-011-07 | Cancelar agendamento: o histórico é preservado e o intervalo deixa de ser ocupado por esse compromisso, se aplicável. |
| CT-AGE-011-08 | Cancelar agendamento: a notificação correspondente é produzida, se esse recurso estiver habilitado. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-011-02 | Profissional tenta cancelar agendamento de outra agenda: a operação é recusada. |
| CT-AGE-011-03 | Tentar cancelar agendamento inexistente: o sistema retorna o erro previsto. |
| CT-AGE-011-04 | Tentar cancelar agendamento concluído: a operação é recusada. |

### US-AGE-012 — Confirmar um agendamento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-012-01 | Profissional confirma agendamento pendente da própria agenda: o estado é atualizado. |
| CT-AGE-012-04 | Confirmar agendamento já confirmado: o sistema evita efeitos duplicados. |
| CT-AGE-012-07 | Confirmar agendamento válido: horário, preço e duração registrados não são alterados indevidamente. |
| CT-AGE-012-08 | Confirmar agendamento: a notificação é produzida quando prevista. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-012-02 | Profissional tenta confirmar agendamento de outra agenda: a operação é recusada. |
| CT-AGE-012-03 | Confirmar agendamento inexistente: o sistema retorna o erro previsto. |
| CT-AGE-012-05 | Confirmar agendamento cancelado ou concluído: a operação é recusada. |
| CT-AGE-012-06 | Confirmar agendamento cujo pagamento ainda não satisfaz os requisitos de confirmação: o sistema aplica o fluxo financeiro definido. |

### US-AGE-013 — Registrar a conclusão do atendimento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-013-01 | Profissional conclui agendamento elegível da própria agenda: o estado passa para concluído. |
| CT-AGE-013-06 | Concluir agendamento já concluído: a operação não gera efeitos duplicados. |
| CT-AGE-013-07 | Concluir atendimento: o estado atualizado é retornado nas consultas posteriores. |
| CT-AGE-013-08 | Concluir atendimento: os eventos necessários para habilitar avaliação ou outros processos são produzidos, quando previstos. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-013-02 | Profissional tenta concluir agendamento de outro profissional: a operação é recusada. |
| CT-AGE-013-03 | Tentar concluir agendamento inexistente: o sistema retorna o erro previsto. |
| CT-AGE-013-04 | Tentar concluir agendamento cancelado: a operação é recusada. |
| CT-AGE-013-05 | Tentar concluir agendamento em estado incompatível: a transição é recusada. |

### US-AGE-014 — Registrar ausência do cliente

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-014-01 | Profissional registra ausência em agendamento elegível: o estado é atualizado conforme a regra definida. |
| CT-AGE-014-06 | Registrar ausência duas vezes: não são gerados efeitos duplicados. |
| CT-AGE-014-07 | Registrar ausência com pagamento associado: os efeitos financeiros seguem a política definida. |
| CT-AGE-014-08 | Registrar ausência: o estado atualizado permanece disponível no histórico. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-014-02 | Profissional tenta registrar ausência em agendamento de outra agenda: a operação é recusada. |
| CT-AGE-014-03 | Registrar ausência antes do limite temporal permitido: a operação é recusada. |
| CT-AGE-014-04 | Registrar ausência em agendamento concluído: a operação é recusada. |
| CT-AGE-014-05 | Registrar ausência em agendamento cancelado: a operação é recusada. |

### US-AGE-015 — Impedir conflito entre agendamentos

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-015-02 | Criar agendamentos consecutivos sem sobreposição: ambos são aceitos quando as demais condições permitem. |
| CT-AGE-015-03 | Criar agendamentos sobrepostos para profissionais diferentes: ambos podem ser aceitos, se não houver outra restrição. |
| CT-AGE-015-05 | Tentar reservar período ocupado por agendamento cancelado: o intervalo não é bloqueado por esse registro, desde que não existam outras restrições. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-015-01 | Criar dois agendamentos com intervalos sobrepostos para o mesmo profissional: o segundo é recusado. |
| CT-AGE-015-04 | Tentar reservar período bloqueado: a operação é recusada. |
| CT-AGE-015-06 | Enviar solicitações simultâneas para o mesmo horário: não são confirmadas duas reservas incompatíveis. |
| CT-AGE-015-07 | Enviar solicitações simultâneas para intervalos parcialmente sobrepostos: não são confirmadas reservas incompatíveis. |
| CT-AGE-015-08 | Falha durante a transação de criação: não permanece uma reserva parcial que bloqueie o horário indevidamente. |
| CT-AGE-015-09 | Executar testes de concorrência com o banco de dados utilizado em produção: a garantia contra dupla reserva é mantida. |
| CT-AGE-015-10 | Tentar criar reserva após alteração concorrente da disponibilidade ou criação de bloqueio: a operação revalida as condições e respeita a regra transacional adotada. |

### US-AGE-016 — Notificar sobre alterações no agendamento

#### Cenários de sucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-016-01 | Confirmar agendamento com notificação habilitada: o evento de notificação é produzido. |
| CT-AGE-016-02 | Cancelar agendamento com notificação habilitada: o evento correspondente é produzido. |
| CT-AGE-016-04 | Falhar o envio da notificação: a operação de negócio já persistida não é desfeita indevidamente. |
| CT-AGE-016-05 | Reprocessar evento de notificação: a infraestrutura evita duplicidade quando há suporte à idempotência. |
| CT-AGE-016-06 | Notificação destinada a cliente ou profissional: o destinatário corresponde à parte correta do agendamento. |
| CT-AGE-016-07 | Gerar conteúdo da notificação: informações privadas desnecessárias não são expostas. |
| CT-AGE-016-08 | Notificação não pode ser enviada temporariamente: o sistema segue a política definida de repetição ou registro da falha. |

### Cenários de insucesso

| ID | Cenário e resultado esperado |
|---|---|
| CT-AGE-016-03 | Atualizar estado sem evento notificável previsto: nenhuma notificação indevida é produzida. |

> **Observação sobre concorrência:** os cenários CT-AGE-015-06 a CT-AGE-015-10 exigem testes de integração com transações e concorrência reais no banco de dados utilizado; testes unitários isolados não são suficientes.

## ** Próximo passo recomendado**

Agora você possui uma base de cenários para todas as histórias de usuário. O próximo passo é transformar esses cenários em uma especificação de testes executável e rastreável, antes de avançar para a implementação completa.

Minha recomendação é seguir esta sequência.

### **1** Fechar as regras de negócio pendentes

Defina os comportamentos que ainda estão em aberto, como confirmação de agendamento, cancelamento, pagamento, fuso horário, antecedência mínima e bloqueios. Sem isso, alguns testes não terão um resultado esperado inequívoco.

### **2** Criar a matriz de rastreabilidade

Relacione cada requisito, história de usuário e cenário de teste. Isso permitirá identificar histórias sem cobertura e cenários sem requisito correspondente.

### **3** Definir os níveis de teste

Separe os testes unitários para regras de negócio, os testes de integração para persistência e transações, os testes de API para contratos HTTP e os testes de concorrência para impedir dupla reserva.

### **4** Implementar por fatias verticais com TDD

Comece por uma funcionalidade completa, do teste à implementação, em vez de implementar todas as entidades primeiro. Um bom ponto de partida é criar um agendamento válido e impedir conflitos.

### **5** Automatizar a execução dos testes

Configure a execução no Maven e no pipeline de integração contínua. Os testes devem ser repetíveis e executados a cada alteração relevante.

### **Qual domínio implementar primeiro?**

Eu começaria por Disponibilidade e Agendamentos, desde que você estabeleça previamente os contratos mínimos de identidade, profissional e serviço.

A ordem sugerida é:

1. Identidade e autenticação mínimas.  
2. Perfil profissional e cadastro de serviços.  
3. Horários de atendimento e bloqueios.  
4. Consulta de disponibilidade.  
5. Criação de agendamento com proteção contra conflitos.  
6. Cancelamento e transições de estado.  
7. Integração com pagamentos e notificações.

Essa sequência cria as dependências necessárias sem exigir que todos os recursos de cada domínio estejam completos desde o início.

### **O primeiro artefato que eu criaria**

Uma matriz de rastreabilidade de testes, em Markdown ou planilha, com as colunas:

| Coluna | Finalidade |
| ----- | ----- |
| ID da história | Ex.: `US-AGE-006` |
| ID do cenário | Ex.: `CT-AGE-006-01` |
| Pré-condições | Estado necessário para executar o teste |
| Dados de entrada | Valores utilizados |
| Ação | Operação executada |
| Resultado esperado | Comportamento que deve ser verificado |
| Tipo de teste | Unitário, integração, API ou concorrência |
| Status | Pendente, implementado, aprovado ou reprovado |

Essa matriz será o elo entre os requisitos, os testes e o código. Depois, os cenários podem ser convertidos em testes JUnit, usando Mockito quando fizer sentido e Spring Boot Test para os testes de integração.

**Minha sugestão final**: antes de escrever dezenas de testes em código, formalize as regras pendentes e monte essa matriz. Isso evita escrever testes baseados em suposições e reduz retrabalho durante o desenvolvimento.

