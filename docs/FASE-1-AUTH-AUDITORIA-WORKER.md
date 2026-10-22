# Auditoria inicial do Worker SNCR — 22/10/2026

**Escopo:** leitura do código e das configurações do Worker existente. Nenhuma configuração foi alterada e nenhum deploy foi executado.

## Resultado confirmado

- O Worker `medunity-sncr-api` existe na conta Cloudflare conectada.
- O Worker está configurado para `homologacao`.
- Os bindings necessários ao fluxo de teste estão presentes: `SNCR_CALLBACK_URL`, `SNCR_FRONTEND_ORIGIN`, `SNCR_SESSIONS`, `SNCR_STATE_HMAC_SECRET` e `SNCR_ADMIN_TEST_KEY`. Os valores secretos não foram lidos nem registrados.
- A origem do callback coincide com a origem de frontend configurada.
- O código implantado contém rotas `POST /auth/login` e `POST /auth/callback`, além de rotas de consulta de numeração. A existência das rotas não prova que o fluxo completo tenha passado em teste real.
- O código exige `X-MedUnity-Admin-Key` antes de permitir as rotas de integração; o bloqueio administrativo de teste está preservado.
- O portal `instituto-delyone/Sncr` permanece estático em GitHub Pages; a pasta `sncr/` não contém backend versionado.

## Verificações ainda necessárias

1. Comparar o script implantado com uma versão versionada e revisável; no momento, o código do Worker consultado não foi localizado como arquivo no repositório `Sncr`.
2. Testar o fluxo de redirecionamento real, inclusive o retorno de `state` e `session_id`, usando somente homologação e uma conta de teste autorizada.
3. Verificar que o callback recebido pelo portal transforma os parâmetros da URL em uma requisição de backend segura e não deixa tokens em histórico, analytics ou logs.
4. Confirmar que a chamada a `/api/v1/auth/token` corresponde exatamente ao contrato da edição vigente do manual SNCR e que o token é consumido uma única vez.
5. Testar state inválido, replay, callback expirado, origem divergente, timeout e falha do serviço.
6. Validar o estado do Worker e o retorno HTTP com dados não clínicos antes de declarar a autenticação concluída.

## Limite deste relatório

Esta auditoria é de leitura/configuração, não é um teste de penetração nem um teste de ponta a ponta. Não houve alteração de secrets, consulta de dados de paciente, chamada de emissão/numeração ou implantação. Não habilitar produção com base somente nestas verificações.
