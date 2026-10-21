# Fase 1 — Mapa de dados MedUnity → SNCRbr

**Status:** levantamento inicial baseado no código-fonte consultado em 2026-10-21.  
**Regra:** este documento é uma matriz de análise; não autoriza chamadas reais à API nem define campos de receita que o manual não exija.

## Objetivo

Identificar o menor conjunto de dados necessário para preparar a operação SNCR e preencher a receita, separando:

1. identidade e sessão MedUnity;
2. dados profissionais do prescritor;
3. dados do atendimento/paciente;
4. parâmetros da solicitação de numeração SNCR;
5. resposta oficial e estado da operação.

A sessão MedUnity e a autenticação SNCR/Gov.br são mecanismos distintos. Uma sessão MedUnity válida não deve ser tratada como prova de autenticação ou autorização no SNCR.

## Fontes do código examinadas

- `cloudflare/medunity-api/migrations/0001_auth.sql`: usuário, nome de usuário, perfil e estado ativo.
- `cloudflare/medunity-api/migrations/0002_prescritores.sql`: tabela de prescritores com `nome_completo`, `cpf`, `conselho_tipo`, `conselho_numero`, `conselho_uf`, especialidade, e-mail, telefone e estado ativo.
- `cloudflare/medunity-api/src/index.js`: autenticação por JWT Bearer e validação do usuário ativo; é necessário confirmar as rotas e permissões exatas antes de usar esses dados em outro serviço.
- `App/Data/canonical-fields.json`: dicionário clínico que inclui nome do paciente, data de nascimento, sexo biológico, CPF, CNS, nome da mãe e campos de atendimento. Esse dicionário serve ao motor de documentos e não comprova que todos esses dados sejam necessários ao SNCR.
- `App/Js/receituario-sncr.js`: interface de teste antiga que solicita conselho, UF, documento profissional, tipo de receita, quantidade e, para RCE/RET, CNPJ. O próprio código informa que depende de Worker e não é uma integração operacional de produção.
- `workers/sncr-api/src/index.js` e `workers/sncr-api/README.md`: adaptador Cloudflare de teste existente, com barreira administrativa temporária.
- `docs/SNCR_API_V3_MIGRATION.md`: resumo técnico baseado no Manual da API SNCR, 3ª edição (setembro de 2026), fornecido ao projeto.

## Matriz inicial

| Grupo | Campo | Fonte conhecida | Uso previsto | Situação |
|---|---|---|---|---|
| Sessão | Identificador de usuário MedUnity | JWT validado pelo backend MedUnity | Identificar a sessão de origem no backend | Exige mecanismo de troca/handshake seguro |
| Prescritor | Nome completo | `prescritores.nome_completo` | Exibição/identificação profissional quando necessário | Confirmar exigência no formulário oficial |
| Prescritor | Conselho | `prescritores.conselho_tipo` | Parâmetro `conselho` da API | Confirmar enumeração aceita pelo SNCR |
| Prescritor | Número de registro | `prescritores.conselho_numero` | Parâmetro `documento` da API | Confirmar formato e correspondência cadastral |
| Prescritor | UF do conselho | `prescritores.conselho_uf` | Parâmetro `uf` da API | Confirmar formato esperado |
| Prescritor | CPF | `prescritores.cpf` ou cadastro do usuário | Só para validação/identificação se exigido | Não enviar por padrão sem requisito documentado |
| Receita | Tipo de operação | Seleção A/B existente no `docs/index.html` | Mapear para tipo/endpoint oficial | A seleção A/B da interface ainda não mapeia todos os tipos da API |
| Receita | Quantidade | Campo da operação, se aplicável | Parâmetro `quantidade` do endpoint de notificação | Manual resumido no projeto indica inteiro de 10 a 50; validar contrato exato |
| Receita | CNPJ da plataforma | Cadastro institucional configurado no backend, quando aplicável | Parâmetro `cnpj` para RCE/RET | Não solicitar ou confiar em valor arbitrário do navegador; confirmar com a autoridade |
| Paciente | Nome | Ficha-mãe/atendimento, se disponível | Preencher a receita, caso o modelo oficial exija | Ainda não identificado um contrato de dados da ficha-mãe entre páginas |
| Paciente | Data de nascimento/CPF/CNS/endereço | Dicionário canônico ou fonte clínica, conforme disponibilidade | Somente campos exigidos pelo modelo oficial da receita | Não transferir até mapear exigência e origem |
| Operação | Identificador opaco da operação | A definir no backend | Correlação e prevenção de repetição/replay | Pendente |
| Operação | Número/faixa devolvida | Resposta oficial SNCR | Exibir na receita depois de validar a resposta | Nunca gerar localmente nem assumir sucesso sem resposta |

## O que ainda não está comprovado

- O HTML atual do portal SNCRbr não recebe ainda, de forma implementada, um contexto autenticado de uma ficha-mãe do MedUnity.
- O esquema de sessão usado pelo usuário no MedUnity que abre o portal não está demonstrado como uma sessão reutilizável entre domínios.
- O cadastro de prescritor e a sessão do MedUnity não comprovam autorização SNCR; essa confirmação deve vir do fluxo/serviço oficial.
- O contrato do formulário final de receita (campos, layout, assinatura e requisitos de impressão) ainda precisa ser confrontado com o modelo oficial aplicável.
- O domínio de callback aceito/cadastrado pelo SNCR e as credenciais/ambiente de homologação devem ser confirmados.
- O Worker existente contém uma barreira administrativa de teste; essa chave não pode ser tratada como autenticação definitiva de profissionais.

## Contrato de transferência recomendado para investigar

1. MedUnity autentica o usuário normalmente.
2. O backend MedUnity valida a sessão e a autorização para iniciar a operação.
3. O backend emite uma referência opaca, de uso limitado e curta duração, vinculada ao usuário e ao contexto mínimo da operação.
4. O navegador abre SNCRbr com apenas essa referência não sensível; nenhum dado clínico, JWT MedUnity, token SNCR ou segredo deve ser colocado na URL.
5. O backend SNCRbr valida/consome a referência por canal autenticado e obtém apenas os campos autorizados.
6. SNCRbr inicia a autenticação oficial SNCR/Gov.br separadamente, conforme o manual.
7. Após a resposta oficial, o portal exibe apenas dados validados e habilita a impressão de acordo com os critérios de conclusão.

Este desenho é uma proposta a validar; exige contrato entre os dois backends, controles contra CSRF/replay, expiração, autorização por usuário e testes de segurança. Não se deve simplesmente compartilhar o JWT atual ou copiar todos os campos do navegador.

## Próxima tarefa técnica

Antes de conectar o botão `Autenticar com SNCR`, definir e documentar:

- qual rota autenticada do MedUnity pode fornecer dados do prescritor ao backend;
- como a ficha-mãe é identificada e quais campos ela realmente possui;
- qual mecanismo de contexto temporário será usado entre domínios;
- qual URL de callback e qual ambiente SNCR estão autorizados;
- qual resposta oficial será considerada suficiente para mostrar a numeração e liberar impressão.

## Critério de conclusão da Fase 1

A Fase 1 só estará concluída quando cada campo tiver origem confirmada, finalidade, requisito do manual/modelo, regra de validação e classificação de sensibilidade; campos sem necessidade comprovada devem ser excluídos da transferência.
