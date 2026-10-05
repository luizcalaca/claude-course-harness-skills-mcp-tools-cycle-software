## Rotina Infantil: travar Observações e botão Enviar após envio (SPEC DRIVEN DEVELOPMENT)

Descrição geral e objetivos

Hoje, mesmo após o envio da Rotina Infantil ao responsável, o campo de Observações e o botão de Enviar continuam ativos (o botão vira "Reenviar"). O objetivo é travar a edição assim que o registro é enviado, impedindo retificação posterior.

O que ele resolve

Impede que o professor altere o campo "Observações" de um registro com status === 'sent' (frontend/app/src/pages/academics/DailyRoutine.tsx).

Desativa o botão Enviar/Reenviar (dailyRoutine.sendAria / dailyRoutine.resendAria) quando o registro já foi enviado, removendo o reenvio atualmente permitido.

O que ele não resolve

Não altera o comportamento dos toggles de Lanche, Cocô e Xixi (esses seguem via handleBulkMark/upsert normal — fora de escopo, exceto decisão abaixo).

Não cria um fluxo de "reabrir registro" pelo backoffice/coordenação — se for necessário no futuro, é um card à parte.

Não altera o contrato da API (sendDailyRoutineEntry / upsertDailyRoutineEntry em frontend/app/src/services/dailyRoutineApi.ts); é tratado como regra de UI (confirmar se o backend também deve rejeitar PUT em entry já sent).

Pre-conditions

Tela DailyRoutine.tsx já exibe a coluna "Situação" (dailyRoutine.status.sent = "Enviado") e o botão de envio por linha e em lote (dailyRoutine.bulk.sendAll).

Decisão de produto confirmada: todos os campos editáveis da linha (Observações, Lanche, Cocô, Xixi) ficam bloqueados após o envio, ou só Observações + botão Enviar? (ajustar escopo acima conforme resposta).

Post-conditions

Com entry.status === 'sent': textarea de Observações renderiza disabled/readOnly, e o botão Enviar fica desabilitado (sem mais a ação "Reenviar").

Enviar tudo (handleSendAll) continua afetando apenas linhas com status === 'draft' (comportamento já existente, sem regressão).

Chaves i18n atualizadas se necessário (ex.: tooltip explicando "Registro enviado — não é mais possível editar").

Plano de testes (BDD)

Cenário 1 (happy path — bloqueio após envio): marcar Lanche/Cocô/Xixi, preencher Observações, clicar Enviar em uma linha → Validação: campo Observações fica desabilitado e botão Enviar não é mais clicável/some; status exibe "Enviado".

Cenário 2 (envio em lote): usar "Enviar tudo" com múltiplas linhas em rascunho → Validação: todas as linhas enviadas ficam com Observações e botão travados; linhas que já estavam sent não são reprocessadas.

Cenário 3 (tentativa de burlar via API): com a linha travada na UI, tentar PUT direto em upsertDailyRoutineEntry para uma entry sent → Validação: confirmar com o time de backend se a API também deve responder 409/erro (ou se o bloqueio é só client-side por ora — registrar decisão).

Cenário 4 (regressão): registro ainda em rascunho (status === 'draft') → Validação: Observações e botão Enviar continuam editáveis normalmente, sem bloqueio indevido.

Critério de aceite (Definition of Done): teste automatizado em DailyRoutine.test.tsx cobrindo os cenários 1 e 4 (campo/botão desabilitados quando sent, habilitados quando draft); validação manual do cenário 2; decisão documentada sobre o cenário 3.
