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

## **Domínio: Identidade e Acesso**

### **US-IDA-001 — Criar uma conta**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-001-01 | Cadastro com todos os dados válidos: a conta é criada. |
| CT-IDA-001-02 | Cadastro sem um campo obrigatório: a operação é recusada e o campo é identificado. |
| CT-IDA-001-03 | Cadastro com e-mail já registrado: a duplicidade é impedida. |
| CT-IDA-001-04 | Cadastro com e-mail em formato inválido: os dados são rejeitados. |
| CT-IDA-001-05 | Cadastro com senha abaixo dos requisitos mínimos: a conta não é criada. |
| CT-IDA-001-06 | Cadastro com senha válida: a senha persistida não corresponde ao texto puro enviado. |
| CT-IDA-001-07 | Cadastro com espaços ou variações de maiúsculas no e-mail: a aplicação respeita a política de normalização definida. |
| CT-IDA-001-08 | Duas solicitações concorrentes de cadastro com o mesmo e-mail: apenas uma conta é criada. |

### **US-IDA-002 — Autenticar na plataforma**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-002-01 | Autenticação com credenciais válidas e conta habilitada: o acesso é concedido. |
| CT-IDA-002-02 | Autenticação com senha incorreta: o acesso é recusado. |
| CT-IDA-002-03 | Autenticação com e-mail inexistente: o acesso é recusado sem revelar informações sensíveis. |
| CT-IDA-002-04 | Autenticação de conta desativada: o acesso é recusado. |
| CT-IDA-002-05 | Autenticação de conta bloqueada: o acesso é recusado. |
| CT-IDA-002-06 | Autenticação bem-sucedida: a credencial ou sessão emitida é válida. |
| CT-IDA-002-07 | Uso de credencial inválida em recurso protegido: o acesso é recusado. |
| CT-IDA-002-08 | Tentativas repetidas de autenticação: o comportamento segue a política de limitação de tentativas, se adotada. |

### **US-IDA-003 — Encerrar a sessão**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-003-01 | Encerrar uma sessão válida: a sessão é invalidada. |
| CT-IDA-003-02 | Utilizar a sessão encerrada em uma operação protegida: o acesso é recusado. |
| CT-IDA-003-03 | Encerrar a mesma sessão novamente: a operação é tratada sem conceder acesso. |
| CT-IDA-003-04 | Encerrar sessão inexistente ou inválida: o sistema responde de forma consistente com o contrato definido. |
| CT-IDA-003-05 | Encerrar uma sessão: outras sessões do usuário permanecem válidas, caso a política seja de encerramento individual. |

### **US-IDA-004 — Recuperar o acesso à conta**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-004-01 | Solicitar recuperação para uma conta existente: o fluxo é iniciado. |
| CT-IDA-004-02 | Solicitar recuperação para um e-mail inexistente: a resposta não revela desnecessariamente a existência da conta. |
| CT-IDA-004-03 | Redefinir a senha com token válido: a nova senha é aceita. |
| CT-IDA-004-04 | Utilizar token expirado: a redefinição é recusada. |
| CT-IDA-004-05 | Utilizar token inválido ou adulterado: a redefinição é recusada. |
| CT-IDA-004-06 | Reutilizar token já consumido: a operação é recusada. |
| CT-IDA-004-07 | Redefinir senha para valor que viola a política: a operação é recusada. |
| CT-IDA-004-08 | Autenticar após redefinição: a nova senha funciona e a anterior deixa de funcionar. |

### **US-IDA-005 — Alterar a senha**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-005-01 | Alterar senha com os dados válidos exigidos: a alteração é persistida. |
| CT-IDA-005-02 | Informar senha atual incorreta, quando exigida: a alteração é recusada. |
| CT-IDA-005-03 | Informar nova senha fora da política: a alteração é recusada. |
| CT-IDA-005-04 | Confirmar a senha de forma divergente, quando houver campo de confirmação: a alteração é recusada. |
| CT-IDA-005-05 | Autenticar após alteração: a nova senha é aceita. |
| CT-IDA-005-06 | Tentar autenticar com a senha antiga: a autenticação é recusada, conforme a política definida. |

### **US-IDA-006 — Controlar permissões de acesso**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-006-01 | Acessar recurso protegido sem autenticação: o acesso é recusado. |
| CT-IDA-006-02 | Cliente acessar recurso exclusivo de administrador: o acesso é recusado. |
| CT-IDA-006-03 | Profissional acessar recurso exclusivo de outro papel: o acesso é recusado. |
| CT-IDA-006-04 | Administrador executar operação autorizada: a operação é permitida. |
| CT-IDA-006-05 | Cliente tentar consultar dados privados de outro cliente: o acesso é recusado. |
| CT-IDA-006-06 | Profissional tentar alterar o perfil de outro profissional: a operação é recusada. |
| CT-IDA-006-07 | Usuário autenticado sem permissão específica tentar uma operação: a autorização é recusada. |
| CT-IDA-006-08 | Alterar um identificador na requisição para acessar recurso alheio: a autorização continua sendo validada no servidor. |

### **US-IDA-007 — Desativar uma conta**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-IDA-007-01 | Desativar a própria conta conforme as regras: a conta muda para o estado previsto. |
| CT-IDA-007-02 | Conta desativada tentar autenticar: o acesso é recusado. |
| CT-IDA-007-03 | Usuário tentar desativar a conta de outra pessoa: a operação é recusada sem autorização. |
| CT-IDA-007-04 | Desativar conta com histórico de agendamentos: o histórico é preservado. |
| CT-IDA-007-05 | Desativar conta com agendamentos futuros: o sistema aplica a política definida para esses compromissos. |
| CT-IDA-007-06 | Repetir a solicitação de desativação: o sistema mantém um estado consistente, sem efeitos indevidos. |

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

