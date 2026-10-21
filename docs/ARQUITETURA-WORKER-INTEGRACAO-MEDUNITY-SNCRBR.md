# Decisão arquitetural — Worker de integração MedUnity ↔ SNCRbr

**Status:** proposta aprovada para orientar o desenvolvimento; implementação ainda pendente de testes de contrato.

## Decisão

Criar um Worker dedicado à integração entre MedUnity e o portal independente SNCRbr, ou adaptar o Worker existente `medunity-sncr-api` somente depois de avaliar se suas responsabilidades podem ser separadas com segurança.

O Worker de integração deve coordenar a transferência mínima e autenticada do contexto de uma ficha-mãe, buscar dados do prescritor autenticado, e disponibilizar ao SNCRbr somente os campos necessários à preparação da receita e à chamada autorizada da API oficial.

## Fluxo proposto

1. O profissional entra no MedUnity e mantém sua sessão autenticada.
2. A página de origem pede ao backend MedUnity a criação de um contexto de integração de curta duração.
3. O backend valida a sessão, identifica o profissional e a ficha-mãe autorizada e cria uma referência opaca, de uso limitado, com expiração.
4. O navegador abre `https://sncrbr.delyone.com.br` com somente essa referência opaca; nunca coloca CPF, dados clínicos, JWT ou tokens SNCR na URL.
5. O Worker valida e consome a referência, verifica usuário e autorização, e obtém somente os dados permitidos.
6. O portal preenche os campos da receita para conferência do profissional.
7. O fluxo oficial de autenticação SNCR/Gov.br ocorre separadamente, conforme o manual vigente.
8. O Worker chama a API oficial pelo backend, valida a resposta e retorna ao portal apenas os campos necessários.
9. A receita só apresenta a numeração oficial quando ela tiver sido recebida e validada. A impressão depende também dos requisitos de assinatura e do modelo aplicável.

## Separação de responsabilidades

- **MedUnity:** autenticação de origem, autorização para acessar a ficha-mãe, fonte de dados do prescritor e contexto clínico.
- **Worker de integração:** troca segura de contexto, minimização de dados, validações, proteção de segredos, coordenação do callback e chamadas autorizadas ao serviço oficial.
- **SNCRbr:** interface independente para revisão da receita, estados de operação e impressão após validação.
- **API oficial SNCR/Gov.br:** autenticação e autorização oficiais, regras de negócio e eventual retorno da numeração.

## Dados mínimos

A lista final depende do contrato oficial por tipo de receita. Como candidatos já encontrados no cadastro do prescritor: nome completo, CPF quando exigido, tipo/número/UF do conselho. Dados do paciente e atendimento só devem ser transferidos quando o modelo da receita exigir e quando a origem na ficha-mãe estiver identificada. O campo de tipo A/B da interface não deve ser presumido equivalente aos códigos de receita da API.

## Worker novo ou existente?

Há um Worker existente chamado `medunity-sncr-api`, com configuração de homologação, callback e namespace KV de sessões. Seu código tem um bloqueio administrativo de teste. Antes de reaproveitá-lo, avaliar seu contrato e suas rotas; não remover o bloqueio de teste nem publicar emissão real por conveniência.

A recomendação é manter uma camada de integração lógica dedicada. Ela pode ser um Worker novo ou um módulo separado no Worker existente, desde que haja isolamento de responsabilidades, rotas e permissões, segredos independentes quando apropriado, e testes claros. A escolha física deve ser feita após comparar o estado implantado com o código versionado no GitHub.

## Requisitos de segurança obrigatórios

- Validar a sessão MedUnity no backend; nunca aceitar nome/CPF/CRM enviados pelo navegador como prova de identidade.
- Vincular a referência temporária ao usuário, ficha-mãe, finalidade e tipo de operação autorizado.
- Aplicar expiração curta, uso único, proteção contra replay e verificação de origem.
- Não compartilhar JWT MedUnity nem token oficial SNCR com o frontend.
- Não registrar tokens, credenciais, nem dados clínicos desnecessários em logs.
- Tratar chamadas de numeração como operações potencialmente não idempotentes; não repetir automaticamente em caso de timeout sem regra oficial segura.
- Testar somente no ambiente autorizado de homologação até a aprovação formal do fluxo.
- Nunca inventar ou gerar localmente uma numeração oficial.

## Testes de aceitação

1. Sem sessão MedUnity válida, a criação do contexto falha.
2. Um usuário não consegue recuperar a ficha-mãe de outro usuário.
3. A referência expira e não pode ser reutilizada.
4. A URL do portal não contém dados pessoais, clínicos ou tokens.
5. Somente os campos necessários são retornados ao SNCRbr.
6. O fluxo oficial SNCR/Gov.br continua independente.
7. Falha, timeout, resposta inválida ou ausência de número não habilitam impressão como se houvesse sucesso.
8. A operação é testada em homologação com dados fictícios/autorizados antes de uso clínico real.

## Próximas ações

1. Identificar a fonte real da ficha-mãe no MedUnity (banco, API, estado da aplicação ou rascunho local).
2. Fechar a matriz de campos com o manual oficial por tipo de receita.
3. Comparar o Worker implantado `medunity-sncr-api` com o código do repositório e escolher entre Worker novo e módulo isolado.
4. Implementar primeiro a troca segura de contexto; não iniciar pelo botão de autenticação oficial.
