# Plano de integração MedUnity ↔ SNCR

**Estado:** especificação de implementação; não habilitada para uso clínico.

## Fluxo pretendido

1. O profissional inicia a ação a partir do atendimento correto no MedUnity e escolhe Notificação de Receita A ou B.
2. O MedUnity cria uma sessão de integração curta, autenticada e de uso único no backend. Não incluir nome, CPF, diagnóstico, medicamentos ou outros dados clínicos em query strings, URLs, analytics ou logs.
3. O navegador é encaminhado ao fluxo de autenticação oficial descrito no manual do SNCR. A aplicação nunca pede, recebe ou armazena credenciais Gov.br.
4. O backend valida a sessão, a origem, o profissional e as permissões antes de fazer a chamada oficial correspondente ao tipo selecionado.
5. O backend valida estritamente o esquema e o significado da resposta oficial. Não gera números, não completa faixas e não presume disponibilidade. Em erro, saldo insuficiente ou resposta inesperada, interrompe o fluxo e informa o profissional.
6. Os dados permitidos retornam ao atendimento de origem por canal autenticado, com proteção contra replay. O médico revisa e confirma antes de qualquer finalização.
7. Registrar trilha de auditoria mínima, sem tokens ou dados clínicos desnecessários.

## Restrições importantes

- O manual informa que o fluxo de autenticação cria um `session_id` de uso único com validade curta (30 segundos) e que a obtenção do token deve respeitar o vínculo com a origem que iniciou o fluxo. Projetar e testar a arquitetura com essas restrições; não transferir tokens entre origens.
- A implantação de produção exige HTTPS e domínio brasileiro `.br` para o `client_url`, conforme o manual consultado.
- Tokens e segredos devem permanecer no servidor, em variáveis/secret manager, nunca em HTML estático, JavaScript do navegador, repositório ou logs.
- O frontend está hospedado em GitHub Pages, que serve conteúdo estático. Ele não pode, sozinho, guardar segredos nem implementar o backend seguro necessário.
- A página estática atual apenas seleciona A/B e comunica o estado da integração. Não autentica, não consulta endpoints, não reserva números e não emite receitas.
- Não misturar os fluxos de Notificação de Receita com o endpoint distinto de Receita Especial/Retenção. Confirmar no manual vigente o endpoint, parâmetros, limites e resposta aplicáveis a cada tipo antes de codificar chamadas.
- A receita precisa ser revisada pelo profissional; autenticação, por si só, não autoriza emissão automática.

## Contrato de integração a implementar

O backend deverá expor rotas próprias autenticadas para:
- criar/validar a sessão de prescrição a partir do MedUnity;
- iniciar e concluir a autenticação SNCR segundo o manual;
- consultar a numeração oficial para o tipo e os dados do profissional autorizados;
- devolver ao MedUnity apenas os campos permitidos e o estado da operação.

Antes da implementação das rotas, confirmar a URL base oficial do SNCR, o mapeamento exato dos tipos A/B, os campos obrigatórios e a política de CORS/origem. Não adivinhar endpoints ou tratar exemplos como produção.

## Critérios mínimos de aceitação

- Nenhuma informação de paciente ou token em URL, local storage, telemetria ou logs.
- Testes para sessão expirada/reutilizada, origem divergente, profissional não autorizado, saldo insuficiente, timeout, erro de rede, resposta malformada e tentativa de replay.
- Respostas do serviço oficial validadas no servidor e mensagens seguras para o usuário.
- Nenhuma numeração inventada ou gerada localmente.
- Revisão médica explícita antes da finalização.
- Testes em ambiente autorizado antes de habilitar produção.
