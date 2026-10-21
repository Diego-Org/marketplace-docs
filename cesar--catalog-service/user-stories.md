
# Domínio: Catálogo de Serviços

**Responsabilidade**: categorias, serviços oferecidos, preços, duração e opções de atendimento.

## US-CAT-001 — Consultar categorias de serviços

Como cliente, quero consultar as categorias de serviços disponíveis, para encontrar o tipo de atendimento que procuro.

### Critérios de aceitação

- O sistema deverá retornar as categorias ativas e disponíveis para consulta pública.
- Categorias inativas não deverão aparecer como opções para novas contratações.
- Se não houver categorias disponíveis, o sistema deverá retornar uma lista vazia.
- A consulta não deverá permitir que clientes alterem a estrutura do catálogo.

## US-CAT-002 — Consultar serviços de uma categoria

Como cliente, quero consultar os serviços de uma categoria, para identificar quais atendimentos posso contratar.

### Critérios de aceitação

- O sistema deverá retornar os serviços ativos pertencentes à categoria informada.
- Uma categoria inexistente deverá ser tratada conforme o padrão de erros da API.
- Serviços inativos não deverão ser oferecidos para novas contratações.
- A consulta deverá apresentar as informações públicas necessárias para identificar cada serviço.

## US-CAT-003 — Cadastrar serviço oferecido

Como profissional, quero cadastrar um serviço que realizo, para disponibilizá-lo aos clientes.

### Critérios de aceitação

- O profissional deverá conseguir informar os dados obrigatórios, incluindo nome, preço e duração.
- Preços negativos, durações inválidas e campos obrigatórios ausentes deverão ser rejeitados.
- O serviço deverá ficar associado ao profissional autenticado.
- Um profissional não deverá cadastrar serviços em nome de outro profissional sem autorização.
- O cadastro deverá respeitar as categorias e os tipos de serviço aceitos pela plataforma.

## US-CAT-004 — Definir preço e duração de um serviço

Como profissional, quero definir o preço e a duração de cada serviço que ofereço, para informar corretamente as condições de contratação.

### Critérios de aceitação

- O profissional deverá conseguir definir um preço não negativo e uma duração válida.
- Uma alteração válida deverá ser persistida e refletida nas consultas posteriores.
- Valores inválidos deverão ser rejeitados.
- Uma alteração no preço ou na duração não deverá modificar retroativamente agendamentos já confirmados.
- A política de aplicação das alterações a agendamentos futuros deverá ser explícita.

## US-CAT-005 — Atualizar informações de um serviço

Como profissional, quero atualizar a descrição e as condições de um serviço, para manter minha oferta atualizada.

### Critérios de aceitação

- O proprietário do serviço deverá conseguir atualizar os campos autorizados.
- Alterações com dados inválidos deverão ser rejeitadas.
- O serviço atualizado deverá continuar associado ao profissional correto.
- Um profissional não deverá modificar serviços pertencentes a outro profissional.
- Alterações não relacionadas não deverão ser sobrescritas indevidamente.

## US-CAT-006 — Desativar um serviço

Como profissional, quero desativar um serviço que não ofereço mais, para impedir novas contratações desse atendimento.

### Critérios de aceitação

- O proprietário deverá conseguir desativar seu serviço.
- Um serviço desativado não deverá estar disponível para novas contratações.
- A desativação não deverá apagar o histórico de agendamentos relacionados.
- Agendamentos existentes deverão ser tratados conforme a política de negócio definida.
- Uma nova tentativa de contratar um serviço desativado deverá ser recusada.

## US-CAT-007 — Consultar os serviços de um profissional

Como cliente, quero consultar os serviços oferecidos por um profissional específico, para comparar preços e escolher o atendimento desejado.

### Critérios de aceitação

- O sistema deverá retornar os serviços ativos do profissional informado.
- Os resultados deverão apresentar, no mínimo, identificação do serviço, preço e duração.
- Serviços inativos não deverão ser oferecidos para contratação.
- Um profissional sem serviços ativos deverá retornar uma lista vazia.
- A consulta deverá respeitar a visibilidade pública do perfil profissional.

## US-CAT-008 — Oferecer variações ou combinações de serviços

Como profissional, quero cadastrar variações ou combinações de atendimentos, para oferecer opções que correspondam à forma como trabalho.

### Critérios de aceitação

- O sistema deverá permitir representar as variações previstas pelo modelo de catálogo adotado.
- Cada opção contratável deverá possuir preço e duração definidos.
- Uma combinação deverá respeitar as regras de validação dos serviços que a compõem, quando aplicável.
- O cliente deverá conseguir identificar a opção escolhida antes de confirmar o agendamento.
- O sistema não deverá presumir que todos os profissionais oferecem exatamente as mesmas variações.

**Observação de escopo**: o catálogo deve permitir flexibilidade na forma como cada profissional organiza seus serviços. Categorias comuns podem ser compartilhadas pela plataforma, mas preços, durações e opções de atendimento pertencem à oferta individual do profissional.