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

## **Domínio: Catálogo de Serviços**

### **US-CAT-001 — Consultar categorias de serviços**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-001-01 | Consultar categorias existentes e ativas: as categorias disponíveis são retornadas. |
| CT-CAT-001-02 | Existirem categorias inativas: elas não aparecem na consulta pública. |
| CT-CAT-001-03 | Não existirem categorias ativas: é retornada uma lista vazia. |
| CT-CAT-001-04 | Consultar categorias: a operação não altera o catálogo. |
| CT-CAT-001-05 | Tentar alterar categorias por meio de uma operação de consulta: nenhuma alteração é realizada. |

### **US-CAT-002 — Consultar serviços de uma categoria**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-002-01 | Consultar categoria com serviços ativos: são retornados os serviços associados. |
| CT-CAT-002-02 | Consultar categoria sem serviços ativos: é retornada uma lista vazia. |
| CT-CAT-002-03 | Consultar categoria inexistente: o sistema retorna o erro previsto. |
| CT-CAT-002-04 | Existirem serviços inativos na categoria: eles são excluídos das opções de contratação. |
| CT-CAT-002-05 | Consultar serviços: cada resultado pertence à categoria solicitada. |

### **US-CAT-003 — Cadastrar serviço oferecido**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-003-01 | Cadastrar serviço com nome, preço e duração válidos: o serviço é criado. |
| CT-CAT-003-02 | Cadastrar serviço sem nome ou outro campo obrigatório: a operação é recusada. |
| CT-CAT-003-03 | Cadastrar serviço com preço negativo: a operação é recusada. |
| CT-CAT-003-04 | Cadastrar serviço com duração zero ou negativa: a operação é recusada. |
| CT-CAT-003-05 | Cadastrar serviço com categoria inexistente ou inválida: a operação é recusada, caso a categoria seja obrigatória. |
| CT-CAT-003-06 | Cadastrar serviço sem autenticação: a operação é recusada. |
| CT-CAT-003-07 | Tentar cadastrar serviço em nome de outro profissional: a operação é recusada. |
| CT-CAT-003-08 | Consultar o serviço depois do cadastro: os dados aceitos estão persistidos e associados ao profissional correto. |

### **US-CAT-004 — Definir preço e duração de um serviço**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-004-01 | Alterar preço e duração para valores válidos: os novos valores são persistidos. |
| CT-CAT-004-02 | Definir preço negativo: a operação é recusada. |
| CT-CAT-004-03 | Definir duração inválida: a operação é recusada. |
| CT-CAT-004-04 | Alterar serviço de outro profissional: a operação é recusada. |
| CT-CAT-004-05 | Consultar serviço após alteração: os valores atuais correspondem aos novos valores. |
| CT-CAT-004-06 | Alterar preço de serviço já agendado: o preço registrado no agendamento existente permanece inalterado. |
| CT-CAT-004-07 | Alterar duração de serviço já agendado: a duração registrada no agendamento existente permanece inalterada. |
| CT-CAT-004-08 | Tentar alterar serviço inexistente: o sistema retorna o resultado de erro previsto. |

### **US-CAT-005 — Atualizar informações de um serviço**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-005-01 | Atualizar descrição e outros campos válidos: as alterações são persistidas. |
| CT-CAT-005-02 | Atualizar com dados inválidos: a operação é recusada. |
| CT-CAT-005-03 | Atualizar serviço de outro profissional: a operação é recusada. |
| CT-CAT-005-04 | Atualizar serviço inexistente: o sistema retorna o erro previsto. |
| CT-CAT-005-05 | Alterar apenas um campo: os demais permanecem inalterados conforme o contrato da operação. |
| CT-CAT-005-06 | Atualizar dados de um serviço já utilizado: os registros históricos não são alterados indevidamente. |

### **US-CAT-006 — Desativar um serviço**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-006-01 | Desativar serviço próprio: o estado passa a inativo. |
| CT-CAT-006-02 | Tentar desativar serviço de outro profissional: a operação é recusada. |
| CT-CAT-006-03 | Tentar contratar serviço inativo: a operação é recusada. |
| CT-CAT-006-04 | Desativar serviço com agendamentos futuros: os agendamentos existentes são preservados e tratados conforme a política definida. |
| CT-CAT-006-05 | Desativar serviço inexistente: o sistema retorna o erro previsto. |
| CT-CAT-006-06 | Repetir a desativação: o estado final permanece consistente e sem efeitos colaterais indevidos. |

### **US-CAT-007 — Consultar os serviços de um profissional**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-007-01 | Consultar profissional com serviços ativos: são retornados seus serviços disponíveis. |
| CT-CAT-007-02 | Consultar profissional sem serviços ativos: é retornada uma lista vazia. |
| CT-CAT-007-03 | Consultar profissional inexistente: o sistema retorna o erro previsto. |
| CT-CAT-007-04 | Existirem serviços inativos: eles não são oferecidos para novas contratações. |
| CT-CAT-007-05 | Consultar os serviços de um profissional: nenhum serviço de outro profissional aparece na lista. |
| CT-CAT-007-06 | Consultar os resultados: cada serviço apresenta identificação, preço e duração válidos. |
| CT-CAT-007-07 | Consultar perfil não publicado: o acesso aos serviços respeita as regras de visibilidade do profissional. |

### **US-CAT-008 — Oferecer variações ou combinações de serviços**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-CAT-008-01 | Cadastrar uma variação válida: a opção fica disponível para consulta. |
| CT-CAT-008-02 | Cadastrar variação sem preço ou duração obrigatórios: a operação é recusada. |
| CT-CAT-008-03 | Cadastrar combinação com componentes válidos: a combinação é persistida conforme o modelo adotado. |
| CT-CAT-008-04 | Cadastrar combinação com componente inexistente ou inativo: a operação é recusada, conforme a regra definida. |
| CT-CAT-008-05 | Consultar uma opção contratável: suas condições de preço e duração são apresentadas corretamente. |
| CT-CAT-008-06 | Profissionais diferentes oferecerem opções distintas: o sistema permite catálogos individuais sem exigir variações idênticas. |
| CT-CAT-008-07 | Alterar componentes de uma combinação já utilizada em agendamento: o histórico do agendamento permanece preservado. |

**Observação de modelagem**: se variações e combinações forem representadas de maneiras diferentes no sistema, seus testes também devem ser separados. Não é necessário implementar um mecanismo genérico complexo de combinações se o MVP puder começar com serviços independentes e opções simples.

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

