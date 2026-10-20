## **Prioridade recomendada para o MVP**

| Prioridade | Domínio | Histórias iniciais |
| ----- | ----- | ----- |
| P0 | Identidade e Acesso | US-IDA-001, US-IDA-002, US-IDA-006 |
| P0 | Perfis e Profissionais | US-PRO-001, US-PRO-002, US-PRO-004 |
| P0 | Catálogo de Serviços | US-CAT-001, US-CAT-003, US-CAT-004, US-CAT-007 |
| P0 | Disponibilidade e Agendamentos | US-AGE-001, US-AGE-003, US-AGE-005, US-AGE-006, US-AGE-007, US-AGE-010, US-AGE-015 |

As demais histórias podem ser implementadas em etapas posteriores, conforme o escopo definitivo. A prioridade é uma recomendação inicial, não uma decisão de negócio já confirmada.

## **Tipos de testes por domínio**

Identidade e Acesso

Testes unitários e de integração

* Credenciais válidas e inválidas.  
* Regras de senha e duplicidade de e-mail.  
* Autorização por papel e propriedade dos recursos.  
* Expiração e invalidação de credenciais.

Perfis e Profissionais

Testes unitários, integração e API

* Validação e atualização de dados.  
* Associação entre conta e perfil.  
* Visibilidade pública.  
* Busca e filtros.  
* Proteção de dados privados.

Catálogo de Serviços

Testes unitários, integração e API

* Preço e duração válidos.  
* Associação do serviço ao profissional.  
* Ativação e desativação.  
* Isolamento entre profissionais.  
* Preservação dos dados históricos.

Disponibilidade e Agendamentos

Testes unitários, integração, API e concorrência

* Cálculo de intervalos disponíveis.  
* Sobreposição de horários.  
* Bloqueios e duração dos serviços.  
* Transições de estado.  
* Cancelamentos e liberação de horários.  
* Duas requisições concorrentes para o mesmo horário.

## **Exemplo de decomposição de uma história em cenários**

Considere a história `US-AGE-006 — Criar um agendamento`.

| ID | Cenário | Resultado esperado |
| ----- | ----- | ----- |
| CT-AGE-006-01 | Criar agendamento em horário disponível | Agendamento persistido com estado inicial válido |
| CT-AGE-006-02 | Tentar reservar um horário já ocupado | Operação recusada por conflito |
| CT-AGE-006-03 | Tentar reservar um horário bloqueado | Operação recusada |
| CT-AGE-006-04 | Informar serviço inexistente ou inativo | Operação recusada |
| CT-AGE-006-05 | Informar duração inválida | Operação recusada |
| CT-AGE-006-06 | Tentar agendar sem autenticação | Operação recusada |
| CT-AGE-006-07 | Tentar agendar em nome de outro cliente | Operação recusada, salvo autorização prevista |
| CT-AGE-006-08 | Alterar o preço do serviço depois de um agendamento | O preço histórico do agendamento permanece inalterado |
| CT-AGE-006-09 | Enviar duas solicitações concorrentes para o mesmo horário | No máximo uma reserva incompatível é confirmada |

Cada cenário deverá especificar os dados de entrada, as pré-condições, a ação executada e as verificações esperadas. Os cenários de API também devem verificar o código HTTP e a estrutura da resposta, quando aplicável.

## **Premissas que precisam ser definidas antes de fechar os testes**

Algumas regras ainda precisam de decisão explícita para que os testes não imponham comportamentos arbitrários:

* Identidade: verificação de e-mail obrigatória? Quais são as regras de senha e de recuperação?  
* Perfis: quais campos são obrigatórios para clientes e profissionais? Quais requisitos permitem publicar um perfil?  
* Catálogo: qual é a política para alterações de preço, duração ou serviço quando já existem agendamentos futuros?  
* Disponibilidade: qual é o intervalo mínimo para agendar? Há pausas, buffers entre atendimentos ou antecedência mínima para cancelamento?  
* Agendamentos: a confirmação será automática ou manual? Qual será o fluxo quando o pagamento falhar ou ficar pendente?  
* Concorrência: qual mecanismo de persistência ou controle transacional garantirá que duas reservas incompatíveis não sejam confirmadas?  
* Notificações: quais eventos serão notificados, por quais canais e com qual política de repetição?

Minha recomendação: mantenha essas histórias como especificação funcional e crie um arquivo de cenários de teste para cada história priorizada. Depois, para cada cenário, escreva primeiro o teste que falha, implemente o mínimo necessário para fazê-lo passar e refatore sem alterar o comportamento esperado. Assim, as histórias se tornam uma ponte rastreável entre os requisitos do produto e a implementação em Java/Spring Boot.