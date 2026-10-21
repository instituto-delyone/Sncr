# SNCRbr — Portal de Integração do Instituto Delyone

> **Status do projeto:** em desenvolvimento. Este README descreve a finalidade e a arquitetura pretendida; não declara que a integração oficial, a emissão de receitas ou os requisitos regulatórios já foram validados em produção.

## Visão geral

O **SNCRbr** é um portal web próprio, desenvolvido e gerenciado pelo **Instituto Delyone de Medicina e Tecnologia**. Ele terá uma aplicação e um ciclo de evolução separados do portal institucional e do MedUnity, com foco na experiência de acesso e nos fluxos de integração relacionados a receituários controlados.

O nome **SNCRbr** é uma referência/homenagem à finalidade do projeto. O portal é independente e **não deve ser apresentado como produto oficial, mantido, endossado ou homologado pela Anvisa ou pelo Governo Federal**, salvo se houver autorização ou reconhecimento formal documentado.

O repositório GitHub é público e separado:
- Repositório: [instituto-delyone/Sncr](https://github.com/instituto-delyone/Sncr)
- Domínio planejado: [sncrbr.delyone.com.br](https://sncrbr.delyone.com.br)
- Caminho institucional de referência: [delyone.com.br/sncr/](https://www.delyone.com.br/sncr/)

O domínio e a publicação precisam estar configurados e verificados antes de serem considerados operacionais.

## Objetivos

- Disponibilizar um portal dedicado, com identidade e ciclo de implantação próprios.
- Integrar o portal ao MedUnity para reconhecer de forma segura o profissional autenticado, quando houver uma sessão válida.
- Permitir que o profissional inicie o fluxo de autenticação/autorização exigido pelo serviço oficial aplicável.
- Centralizar a interface de consulta, preparação e acompanhamento de documentos, conforme as capacidades efetivamente autorizadas pela API.
- Validar respostas e apresentar ao usuário apenas os dados necessários.
- Manter a aplicação desacoplada da landing page institucional e dos demais módulos.

## Arquitetura pretendida

1. **MedUnity** — o profissional escolhe a função ou o tipo de receituário pertinente.
2. **Portal SNCRbr** — a aplicação intermediária é aberta no domínio próprio. Ela identifica uma sessão MedUnity por um mecanismo seguro, ou solicita autenticação se necessário.
3. **Backend de integração** — um componente de servidor valida a identidade e as permissões do profissional, mantém segredos fora do navegador e conversa com os serviços oficiais usando os endpoints e credenciais autorizados.
4. **Serviço oficial** — a autenticação, consulta, autorização, numeração e/ou emissão só são executadas quando esses recursos estiverem disponíveis e permitidos no contrato oficial da API.
5. **Retorno seguro** — a aplicação valida a resposta, correlaciona-a à solicitação original e apresenta os dados pertinentes ao profissional, sem expor credenciais ou tokens sensíveis.

### Fluxo conceitual

`MedUnity → Portal SNCRbr → Backend seguro → Serviço oficial autorizado → validação da resposta → Portal SNCRbr`

O portal pode reconhecer que o usuário está autenticado no MedUnity, mas isso **não equivale** a uma autorização do serviço oficial. São autenticações e permissões distintas e devem ser tratadas separadamente.

## Integração segura com o MedUnity

A integração deverá usar um mecanismo verificável de identidade e sessão. Não se deve confiar apenas em nome, CPF, CRM ou identificadores recebidos em parâmetros de URL, nem aceitar como prova de identidade dados que o navegador possa alterar.

Requisitos esperados:

- validar a sessão com a API do MedUnity ou por um mecanismo de token de curta duração, assinado e com audiência/escopo restritos;
- validar identidade, situação e permissões do profissional no servidor;
- evitar colocar tokens, senhas ou dados pessoais sensíveis em URLs, logs ou armazenamento acessível indevidamente;
- validar origem, destino e parâmetros de retorno, usando URLs permitidas explicitamente;
- aplicar proteção contra CSRF, replay e redirecionamentos abertos quando pertinente;
- encerrar sessões e tratar expiração/revogação de forma segura.

A implementação concreta depende do contrato atual da API do MedUnity e deverá ser documentada antes da ativação em produção.

## Integração com o serviço oficial

Antes de habilitar a integração, confirmar por documentação oficial e credenciais autorizadas:

- base URL e endpoints corretos;
- ambiente de homologação e ambiente de produção;
- registro do cliente, credenciais e escopos necessários;
- fluxo oficial de autenticação e autorização;
- URLs de retorno/callback autorizadas e requisitos de origem;
- regras para consulta, numeração, emissão, cancelamento e auditoria, conforme aplicável;
- condições regulatórias e de habilitação do profissional e do estabelecimento.

O portal não deve presumir que qualquer endpoint, recurso de emissão ou numeração esteja disponível apenas porque existe um endpoint de autenticação. A implementação deve seguir o contrato oficial vigente.

## Segurança e privacidade

Princípios mínimos para implementação:

- **Segredos apenas no servidor:** nunca versionar chaves privadas, client secrets, senhas ou tokens de produção.
- **Menor privilégio:** solicitar apenas os escopos e permissões necessários.
- **Validação de entrada e saída:** validar os dados enviados e a estrutura, origem e correlação das respostas.
- **Proteção de dados:** minimizar dados pessoais, limitar retenção e controlar acesso a informações clínicas e identificadores profissionais.
- **Transporte seguro:** HTTPS válido para todos os domínios envolvidos.
- **Auditoria sem vazamento:** registrar eventos operacionais úteis sem registrar tokens, senhas ou payloads clínicos desnecessários.
- **Resiliência:** tratar timeout, indisponibilidade, respostas inesperadas, duplicidade e repetição de requisições.
- **Separação de ambientes:** usar credenciais e dados de teste em homologação, sem misturar com produção.

## Emissão de documentos e assinatura

A geração de um documento pelo portal não significa, por si só, que ele foi emitido, registrado ou assinado digitalmente pelo serviço oficial. A interface deve distinguir claramente:

- documento preparado/gerado localmente;
- solicitação enviada ao serviço;
- resposta confirmada pelo serviço;
- numeração ou registro efetivamente retornado;
- assinatura digital concluída, quando aplicável.

A numeração e o estado de uma receita só podem ser apresentados como confirmados após resposta válida e verificável do serviço responsável. O login no MedUnity não substitui a autenticação, autorização ou assinatura exigida por outros sistemas.

## Testes e critérios para liberar produção

Antes de declarar o portal pronto para uso, validar pelo menos:

- domínio, DNS, GitHub Pages e certificado HTTPS;
- login e expiração de sessão do MedUnity;
- usuário não autenticado, sem permissão e com cadastro inconsistente;
- fluxo de autenticação oficial e retorno/callback;
- callbacks inválidos, repetidos ou expirados;
- erros de rede, timeout, indisponibilidade e respostas malformadas;
- validação de identidade, permissões, numeração e estado das operações;
- proteção de segredos, dados pessoais e informações clínicas;
- testes em homologação antes de qualquer operação real;
- monitoramento, trilha de auditoria e procedimento de suporte.

**Até que esses testes sejam concluídos, o portal deve ser considerado em desenvolvimento, não certificado para emissão em produção.**

## Responsabilidades e transparência

O SNCRbr é uma aplicação do Instituto Delyone. A integração com sistemas externos depende das permissões, contratos e requisitos estabelecidos pelos respectivos responsáveis. A existência deste portal não implica vínculo institucional, homologação ou endosso do governo ou da Anvisa.

## Próximas etapas

1. Confirmar o modo de publicação do repositório e o domínio personalizado no GitHub Pages.
2. Corrigir e verificar DNS e HTTPS do domínio planejado.
3. Documentar o contrato de autenticação da API do MedUnity.
4. Confirmar a documentação oficial, os endpoints, credenciais, ambientes e callbacks do serviço SNCR.
5. Implementar a integração de servidor e a validação das respostas.
6. Executar testes automatizados e integrados em homologação.
7. Revisar segurança, privacidade e requisitos regulatórios antes da liberação.

---

**Instituto Delyone de Medicina e Tecnologia**  
Projeto SNCRbr — portal independente de integração, em desenvolvimento.
