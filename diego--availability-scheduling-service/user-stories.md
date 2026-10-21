# Domínio: Disponibilidade e Agendamentos

> **Responsabilidade**: horários de trabalho, bloqueios, consulta de disponibilidade, criação e ciclo de vida dos agendamentos.

Este domínio reúne dois comportamentos relacionados: o profissional define quando pode atender, e o cliente utiliza essa disponibilidade para solicitar um horário. A consulta de disponibilidade não garante, sozinha, que o horário continuará livre até a confirmação do agendamento.

## US-AGE-001 — Definir horários de atendimento

Como profissional, quero cadastrar meus horários regulares de atendimento, para informar quando posso receber clientes.

### Critérios de aceitação

- O profissional deverá conseguir cadastrar períodos de atendimento para os dias previstos.
- Os períodos deverão ter horários válidos, com início anterior ao término.
- Períodos inválidos ou incompatíveis com as regras definidas deverão ser rejeitados.
- Os horários deverão ficar associados ao profissional autenticado.
- Um profissional não deverá conseguir alterar a disponibilidade de outro profissional sem autorização.

## US-AGE-002 — Atualizar horários de atendimento

Como profissional, quero alterar meus horários de atendimento, para adaptar minha agenda à minha rotina.

### Critérios de aceitação

- O profissional deverá conseguir atualizar os períodos permitidos.
- A alteração deverá ser persistida e refletida nas consultas posteriores.
- Horários inválidos deverão ser rejeitados.
- A alteração não deverá apagar o histórico de agendamentos.
- A existência de agendamentos futuros incompatíveis com a nova disponibilidade deverá ser tratada por uma regra explícita, sem cancelamentos silenciosos.

## US-AGE-003 — Bloquear períodos da agenda

Como profissional, quero bloquear um período da minha agenda, para impedir que sejam marcados atendimentos durante esse intervalo.

### Critérios de aceitação

- O profissional deverá conseguir bloquear um intervalo válido.
- O bloqueio deverá impedir a oferta de horários que se sobreponham ao período bloqueado.
- Um bloqueio inválido deverá ser rejeitado.
- A criação de um bloqueio incompatível com agendamentos existentes deverá seguir uma política definida.
- Um profissional não deverá bloquear a agenda de outro profissional sem autorização.

## US-AGE-004 — Remover um bloqueio da agenda

Como profissional, quero remover um bloqueio existente, para voltar a disponibilizar o período correspondente quando possível.

### Critérios de aceitação

- O profissional deverá conseguir remover um bloqueio sob sua responsabilidade.
- Um bloqueio inexistente ou pertencente a outro profissional não deverá ser removido indevidamente.
- Após a remoção, o período poderá voltar a aparecer como disponível, desde que também respeite horários de atendimento, agendamentos e demais restrições.
- A remoção do bloqueio não deverá criar agendamentos automaticamente.

## US-AGE-005 — Consultar horários disponíveis

Como cliente, quero consultar os horários disponíveis de um profissional para determinado serviço e período, para escolher quando realizar o atendimento.

### Critérios de aceitação

- A consulta deverá considerar o profissional, o serviço e o período solicitados.
- O sistema deverá respeitar os horários de atendimento, os bloqueios, os agendamentos existentes e a duração do serviço.
- Horários no passado não deverão ser oferecidos para novos agendamentos.
- O resultado deverá excluir intervalos que não comportem integralmente a duração do serviço.
- Se não houver horários disponíveis, o sistema deverá retornar uma lista vazia.
- A consulta não deverá criar um agendamento nem reservar o horário automaticamente.

## US-AGE-006 — Criar um agendamento

Como cliente, quero agendar um serviço com um profissional em um horário disponível, para garantir o atendimento desejado.

### Critérios de aceitação

- O cliente deverá estar autenticado e fornecer os dados obrigatórios do agendamento.
- O profissional e o serviço deverão existir e estar elegíveis para contratação.
- O sistema deverá validar novamente a disponibilidade no momento da criação.
- O agendamento deverá ser recusado se houver conflito com outro agendamento ativo ou bloqueio aplicável.
- A validação deverá considerar a duração integral do atendimento, e não apenas o horário de início.
- Um agendamento válido deverá ser persistido com seu estado inicial definido pela regra de negócio.
- O agendamento deverá registrar o preço e a duração aplicáveis no momento da contratação, preservando o histórico caso a oferta seja alterada posteriormente.
- Uma tentativa de criar dois agendamentos incompatíveis para o mesmo profissional não deverá resultar em dupla reserva.

**Ponto crítico para testes**: consultar um horário e, em seguida, criar o agendamento são operações diferentes. A disponibilidade precisa ser validada novamente durante a criação. A implementação também deverá proteger a operação contra solicitações concorrentes; uma verificação simples anterior à gravação não é suficiente, por si só, para garantir isso.

## US-AGE-007 — Consultar os detalhes de um agendamento

Como cliente ou profissional envolvido no atendimento, quero consultar os detalhes do agendamento, para verificar as condições do compromisso.

### Critérios de aceitação

- O cliente deverá conseguir consultar os próprios agendamentos.
- O profissional deverá conseguir consultar os agendamentos associados à sua agenda.
- Um usuário não autorizado não deverá acessar os detalhes privados de um agendamento alheio.
- A consulta deverá apresentar os dados pertinentes, como serviço, horário, participantes, preço registrado e estado.
- A consulta não deverá modificar o agendamento.

## US-AGE-008 — Listar meus agendamentos

Como cliente, quero listar meus agendamentos, para acompanhar os atendimentos marcados e o histórico.

### Critérios de aceitação

- O sistema deverá retornar os agendamentos associados ao cliente autenticado.
- A listagem deverá permitir distinguir agendamentos futuros dos históricos, conforme os filtros disponibilizados.
- Uma conta sem agendamentos deverá receber uma lista vazia.
- O cliente não deverá conseguir listar os agendamentos privados de outro cliente.
- Os resultados deverão refletir os estados atuais dos agendamentos.

## US-AGE-009 — Consultar a agenda profissional

Como profissional, quero consultar meus agendamentos, para organizar meus atendimentos.

### Critérios de aceitação

- O sistema deverá retornar os agendamentos associados ao profissional autenticado.
- A consulta deverá permitir filtrar ou organizar os resultados por período, conforme os recursos previstos.
- Os agendamentos deverão apresentar informações suficientes para o profissional identificar os atendimentos.
- Um profissional não deverá consultar os dados privados da agenda de outro profissional.
- A agenda deverá distinguir compromissos ativos daqueles cancelados ou concluídos.

## US-AGE-010 — Cancelar um agendamento como cliente

Como cliente, quero cancelar um agendamento meu, para informar que não poderei comparecer.

### Critérios de aceitação

- O cliente deverá conseguir cancelar um agendamento que lhe pertença e esteja em estado cancelável.
- O cancelamento deverá ser recusado se o agendamento já estiver em um estado que não permita essa operação.
- As regras de antecedência e eventuais condições financeiras deverão ser respeitadas.
- O cancelamento deverá atualizar o estado do agendamento e preservar seu histórico.
- Após o cancelamento, o intervalo poderá voltar a ser disponibilizado, desde que nenhuma outra restrição o impeça.
- Repetir uma solicitação de cancelamento não deverá provocar efeitos duplicados.

## US-AGE-011 — Cancelar um agendamento como profissional

Como profissional, quero cancelar um agendamento da minha agenda, para lidar com situações em que não poderei realizar o atendimento.

### Critérios de aceitação

- O profissional deverá conseguir cancelar apenas agendamentos associados à sua agenda e que estejam em estado cancelável.
- Uma solicitação para cancelar um agendamento de outro profissional deverá ser recusada.
- O cancelamento deverá preservar o histórico e registrar o novo estado.
- O sistema deverá aplicar a política definida para notificações e efeitos financeiros.
- Um agendamento concluído ou já cancelado não deverá ser cancelado novamente como se ainda estivesse ativo.

## US-AGE-012 — Confirmar um agendamento

Como profissional, quero confirmar um agendamento pendente, para indicar que estou ciente e aceito realizar o atendimento.

### Critérios de aceitação

- O profissional deverá conseguir confirmar um agendamento pendente associado à sua agenda.
- Um agendamento inexistente, não autorizado ou em estado incompatível não deverá ser confirmado.
- A confirmação deverá alterar o estado de acordo com as transições permitidas.
- A confirmação não deverá duplicar o agendamento nem alterar indevidamente seu horário ou preço registrado.
- O comportamento deverá ser compatível com o fluxo de pagamento escolhido pela plataforma.

**Premissa a confirmar**: esta história só é necessária se o negócio exigir confirmação manual do profissional. Se a reserva for confirmada automaticamente após a validação e o pagamento, o fluxo será diferente.

## US-AGE-013 — Registrar a conclusão do atendimento

Como profissional, quero marcar um agendamento como concluído, para manter o histórico atualizado e permitir os próximos passos do atendimento.

### Critérios de aceitação

- O profissional deverá conseguir concluir um agendamento associado à sua agenda, desde que o estado atual permita a transição.
- Um agendamento cancelado não deverá ser marcado como concluído.
- A conclusão deverá ser persistida e refletida nas consultas posteriores.
- Repetir a operação não deverá gerar efeitos duplicados.
- O sistema deverá permitir identificar que o atendimento foi concluído para viabilizar processos dependentes, como avaliação, quando previstos.

## US-AGE-014 — Registrar ausência do cliente

Como profissional, quero registrar que o cliente não compareceu, para manter o histórico fiel ao que ocorreu.

### Critérios de aceitação

- O profissional deverá conseguir registrar a ausência apenas para um agendamento associado à sua agenda.
- O agendamento deverá estar em estado e período compatíveis com o registro de ausência.
- Um agendamento concluído ou cancelado não deverá ser alterado para ausência sem uma regra que permita essa transição.
- O estado atualizado deverá ficar disponível nas consultas posteriores.
- Eventuais efeitos sobre pagamentos, penalidades ou indicadores deverão respeitar regras próprias e explícitas.

**Premissa a confirmar**: é necessário definir se o registro de ausência será manual ou automático, e qual será o tempo de tolerância após o horário marcado.

## US-AGE-015 — Impedir conflito entre agendamentos

Como responsável pela confiabilidade da plataforma, quero impedir reservas incompatíveis para um mesmo profissional, para evitar dupla marcação de horários.

### Critérios de aceitação

- Dois agendamentos ativos do mesmo profissional não poderão ocupar intervalos sobrepostos, conforme a política de intervalos adotada.
- Agendamentos de profissionais diferentes poderão ocorrer simultaneamente, respeitadas as demais regras.
- Agendamentos cancelados não deverão bloquear horários, salvo regra explícita em contrário.
- Uma tentativa de agendamento conflitante deverá ser recusada de forma consistente.
- Solicitações concorrentes para o mesmo horário deverão ser tratadas de modo que não seja possível confirmar duas reservas incompatíveis.
- O comportamento deverá ser validado com o banco de dados e a estratégia de concorrência utilizados no ambiente de produção.

Esta história é especialmente importante para os testes de integração e concorrência. Um teste executado somente com H2 não comprova automaticamente o comportamento de concorrência do PostgreSQL.

## US-AGE-016 — Notificar sobre alterações no agendamento

Como cliente ou profissional envolvido, quero receber notificações sobre eventos relevantes do agendamento, para acompanhar mudanças e não perder informações importantes.

### Critérios de aceitação

- Os eventos previstos, como confirmação e cancelamento, deverão gerar as notificações definidas pela política da plataforma.
- A notificação deverá identificar o agendamento e o evento ocorrido sem expor informações indevidas.
- Falhas no envio não deverão desfazer indevidamente uma operação de negócio já persistida.
- Reprocessamentos não deverão gerar notificações duplicadas quando a infraestrutura oferecer garantia de idempotência.
- Os canais e o conteúdo das notificações deverão respeitar as preferências e os requisitos de privacidade aplicáveis.

**Dependência**: a implementação do envio pode pertencer a um domínio ou serviço de notificações separado. Neste domínio, o importante é produzir os eventos de negócio necessários.