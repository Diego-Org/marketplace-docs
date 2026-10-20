# Domínio: Perfis e Profissionais

**Responsabilidade**: dados públicos e profissionais, preferências de perfil e controle de visibilidade.

## US-PRO-001 — Criar perfil de cliente

Como cliente cadastrado, quero preencher meu perfil, para facilitar minhas interações e meus agendamentos na plataforma.

### Critérios de aceitação

- O cliente deverá conseguir informar os campos obrigatórios definidos para seu perfil.
- Dados inválidos deverão ser rejeitados com uma indicação clara do problema.
- O perfil deverá estar associado à conta autenticada que o criou.
- Um cliente não deverá conseguir criar ou modificar o perfil de outro usuário sem autorização.
- Os campos opcionais poderão permanecer vazios.

## US-PRO-002 — Criar perfil profissional

Como profissional de beleza, quero cadastrar meu perfil profissional, para apresentar meus serviços e ser encontrado por clientes.

### Critérios de aceitação

- O profissional deverá conseguir cadastrar os dados obrigatórios definidos para apresentação pública.
- O perfil deverá estar associado à conta autenticada do profissional.
- Dados obrigatórios ausentes ou inválidos deverão impedir a conclusão do cadastro.
- O cadastro do perfil não deverá, por si só, criar serviços ou horários disponíveis.
- A publicação do perfil deverá respeitar os requisitos mínimos de ativação definidos pela plataforma.

## US-PRO-003 — Atualizar perfil

Como cliente ou profissional, quero atualizar os dados do meu perfil, para manter minhas informações corretas.

### Critérios de aceitação

- O proprietário do perfil deverá conseguir atualizar os campos que tiver permissão para modificar.
- Campos inválidos deverão ser rejeitados.
- Uma atualização válida deverá persistir os novos valores.
- Campos que não forem enviados ou que não fizerem parte da alteração não deverão ser modificados indevidamente.
- Um usuário não deverá conseguir alterar informações privadas de outro perfil.

## US-PRO-004 — Consultar perfil público de profissional

Como cliente, quero consultar o perfil público de um profissional, para avaliar se ele atende às minhas necessidades.

### Critérios de aceitação

- Um perfil publicado e ativo deverá exibir as informações públicas previstas.
- Um perfil não publicado ou desativado não deverá aparecer como profissional disponível para contratação.
- Informações privadas, como credenciais de acesso e dados internos, não deverão ser expostas.
- O perfil deverá permitir identificar o profissional e consultar suas informações relevantes para a contratação.

## US-PRO-005 — Buscar profissionais

Como cliente, quero buscar profissionais por critérios disponíveis, para encontrar opções adequadas ao que procuro.

### Critérios de aceitação

- O sistema deverá retornar profissionais que correspondam aos filtros informados e estejam elegíveis para aparecer nos resultados.
- Uma busca sem resultados deverá retornar uma lista vazia, sem causar erro.
- A ausência de filtros deverá seguir o comportamento padrão definido para a busca.
- Profissionais desativados ou não publicados não deverão aparecer nos resultados públicos.
- A busca não deverá expor dados privados dos profissionais.

## US-PRO-006 — Filtrar profissionais por localização

Como cliente, quero filtrar profissionais por localização ou região atendida, para encontrar opções compatíveis com meu deslocamento.

### Critérios de aceitação

- O sistema deverá aplicar o critério de localização conforme os dados cadastrados e o tipo de atendimento oferecido.
- Profissionais fora dos critérios informados não deverão aparecer nos resultados filtrados.
- A busca deverá tratar adequadamente a ausência de localização cadastrada, conforme a regra definida.
- A resposta deverá permitir identificar a localização pública relevante do profissional.

## US-PRO-007 — Gerenciar a visibilidade do perfil

Como profissional, quero controlar se meu perfil está publicado ou indisponível para novas contratações, para administrar minha presença na plataforma.

### Critérios de aceitação

- Um profissional autorizado deverá conseguir alterar o estado de publicação permitido.
- Um perfil não publicado não deverá ser oferecido em buscas públicas.
- Alterar a visibilidade do perfil não deverá apagar automaticamente o histórico de agendamentos.
- O efeito sobre agendamentos futuros deverá seguir uma política explícita, sem cancelamentos silenciosos.

## US-PRO-008 — Consultar avaliações do profissional

Como cliente, quero consultar as avaliações disponíveis de um profissional, para tomar uma decisão de contratação mais informada.

### Critérios de aceitação

- O perfil deverá exibir apenas avaliações elegíveis para publicação.
- O sistema deverá apresentar a nota agregada de acordo com a regra de cálculo definida.
- Na ausência de avaliações, o sistema deverá apresentar um estado apropriado, sem inventar uma nota.
- O sistema não deverá permitir que o proprietário do perfil altere diretamente as avaliações dos clientes.

**Dependência entre domínios**: as avaliações são produzidas no fluxo de avaliações, mas sua consulta pode ser apresentada no perfil. Caso avaliações ainda não estejam no MVP, esta história deve ser postergada.
