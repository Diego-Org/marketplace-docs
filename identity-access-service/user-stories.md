# Domínio: Identidade e Acesso

**Responsabilidade**: cadastro, autenticação, controle de acesso e ciclo de vida da conta.

## US-IDA-001 — Criar uma conta

Como visitante, quero criar uma conta na plataforma, para acessar as funcionalidades disponíveis ao meu perfil.

### Critérios de aceitação

- Dado que os dados obrigatórios são válidos, quando o visitante solicitar o cadastro, então a conta deverá ser criada.
- Dado que um campo obrigatório esteja ausente, quando o cadastro for solicitado, então a operação deverá ser rejeitada com uma indicação do campo inválido.
- Dado que o e-mail já esteja associado a uma conta, quando um novo cadastro for solicitado com esse e-mail, então a duplicidade deverá ser impedida.
- Dado que a senha não atenda à política de segurança configurada, quando o cadastro for solicitado, então a conta não deverá ser criada.
- A senha não deverá ser armazenada em texto puro.

## US-IDA-002 — Autenticar na plataforma

Como usuário cadastrado, quero autenticar minha identidade, para acessar as funcionalidades permitidas à minha conta.

### Critérios de aceitação

- Dado que as credenciais estejam corretas e a conta esteja habilitada, quando o usuário tentar entrar, então a autenticação deverá ser bem-sucedida.
- Dado que o e-mail ou a senha estejam incorretos, quando o usuário tentar entrar, então a autenticação deverá ser recusada.
- Dado que a conta esteja bloqueada ou desativada, quando houver uma tentativa de autenticação, então o acesso deverá ser recusado.
- Uma autenticação bem-sucedida deverá produzir uma sessão ou credencial de acesso válida, conforme o mecanismo adotado.
- As respostas de erro não deverão revelar se uma senha ou outro dado secreto está correto.

## US-IDA-003 — Encerrar a sessão

Como usuário autenticado, quero encerrar minha sessão, para impedir que outra pessoa continue utilizando meu acesso naquele contexto.

### Critérios de aceitação

- Dado que exista uma sessão válida, quando o usuário solicitar o encerramento, então a sessão ou credencial deverá ser invalidada conforme a estratégia adotada.
- Dado que a sessão tenha sido invalidada, quando uma operação protegida for solicitada com ela, então o acesso deverá ser recusado.
- Encerrar uma sessão já encerrada não deverá reativá-la nem conceder acesso.

## US-IDA-004 — Recuperar o acesso à conta

Como usuário que não consegue acessar minha conta, quero iniciar a recuperação de acesso, para voltar a utilizar a plataforma com segurança.

### Critérios de aceitação

- O sistema deverá permitir iniciar o fluxo de recuperação utilizando o identificador previsto, como o e-mail cadastrado.
- A solicitação não deverá revelar desnecessariamente se a conta existe.
- O mecanismo de recuperação deverá utilizar um token ou procedimento seguro, com validade limitada.
- Um token inválido, expirado ou já utilizado não deverá permitir a redefinição.
- Após a redefinição bem-sucedida, a nova senha deverá obedecer à política configurada e ser armazenada de forma segura.

## US-IDA-005 — Alterar a senha

Como usuário autenticado, quero alterar minha senha, para manter minha conta protegida.

### Critérios de aceitação

- A alteração deverá ser concluída quando os dados exigidos forem válidos.
- Se a operação exigir confirmação da senha atual, uma senha atual incorreta deverá impedir a alteração.
- Uma nova senha que viole a política configurada deverá ser rejeitada.
- Após a alteração, a nova senha deverá permitir autenticação.
- A senha anterior não deverá continuar sendo aceita como credencial, salvo se a política de segurança explicitamente permitir isso.

## US-IDA-006 — Controlar permissões de acesso

Como responsável pela segurança da plataforma, quero restringir operações de acordo com os papéis e as permissões dos usuários, para proteger os recursos de cada domínio.

### Critérios de aceitação

- Um usuário autenticado deverá acessar apenas operações autorizadas ao seu papel.
- Um cliente não deverá executar operações exclusivas de profissionais ou administradores sem a devida autorização.
- Um profissional não deverá acessar dados privados de outros profissionais apenas por estar autenticado.
- Operações administrativas deverão exigir autorização específica.
- Uma requisição sem autenticação para um recurso protegido deverá ser recusada.
- Uma requisição autenticada, mas sem a permissão necessária, deverá ser recusada.

## US-IDA-007 — Desativar uma conta

Como usuário da plataforma, quero solicitar a desativação da minha conta, para deixar de utilizá-la.

### Critérios de aceitação

- Uma solicitação válida deverá alterar o estado da conta conforme a política definida.
- Uma conta desativada não deverá conseguir iniciar novas sessões.
- A desativação não deverá apagar indevidamente registros necessários para agendamentos, pagamentos, auditoria ou obrigações legais.
- O tratamento de agendamentos futuros associados à conta deverá seguir a política de cancelamento e desativação estabelecida.

**Observação**: os fluxos de verificação de e-mail, autenticação multifator, login social e recuperação por telefone podem ser histórias adicionais, caso façam parte do escopo escolhido.