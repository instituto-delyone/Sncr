# Achado de integração — callback e frontend MedUnity (22/10/2026)

**Escopo:** comparação de leitura entre o manual SNCR v3 (setembro/2026), o Worker Cloudflare implantado e os arquivos versionados no MedUnity. Nenhum segredo foi revelado; nenhuma configuração foi alterada; nenhum fluxo de autenticação foi iniciado.

## Achados confirmados

1. **O callback implantado aponta para o MedUnity, não para o portal SNCRbr.** O binding `SNCR_CALLBACK_URL` está configurado como `https://medunity.delyone.com/`; `SNCR_FRONTEND_ORIGIN` está em `https://medunity.delyone.com`. A origem coincide, mas isso precisa ser uma decisão explícita de arquitetura, não uma suposição.
2. **O manual SNCR (3ª edição, setembro de 2026) descreve um fluxo específico.** Nas páginas 7–10: `GET /api/v1/auth/login` recebe `client_url` e `state`; o callback configurável recebe `code` e `state` do Keycloak; valida assinatura e expiração; valida o `nonce`; cria `session_token` de uso único com validade de 30 segundos; redireciona para `client_url` com `session_id`. A recuperação do token é feita por `GET /api/v1/auth/token?session_id=...` e o token é removido da sessão após uso único.
3. **Há duas abordagens de frontend no MedUnity.** `App/Js/sncr-auth.js` implementa um fluxo antigo que chama a API oficial diretamente do navegador e guarda `access_token` em `sessionStorage`. Já `App/Js/receituario-sncr.js` usa o Worker `medunity-sncr-api`, com chave administrativa de teste digitada pelo administrador e callback enviado ao Worker. Não devemos ativar as duas abordagens ao mesmo tempo nem tratar a versão antiga como validada.
4. **O callback precisa ser correlacionado com a solicitação iniciada.** O Worker verifica um `state` assinado e armazenado temporariamente, mas o teste real deve confirmar o comportamento ponta a ponta e a compatibilidade do contrato. A origem e o `client_url` exatos também precisam corresponder ao cadastro autorizado no serviço oficial.
5. **Não há evidência ainda de autenticação real concluída.** A existência de rotas, bindings e configuração de homologação não equivale a uma resposta de autenticação oficial validada.

## Decisão segura para o próximo passo

- Não alterar callback, domínio, segredo, KV ou ambiente automaticamente.
- Confirmar a URL de callback que está registrada/autorizada no SNCR; a configuração atual aponta para `https://medunity.delyone.com/`, enquanto o portal independente planejado usa o domínio `.com.br`.
- Escolher um único frontend responsável pelo callback.
- Manter a chave administrativa restrita ao teste e não a incorporar em HTML, JavaScript versionado, URL ou logs.
- Executar o smoke test apenas em homologação, sem solicitar numeração e sem dados de paciente.
- Depois do teste, documentar a evidência observada (status HTTP e estado genérico), sem registrar tokens ou identificadores de sessão.

## Referência ao manual fornecido

Manual da API SNCR, 3ª edição, setembro de 2026: páginas 6–10 (autenticação, callback, token de uso único e requisitos de segurança). O fluxo de numeração está documentado separadamente nas páginas 11–16 e permanece fora do escopo deste smoke test.
