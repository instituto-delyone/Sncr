# Fase 1 — Smoke test de autenticação SNCR

**Estado:** preparação versionada; ainda não executada contra o serviço oficial.  
**Objetivo:** validar o fluxo de autenticação e callback sem emitir receitas nem consultar/reservar numerações.

## Estado confirmado no repositório

- `docs/index.html` é uma página estática GitHub Pages; permite selecionar A/B e informa que a integração ainda não está conectada.
- `sncr/` não contém implementação de backend neste momento.
- O repositório documenta uma arquitetura pretendida com backend separado e segredos mantidos no servidor.
- O Worker `medunity-api` do MedUnity possui um roteador Gov.br próprio, mas isso não prova que seja compatível com o protocolo específico de autenticação do SNCR. Não misturar os dois fluxos sem revisão.

## Escopo mínimo

1. Confirmar, a partir da documentação oficial vigente, as URLs e os parâmetros exatos de `auth/login`, callback e `auth/token`.
2. Definir a URL de callback HTTPS no domínio `.br` do portal.
3. Implementar endpoints de backend para iniciar o fluxo e processar o callback.
4. Validar a assinatura HMAC do `state`, o `nonce`, a origem/URL de retorno e a expiração conforme o manual.
5. Recuperar o token exclusivamente no backend, respeitando a validade curta e o uso único; nunca devolvê-lo ao navegador nem registrá-lo em logs.
6. Criar uma página de teste mínima que inicia o fluxo e exibe somente um estado genérico de sucesso/erro.
7. Testar primeiro em ambiente autorizado, sem dados de pacientes e sem chamar endpoints de numeração.
8. Só considerar concluída a fase quando o backend confirmar a sessão válida com uma resposta real do serviço oficial.

## Critérios de aceitação

- Callback ausente, inválido, repetido ou expirado é rejeitado.
- `state` inválido ou assinatura inválida não cria sessão.
- Tokens e credenciais não aparecem na URL, no HTML, no armazenamento do navegador ou nos logs.
- Erros externos são apresentados sem expor detalhes internos.
- Não há emissão, consulta ou reserva de numeração nesta fase.
- Nenhum deploy de produção é feito durante o teste.

## Bloqueios antes de escrever a chamada oficial

- Conferir a configuração atual do Worker `medunity-sncr-api` mencionada na documentação do projeto e compará-la ao código versionado/implantado.
- Confirmar a URL base oficial e os endpoints exatos a partir do manual/Swagger autorizado. Não inferir URLs de exemplos parciais ou capturas de tela.
- Confirmar que existe ambiente autorizado de homologação e obter acesso de teste por meio do processo oficial, se aplicável.

## Próximo passo

Revisar a configuração do Worker existente e fechar o contrato HTTP do SNCR. Só então implementar os handlers específicos, para evitar duplicação ou um fluxo de autenticação incorreto.
