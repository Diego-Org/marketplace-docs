## 1. Domínio: Identidade e Acesso
Responsabilidade: cadastro, autenticação, controle de acesso e ciclo de vida da conta.
US-IDA-001 — Criar uma conta
Como visitante, quero criar uma conta na plataforma, para acessar as funcionalidades disponíveis ao meu perfil.

Critérios de aceitação
- Dado que os dados obrigatórios são válidos, quando o visitante solicitar o cadastro, então a conta deverá ser criada.
- Dado que um campo obrigatório esteja ausente, quando o cadastro for solicitado, então a operação deverá ser rejeitada com uma indicação do campo inválido.
- Dado que o e-mail já esteja associado a uma conta, quando um novo cadastro for solicitado com esse e-mail, então a duplicidade deverá ser impedida.
- Dado que a senha não atenda à política de segurança configurada, quando o cadastro for solicitado, então a conta não deverá ser criada.
- A senha não deverá ser armazenada em texto puro.
US-IDA-002 — Autenticar na plataforma
Como usuário cadastrado, quero autenticar minha identidade, para acessar as funcionalidades permitidas à minha conta.

Critérios de aceitação
- Dado que as credenciais estejam corretas e a conta esteja habilitada, quando o usuário tentar entrar, então a autenticação deverá ser bem-sucedida.
- Dado que o e-mail ou a senha estejam incorretos, quando o usuário tentar entrar, então a autenticação deverá ser recusada.
- Dado que a conta esteja bloqueada ou desativada, quando houver uma tentativa de autenticação, então o acesso deverá ser recusado.
- Uma autenticação bem-sucedida deverá produzir uma sessão ou credencial de acesso válida, conforme o mecanismo adotado.
- As respostas de erro não deverão revelar se uma senha ou outro dado secreto está correto.
US-IDA-003 — Encerrar a sessão
Como usuário autenticado, quero encerrar minha sessão, para impedir que outra pessoa continue utilizando meu acesso naquele contexto.

Critérios de aceitação
- Dado que exista uma sessão válida, quando o usuário solicitar o encerramento, então a sessão ou credencial deverá ser invalidada conforme a estratégia adotada.
- Dado que a sessão tenha sido invalidada, quando uma operação protegida for solicitada com ela, então o acesso deverá ser recusado.
- Encerrar uma sessão já encerrada não deverá reativá-la nem conceder acesso.
US-IDA-004 — Recuperar o acesso à conta
Como usuário que não consegue acessar minha conta, quero iniciar a recuperação de acesso, para voltar a utilizar a plataforma com segurança.

Critérios de aceitação
- O sistema deverá permitir iniciar o fluxo de recuperação utilizando o identificador previsto, como o e-mail cadastrado.
- A solicitação não deverá revelar desnecessariamente se a conta existe.
- O mecanismo de recuperação deverá utilizar um token ou procedimento seguro, com validade limitada.
- Um token inválido, expirado ou já utilizado não deverá permitir a redefinição.
- Após a redefinição bem-sucedida, a nova senha deverá obedecer à política configurada e ser armazenada de forma segura.
US-IDA-005 — Alterar a senha
Como usuário autenticado, quero alterar minha senha, para manter minha conta protegida.

Critérios de aceitação
- A alteração deverá ser concluída quando os dados exigidos forem válidos.
- Se a operação exigir confirmação da senha atual, uma senha atual incorreta deverá impedir a alteração.
- Uma nova senha que viole a política configurada deverá ser rejeitada.
- Após a alteração, a nova senha deverá permitir autenticação.
- A senha anterior não deverá continuar sendo aceita como credencial, salvo se a política de segurança explicitamente permitir isso.
US-IDA-006 — Controlar permissões de acesso
Como responsável pela segurança da plataforma, quero restringir operações de acordo com os papéis e as permissões dos usuários, para proteger os recursos de cada domínio.

Critérios de aceitação
- Um usuário autenticado deverá acessar apenas operações autorizadas ao seu papel.
- Um cliente não deverá executar operações exclusivas de profissionais ou administradores sem a devida autorização.
- Um profissional não deverá acessar dados privados de outros profissionais apenas por estar autenticado.
- Operações administrativas deverão exigir autorização específica.
- Uma requisição sem autenticação para um recurso protegido deverá ser recusada.
- Uma requisição autenticada, mas sem a permissão necessária, deverá ser recusada.
US-IDA-007 — Desativar uma conta
Como usuário da plataforma, quero solicitar a desativação da minha conta, para deixar de utilizá-la.

Critérios de aceitação
- Uma solicitação válida deverá alterar o estado da conta conforme a política definida.
- Uma conta desativada não deverá conseguir iniciar novas sessões.
- A desativação não deverá apagar indevidamente registros necessários para agendamentos, pagamentos, auditoria ou obrigações legais.
- O tratamento de agendamentos futuros associados à conta deverá seguir a política de cancelamento e desativação estabelecida.
Observação: os fluxos de verificação de e-mail, autenticação multifator, login social e recuperação por telefone podem ser histórias adicionais, caso façam parte do escopo escolhido.

## 2. Domínio: Perfis e Profissionais
Responsabilidade: dados públicos e profissionais, preferências de perfil e controle de visibilidade.
US-PRO-001 — Criar perfil de cliente
Como cliente cadastrado, quero preencher meu perfil, para facilitar minhas interações e meus agendamentos na plataforma.

Critérios de aceitação
- O cliente deverá conseguir informar os campos obrigatórios definidos para seu perfil.
- Dados inválidos deverão ser rejeitados com uma indicação clara do problema.
- O perfil deverá estar associado à conta autenticada que o criou.
- Um cliente não deverá conseguir criar ou modificar o perfil de outro usuário sem autorização.
- Os campos opcionais poderão permanecer vazios.
US-PRO-002 — Criar perfil profissional
Como profissional de beleza, quero cadastrar meu perfil profissional, para apresentar meus serviços e ser encontrado por clientes.

Critérios de aceitação
- O profissional deverá conseguir cadastrar os dados obrigatórios definidos para apresentação pública.
- O perfil deverá estar associado à conta autenticada do profissional.
- Dados obrigatórios ausentes ou inválidos deverão impedir a conclusão do cadastro.
- O cadastro do perfil não deverá, por si só, criar serviços ou horários disponíveis.
- A publicação do perfil deverá respeitar os requisitos mínimos de ativação definidos pela plataforma.
US-PRO-003 — Atualizar perfil
Como cliente ou profissional, quero atualizar os dados do meu perfil, para manter minhas informações corretas.

Critérios de aceitação
- O proprietário do perfil deverá conseguir atualizar os campos que tiver permissão para modificar.
- Campos inválidos deverão ser rejeitados.
- Uma atualização válida deverá persistir os novos valores.
- Campos que não forem enviados ou que não fizerem parte da alteração não deverão ser modificados indevidamente.
- Um usuário não deverá conseguir alterar informações privadas de outro perfil.
US-PRO-004 — Consultar perfil público de profissional
Como cliente, quero consultar o perfil público de um profissional, para avaliar se ele atende às minhas necessidades.

Critérios de aceitação
- Um perfil publicado e ativo deverá exibir as informações públicas previstas.
- Um perfil não publicado ou desativado não deverá aparecer como profissional disponível para contratação.
- Informações privadas, como credenciais de acesso e dados internos, não deverão ser expostas.
- O perfil deverá permitir identificar o profissional e consultar suas informações relevantes para a contratação.
US-PRO-005 — Buscar profissionais
Como cliente, quero buscar profissionais por critérios disponíveis, para encontrar opções adequadas ao que procuro.

Critérios de aceitação
- O sistema deverá retornar profissionais que correspondam aos filtros informados e estejam elegíveis para aparecer nos resultados.
- Uma busca sem resultados deverá retornar uma lista vazia, sem causar erro.
- A ausência de filtros deverá seguir o comportamento padrão definido para a busca.
- Profissionais desativados ou não publicados não deverão aparecer nos resultados públicos.
- A busca não deverá expor dados privados dos profissionais.
US-PRO-006 — Filtrar profissionais por localização
Como cliente, quero filtrar profissionais por localização ou região atendida, para encontrar opções compatíveis com meu deslocamento.

Critérios de aceitação
- O sistema deverá aplicar o critério de localização conforme os dados cadastrados e o tipo de atendimento oferecido.
- Profissionais fora dos critérios informados não deverão aparecer nos resultados filtrados.
- A busca deverá tratar adequadamente a ausência de localização cadastrada, conforme a regra definida.
- A resposta deverá permitir identificar a localização pública relevante do profissional.
US-PRO-007 — Gerenciar a visibilidade do perfil
Como profissional, quero controlar se meu perfil está publicado ou indisponível para novas contratações, para administrar minha presença na plataforma.

Critérios de aceitação
- Um profissional autorizado deverá conseguir alterar o estado de publicação permitido.
- Um perfil não publicado não deverá ser oferecido em buscas públicas.
- Alterar a visibilidade do perfil não deverá apagar automaticamente o histórico de agendamentos.
- O efeito sobre agendamentos futuros deverá seguir uma política explícita, sem cancelamentos silenciosos.
US-PRO-008 — Consultar avaliações do profissional
Como cliente, quero consultar as avaliações disponíveis de um profissional, para tomar uma decisão de contratação mais informada.

Critérios de aceitação
- O perfil deverá exibir apenas avaliações elegíveis para publicação.
- O sistema deverá apresentar a nota agregada de acordo com a regra de cálculo definida.
- Na ausência de avaliações, o sistema deverá apresentar um estado apropriado, sem inventar uma nota.
- O sistema não deverá permitir que o proprietário do perfil altere diretamente as avaliações dos clientes.
Dependência entre domínios: as avaliações são produzidas no fluxo de avaliações, mas sua consulta pode ser apresentada no perfil. Caso avaliações ainda não estejam no MVP, esta história deve ser postergada.

## 3. Domínio: Catálogo de Serviços
Responsabilidade: categorias, serviços oferecidos, preços, duração e opções de atendimento.
US-CAT-001 — Consultar categorias de serviços
Como cliente, quero consultar as categorias de serviços disponíveis, para encontrar o tipo de atendimento que procuro.

Critérios de aceitação
- O sistema deverá retornar as categorias ativas e disponíveis para consulta pública.
- Categorias inativas não deverão aparecer como opções para novas contratações.
- Se não houver categorias disponíveis, o sistema deverá retornar uma lista vazia.
- A consulta não deverá permitir que clientes alterem a estrutura do catálogo.
US-CAT-002 — Consultar serviços de uma categoria
Como cliente, quero consultar os serviços de uma categoria, para identificar quais atendimentos posso contratar.

Critérios de aceitação
- O sistema deverá retornar os serviços ativos pertencentes à categoria informada.
- Uma categoria inexistente deverá ser tratada conforme o padrão de erros da API.
- Serviços inativos não deverão ser oferecidos para novas contratações.
- A consulta deverá apresentar as informações públicas necessárias para identificar cada serviço.
US-CAT-003 — Cadastrar serviço oferecido
Como profissional, quero cadastrar um serviço que realizo, para disponibilizá-lo aos clientes.

Critérios de aceitação
- O profissional deverá conseguir informar os dados obrigatórios, incluindo nome, preço e duração.
- Preços negativos, durações inválidas e campos obrigatórios ausentes deverão ser rejeitados.
- O serviço deverá ficar associado ao profissional autenticado.
- Um profissional não deverá cadastrar serviços em nome de outro profissional sem autorização.
- O cadastro deverá respeitar as categorias e os tipos de serviço aceitos pela plataforma.
US-CAT-004 — Definir preço e duração de um serviço
Como profissional, quero definir o preço e a duração de cada serviço que ofereço, para informar corretamente as condições de contratação.

Critérios de aceitação
- O profissional deverá conseguir definir um preço não negativo e uma duração válida.
- Uma alteração válida deverá ser persistida e refletida nas consultas posteriores.
- Valores inválidos deverão ser rejeitados.
- Uma alteração no preço ou na duração não deverá modificar retroativamente agendamentos já confirmados.
- A política de aplicação das alterações a agendamentos futuros deverá ser explícita.
US-CAT-005 — Atualizar informações de um serviço
Como profissional, quero atualizar a descrição e as condições de um serviço, para manter minha oferta atualizada.

Critérios de aceitação
- O proprietário do serviço deverá conseguir atualizar os campos autorizados.
- Alterações com dados inválidos deverão ser rejeitadas.
- O serviço atualizado deverá continuar associado ao profissional correto.
- Um profissional não deverá modificar serviços pertencentes a outro profissional.
- Alterações não relacionadas não deverão ser sobrescritas indevidamente.
US-CAT-006 — Desativar um serviço
Como profissional, quero desativar um serviço que não ofereço mais, para impedir novas contratações desse atendimento.

Critérios de aceitação
- O proprietário deverá conseguir desativar seu serviço.
- Um serviço desativado não deverá estar disponível para novas contratações.
- A desativação não deverá apagar o histórico de agendamentos relacionados.
- Agendamentos existentes deverão ser tratados conforme a política de negócio definida.
- Uma nova tentativa de contratar um serviço desativado deverá ser recusada.
US-CAT-007 — Consultar os serviços de um profissional
Como cliente, quero consultar os serviços oferecidos por um profissional específico, para comparar preços e escolher o atendimento desejado.

Critérios de aceitação
- O sistema deverá retornar os serviços ativos do profissional informado.
- Os resultados deverão apresentar, no mínimo, identificação do serviço, preço e duração.
- Serviços inativos não deverão ser oferecidos para contratação.
- Um profissional sem serviços ativos deverá retornar uma lista vazia.
- A consulta deverá respeitar a visibilidade pública do perfil profissional.
US-CAT-008 — Oferecer variações ou combinações de serviços
Como profissional, quero cadastrar variações ou combinações de atendimentos, para oferecer opções que correspondam à forma como trabalho.

Critérios de aceitação
- O sistema deverá permitir representar as variações previstas pelo modelo de catálogo adotado.
- Cada opção contratável deverá possuir preço e duração definidos.
- Uma combinação deverá respeitar as regras de validação dos serviços que a compõem, quando aplicável.
- O cliente deverá conseguir identificar a opção escolhida antes de confirmar o agendamento.
- O sistema não deverá presumir que todos os profissionais oferecem exatamente as mesmas variações.
Observação de escopo: o catálogo deve permitir flexibilidade na forma como cada profissional organiza seus serviços. Categorias comuns podem ser compartilhadas pela plataforma, mas preços, durações e opções de atendimento pertencem à oferta individual do profissional.

## 4. Domínio: Disponibilidade e Agendamentos
Responsabilidade: horários de trabalho, bloqueios, consulta de disponibilidade, criação e ciclo de vida dos agendamentos.

Este domínio reúne dois comportamentos relacionados: o profissional define quando pode atender, e o cliente utiliza essa disponibilidade para solicitar um horário. A consulta de disponibilidade não garante, sozinha, que o horário continuará livre até a confirmação do agendamento.
US-AGE-001 — Definir horários de atendimento
Como profissional, quero cadastrar meus horários regulares de atendimento, para informar quando posso receber clientes.

Critérios de aceitação
- O profissional deverá conseguir cadastrar períodos de atendimento para os dias previstos.
- Os períodos deverão ter horários válidos, com início anterior ao término.
- Períodos inválidos ou incompatíveis com as regras definidas deverão ser rejeitados.
- Os horários deverão ficar associados ao profissional autenticado.
- Um profissional não deverá conseguir alterar a disponibilidade de outro profissional sem autorização.
US-AGE-002 — Atualizar horários de atendimento
Como profissional, quero alterar meus horários de atendimento, para adaptar minha agenda à minha rotina.

Critérios de aceitação
- O profissional deverá conseguir atualizar os períodos permitidos.
- A alteração deverá ser persistida e refletida nas consultas posteriores.
- Horários inválidos deverão ser rejeitados.
- A alteração não deverá apagar o histórico de agendamentos.
- A existência de agendamentos futuros incompatíveis com a nova disponibilidade deverá ser tratada por uma regra explícita, sem cancelamentos silenciosos.
US-AGE-003 — Bloquear períodos da agenda
Como profissional, quero bloquear um período da minha agenda, para impedir que sejam marcados atendimentos durante esse intervalo.

Critérios de aceitação
- O profissional deverá conseguir bloquear um intervalo válido.
- O bloqueio deverá impedir a oferta de horários que se sobreponham ao período bloqueado.
- Um bloqueio inválido deverá ser rejeitado.
- A criação de um bloqueio incompatível com agendamentos existentes deverá seguir uma política definida.
- Um profissional não deverá bloquear a agenda de outro profissional sem autorização.
US-AGE-004 — Remover um bloqueio da agenda
Como profissional, quero remover um bloqueio existente, para voltar a disponibilizar o período correspondente quando possível.

Critérios de aceitação
- O profissional deverá conseguir remover um bloqueio sob sua responsabilidade.
- Um bloqueio inexistente ou pertencente a outro profissional não deverá ser removido indevidamente.
- Após a remoção, o período poderá voltar a aparecer como disponível, desde que também respeite horários de atendimento, agendamentos e demais restrições.
- A remoção do bloqueio não deverá criar agendamentos automaticamente.
US-AGE-005 — Consultar horários disponíveis
Como cliente, quero consultar os horários disponíveis de um profissional para determinado serviço e período, para escolher quando realizar o atendimento.

Critérios de aceitação
- A consulta deverá considerar o profissional, o serviço e o período solicitados.
- O sistema deverá respeitar os horários de atendimento, os bloqueios, os agendamentos existentes e a duração do serviço.
- Horários no passado não deverão ser oferecidos para novos agendamentos.
- O resultado deverá excluir intervalos que não comportem integralmente a duração do serviço.
- Se não houver horários disponíveis, o sistema deverá retornar uma lista vazia.
- A consulta não deverá criar um agendamento nem reservar o horário automaticamente.
US-AGE-006 — Criar um agendamento
Como cliente, quero agendar um serviço com um profissional em um horário disponível, para garantir o atendimento desejado.

Critérios de aceitação
- O cliente deverá estar autenticado e fornecer os dados obrigatórios do agendamento.
- O profissional e o serviço deverão existir e estar elegíveis para contratação.
- O sistema deverá validar novamente a disponibilidade no momento da criação.
- O agendamento deverá ser recusado se houver conflito com outro agendamento ativo ou bloqueio aplicável.
- A validação deverá considerar a duração integral do atendimento, e não apenas o horário de início.
- Um agendamento válido deverá ser persistido com seu estado inicial definido pela regra de negócio.
- O agendamento deverá registrar o preço e a duração aplicáveis no momento da contratação, preservando o histórico caso a oferta seja alterada posteriormente.
- Uma tentativa de criar dois agendamentos incompatíveis para o mesmo profissional não deverá resultar em dupla reserva.
Ponto crítico para testes: consultar um horário e, em seguida, criar o agendamento são operações diferentes. A disponibilidade precisa ser validada novamente durante a criação. A implementação também deverá proteger a operação contra solicitações concorrentes; uma verificação simples anterior à gravação não é suficiente, por si só, para garantir isso.
US-AGE-007 — Consultar os detalhes de um agendamento
Como cliente ou profissional envolvido no atendimento, quero consultar os detalhes do agendamento, para verificar as condições do compromisso.

Critérios de aceitação
- O cliente deverá conseguir consultar os próprios agendamentos.
- O profissional deverá conseguir consultar os agendamentos associados à sua agenda.
- Um usuário não autorizado não deverá acessar os detalhes privados de um agendamento alheio.
- A consulta deverá apresentar os dados pertinentes, como serviço, horário, participantes, preço registrado e estado.
- A consulta não deverá modificar o agendamento.
US-AGE-008 — Listar meus agendamentos
Como cliente, quero listar meus agendamentos, para acompanhar os atendimentos marcados e o histórico.

Critérios de aceitação
- O sistema deverá retornar os agendamentos associados ao cliente autenticado.
- A listagem deverá permitir distinguir agendamentos futuros dos históricos, conforme os filtros disponibilizados.
- Uma conta sem agendamentos deverá receber uma lista vazia.
- O cliente não deverá conseguir listar os agendamentos privados de outro cliente.
- Os resultados deverão refletir os estados atuais dos agendamentos.
US-AGE-009 — Consultar a agenda profissional
Como profissional, quero consultar meus agendamentos, para organizar meus atendimentos.

Critérios de aceitação
- O sistema deverá retornar os agendamentos associados ao profissional autenticado.
- A consulta deverá permitir filtrar ou organizar os resultados por período, conforme os recursos previstos.
- Os agendamentos deverão apresentar informações suficientes para o profissional identificar os atendimentos.
- Um profissional não deverá consultar os dados privados da agenda de outro profissional.
- A agenda deverá distinguir compromissos ativos daqueles cancelados ou concluídos.
US-AGE-010 — Cancelar um agendamento como cliente
Como cliente, quero cancelar um agendamento meu, para informar que não poderei comparecer.

Critérios de aceitação
- O cliente deverá conseguir cancelar um agendamento que lhe pertença e esteja em estado cancelável.
- O cancelamento deverá ser recusado se o agendamento já estiver em um estado que não permita essa operação.
- As regras de antecedência e eventuais condições financeiras deverão ser respeitadas.
- O cancelamento deverá atualizar o estado do agendamento e preservar seu histórico.
- Após o cancelamento, o intervalo poderá voltar a ser disponibilizado, desde que nenhuma outra restrição o impeça.
- Repetir uma solicitação de cancelamento não deverá provocar efeitos duplicados.
US-AGE-011 — Cancelar um agendamento como profissional
Como profissional, quero cancelar um agendamento da minha agenda, para lidar com situações em que não poderei realizar o atendimento.

Critérios de aceitação
- O profissional deverá conseguir cancelar apenas agendamentos associados à sua agenda e que estejam em estado cancelável.
- Uma solicitação para cancelar um agendamento de outro profissional deverá ser recusada.
- O cancelamento deverá preservar o histórico e registrar o novo estado.
- O sistema deverá aplicar a política definida para notificações e efeitos financeiros.
- Um agendamento concluído ou já cancelado não deverá ser cancelado novamente como se ainda estivesse ativo.
US-AGE-012 — Confirmar um agendamento
Como profissional, quero confirmar um agendamento pendente, para indicar que estou ciente e aceito realizar o atendimento.

Critérios de aceitação
- O profissional deverá conseguir confirmar um agendamento pendente associado à sua agenda.
- Um agendamento inexistente, não autorizado ou em estado incompatível não deverá ser confirmado.
- A confirmação deverá alterar o estado de acordo com as transições permitidas.
- A confirmação não deverá duplicar o agendamento nem alterar indevidamente seu horário ou preço registrado.
- O comportamento deverá ser compatível com o fluxo de pagamento escolhido pela plataforma.
Premissa a confirmar: esta história só é necessária se o negócio exigir confirmação manual do profissional. Se a reserva for confirmada automaticamente após a validação e o pagamento, o fluxo será diferente.
US-AGE-013 — Registrar a conclusão do atendimento
Como profissional, quero marcar um agendamento como concluído, para manter o histórico atualizado e permitir os próximos passos do atendimento.

Critérios de aceitação
- O profissional deverá conseguir concluir um agendamento associado à sua agenda, desde que o estado atual permita a transição.
- Um agendamento cancelado não deverá ser marcado como concluído.
- A conclusão deverá ser persistida e refletida nas consultas posteriores.
- Repetir a operação não deverá gerar efeitos duplicados.
- O sistema deverá permitir identificar que o atendimento foi concluído para viabilizar processos dependentes, como avaliação, quando previstos.
US-AGE-014 — Registrar ausência do cliente
Como profissional, quero registrar que o cliente não compareceu, para manter o histórico fiel ao que ocorreu.

Critérios de aceitação
- O profissional deverá conseguir registrar a ausência apenas para um agendamento associado à sua agenda.
- O agendamento deverá estar em estado e período compatíveis com o registro de ausência.
- Um agendamento concluído ou cancelado não deverá ser alterado para ausência sem uma regra que permita essa transição.
- O estado atualizado deverá ficar disponível nas consultas posteriores.
- Eventuais efeitos sobre pagamentos, penalidades ou indicadores deverão respeitar regras próprias e explícitas.
Premissa a confirmar: é necessário definir se o registro de ausência será manual ou automático, e qual será o tempo de tolerância após o horário marcado.
US-AGE-015 — Impedir conflito entre agendamentos
Como responsável pela confiabilidade da plataforma, quero impedir reservas incompatíveis para um mesmo profissional, para evitar dupla marcação de horários.

Critérios de aceitação
- Dois agendamentos ativos do mesmo profissional não poderão ocupar intervalos sobrepostos, conforme a política de intervalos adotada.
- Agendamentos de profissionais diferentes poderão ocorrer simultaneamente, respeitadas as demais regras.
- Agendamentos cancelados não deverão bloquear horários, salvo regra explícita em contrário.
- Uma tentativa de agendamento conflitante deverá ser recusada de forma consistente.
- Solicitações concorrentes para o mesmo horário deverão ser tratadas de modo que não seja possível confirmar duas reservas incompatíveis.
- O comportamento deverá ser validado com o banco de dados e a estratégia de concorrência utilizados no ambiente de produção.
Esta história é especialmente importante para os testes de integração e concorrência. Um teste executado somente com H2 não comprova automaticamente o comportamento de concorrência do PostgreSQL.
US-AGE-016 — Notificar sobre alterações no agendamento
Como cliente ou profissional envolvido, quero receber notificações sobre eventos relevantes do agendamento, para acompanhar mudanças e não perder informações importantes.

Critérios de aceitação
- Os eventos previstos, como confirmação e cancelamento, deverão gerar as notificações definidas pela política da plataforma.
- A notificação deverá identificar o agendamento e o evento ocorrido sem expor informações indevidas.
- Falhas no envio não deverão desfazer indevidamente uma operação de negócio já persistida.
- Reprocessamentos não deverão gerar notificações duplicadas quando a infraestrutura oferecer garantia de idempotência.
- Os canais e o conteúdo das notificações deverão respeitar as preferências e os requisitos de privacidade aplicáveis.
Dependência: a implementação do envio pode pertencer a um domínio ou serviço de notificações separado. Neste domínio, o importante é produzir os eventos de negócio necessários.