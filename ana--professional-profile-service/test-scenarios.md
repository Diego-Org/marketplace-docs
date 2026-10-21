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

## **Domínio: Perfis e Profissionais**

### **US-PRO-001 — Criar perfil de cliente**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-001-01 | Criar perfil com dados válidos: o perfil é persistido e associado à conta correta. |
| CT-PRO-001-02 | Criar perfil sem campo obrigatório: a operação é recusada. |
| CT-PRO-001-03 | Criar perfil com campo em formato inválido: o dado é rejeitado. |
| CT-PRO-001-04 | Criar perfil com campos opcionais ausentes: o cadastro é permitido. |
| CT-PRO-001-05 | Tentar criar perfil para outra conta sem autorização: a operação é recusada. |
| CT-PRO-001-06 | Tentar criar um segundo perfil quando a regra permitir apenas um por conta: a duplicidade é impedida. |
| CT-PRO-001-07 | Consultar o perfil após criá-lo: os dados persistidos correspondem aos dados aceitos. |

### **US-PRO-002 — Criar perfil profissional**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-002-01 | Criar perfil profissional com dados válidos: o perfil é criado e associado à conta autenticada. |
| CT-PRO-002-02 | Criar perfil sem dados obrigatórios: a operação é recusada. |
| CT-PRO-002-03 | Criar perfil com dados inválidos: a validação impede a persistência. |
| CT-PRO-002-04 | Tentar criar perfil profissional sem autenticação: a operação é recusada. |
| CT-PRO-002-05 | Tentar associar o perfil a uma conta diferente da autenticada: a operação é recusada. |
| CT-PRO-002-06 | Criar perfil sem cadastrar serviços ou horários: o perfil é criado sem gerar ofertas ou disponibilidade automaticamente. |
| CT-PRO-002-07 | Solicitar publicação de perfil que não cumpre os requisitos mínimos: a publicação é recusada. |

### **US-PRO-003 — Atualizar perfil**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-003-01 | Atualizar campos permitidos com dados válidos: as alterações são persistidas. |
| CT-PRO-003-02 | Enviar dados inválidos: a atualização é recusada. |
| CT-PRO-003-03 | Atualizar apenas um campo: os campos não enviados permanecem inalterados, conforme o contrato da operação. |
| CT-PRO-003-04 | Enviar campos desconhecidos ou não editáveis: o sistema rejeita ou ignora os campos conforme o contrato definido, sem alterar dados protegidos. |
| CT-PRO-003-05 | Tentar atualizar perfil de outra pessoa: a operação é recusada. |
| CT-PRO-003-06 | Atualizar perfil inexistente: o sistema retorna o resultado de erro previsto. |
| CT-PRO-003-07 | Consultar o perfil após atualização: os novos valores estão persistidos. |

### **US-PRO-004 — Consultar perfil público de profissional**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-004-01 | Consultar perfil publicado e ativo: os dados públicos são apresentados. |
| CT-PRO-004-02 | Consultar perfil não publicado: o perfil não é apresentado publicamente como disponível. |
| CT-PRO-004-03 | Consultar perfil desativado: o sistema não o apresenta como opção para contratação. |
| CT-PRO-004-04 | Consultar perfil inexistente: o sistema retorna o resultado de erro previsto. |
| CT-PRO-004-05 | Consultar perfil público: senhas, tokens e outros dados privados não são expostos. |
| CT-PRO-004-06 | Consultar perfil sem serviços ativos: o perfil apresenta a situação de catálogo vazio, sem inventar serviços. |

### **US-PRO-005 — Buscar profissionais**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-005-01 | Buscar profissionais sem filtros: o sistema aplica o comportamento padrão definido. |
| CT-PRO-005-02 | Buscar por um critério válido: são retornados profissionais que correspondem ao critério. |
| CT-PRO-005-03 | Buscar por critério sem correspondências: é retornada uma lista vazia. |
| CT-PRO-005-04 | Combinar vários filtros: os resultados atendem à combinação prevista de critérios. |
| CT-PRO-005-05 | Buscar com parâmetros inválidos: os parâmetros são rejeitados ou tratados conforme o contrato. |
| CT-PRO-005-06 | Executar busca pública: perfis não publicados ou desativados são excluídos. |
| CT-PRO-005-07 | Inspecionar resultados: informações privadas não são incluídas. |
| CT-PRO-005-08 | Buscar com paginação, se suportada: as páginas não repetem ou omitem resultados por falha de paginação. |

### **US-PRO-006 — Filtrar profissionais por localização**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-006-01 | Filtrar por localização com correspondências: são retornados profissionais compatíveis. |
| CT-PRO-006-02 | Filtrar por localização sem correspondências: a lista fica vazia. |
| CT-PRO-006-03 | Buscar por região não atendida pelo profissional: ele não aparece nos resultados filtrados. |
| CT-PRO-006-04 | Profissional sem localização cadastrada: seu tratamento respeita a política definida. |
| CT-PRO-006-05 | Informar localização inválida: a requisição é validada de acordo com o formato adotado. |
| CT-PRO-006-06 | Profissional que atende em domicílio ou em mais de uma região: os resultados respeitam o modelo de área de atendimento adotado. |

### **US-PRO-007 — Gerenciar a visibilidade do perfil**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-007-01 | Publicar perfil que atende aos requisitos: ele fica visível publicamente. |
| CT-PRO-007-02 | Tentar publicar perfil incompleto: a publicação é recusada. |
| CT-PRO-007-03 | Despublicar perfil: ele deixa de aparecer nas buscas públicas. |
| CT-PRO-007-04 | Alterar visibilidade sem autorização: a operação é recusada. |
| CT-PRO-007-05 | Despublicar perfil com agendamentos futuros: os agendamentos são preservados e tratados conforme a política definida. |
| CT-PRO-007-06 | Alterar a visibilidade repetidamente: o estado final permanece consistente. |

### **US-PRO-008 — Consultar avaliações do profissional**

| ID | Cenário e resultado esperado |
| ----- | ----- |
| CT-PRO-008-01 | Consultar profissional com avaliações publicadas: as avaliações elegíveis são apresentadas. |
| CT-PRO-008-02 | Consultar profissional sem avaliações: o sistema informa que não há avaliações, sem atribuir uma nota fictícia. |
| CT-PRO-008-03 | Existirem avaliações não elegíveis para publicação: elas são excluídas da consulta pública. |
| CT-PRO-008-04 | Calcular nota agregada: o resultado corresponde às avaliações consideradas pela regra definida. |
| CT-PRO-008-05 | Profissional tentar alterar diretamente uma avaliação: a operação é recusada. |
| CT-PRO-008-06 | Consultar avaliações de perfil inexistente ou indisponível: o resultado respeita a política pública e o contrato de API. |

**Dependência**: os cenários de US-PRO-008 exigem um mecanismo de avaliações. Caso esse domínio ainda não esteja implementado, mantenha-os como testes futuros ou de contrato entre serviços.

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

