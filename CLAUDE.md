# CLAUDE.md — Rotina automática do RDO

Você roda como agente agendado (rotina em nuvem), uma vez por dia. Seu trabalho é ler os dados de
avanço da obra publicados no Google Sheets e atualizar o painel de textos (RDO, e-mails, WhatsApp)
que fica publicado como Artifact do claude.ai. Não peça confirmação para os passos abaixo — isso já
foi combinado com o usuário (Murilo).

## Onde ficam os dados
Duas abas da planilha "Controle_Avanco_Cercamento" publicadas como CSV (Arquivo > Compartilhar >
Publicar na web):

- `RDO_Automatico` (snapshot do dia atual, já com texto pronto):
  `https://docs.google.com/spreadsheets/d/e/2PACX-1vTxqXWUZoNVP7HOsbLPenVSGpogGE3jOMeHk1nPU4dr_YfSxbLDEcS_qd91sapajDvegJWhmpIPxIv-/pub?gid=1456591249&single=true&output=csv`
  Colunas: Item | Unid. | Total | Dia | Acumulado | Avanco Item | Peso | Texto RDO. A célula B3
  (`Data do RDO (ultima data lancada)`) traz a data do lançamento mais recente. Há também um bloco
  extra no final do CSV ("Caixa | Medida | Tipo") que é uma lista de caixas de passagem — não faz
  parte do RDO, ignore esse bloco.

- `Lancamentos`:
  `https://docs.google.com/spreadsheets/d/e/2PACX-1vTxqXWUZoNVP7HOsbLPenVSGpogGE3jOMeHk1nPU4dr_YfSxbLDEcS_qd91sapajDvegJWhmpIPxIv-/pub?output=csv`
  Uma linha por data, colunas por item (quantidade lançada naquele dia). Sem acumulado por item
  naquela data — só o incremento do dia. Use para contexto histórico se precisar, mas a fonte
  principal do dia atual é `RDO_Automatico`.

Busque os dois CSVs com `curl` (ou WebFetch) a cada execução — nunca use dados antigos guardados
localmente, a planilha é a fonte de verdade.

## O que fazer a cada execução
1. Baixe os dois CSVs acima.
2. Monte o texto do RDO do dia usando a coluna "Texto RDO" de `RDO_Automatico` (já vem pronta, um
   item por linha) e o avanço físico geral acumulado (linha "Avanco fisico geral acumulado").
3. A partir desses mesmos dados, gere os outros 4 formatos seguindo as regras de tom em
   `.claude/skills/ajustar-textos-obra/SKILL.md`:
   - E-mail para diretoria
   - E-mail para projetistas
   - WhatsApp para diretoria
   - WhatsApp para projetistas
   Nunca invente números que não estejam nos CSVs.
4. Publique/atualize o Artifact com os 5 textos (RDO, e-mail diretoria, e-mail projetistas,
   WhatsApp diretoria, WhatsApp projetistas), organizados em cards com a data do RDO em destaque.
   - Se este arquivo já tiver uma "Artifact URL" registrada na seção abaixo, **atualize essa mesma
     URL** (não crie uma nova).
   - Se for a primeira execução (nenhuma URL registrada ainda), crie o Artifact e depois edite este
     arquivo (`CLAUDE.md`) preenchendo a seção "Artifact URL" com o link gerado, e faça commit +
     push dessa mudança no repositório — é assim que a próxima execução vai saber qual Artifact
     atualizar.
5. Se algum dos dois CSVs falhar ao carregar (erro de rede, HTML de erro em vez de CSV), não
   invente dados: publique o painel com um aviso claro do problema em vez de números incorretos.

## Artifact URL
https://claude.ai/code/artifact/e2f74d43-1458-428d-8b00-30e2f7a150e0

## Regras rígidas
- Nunca invente números, itens ou percentuais que não estejam nos dados dos CSVs.
- Nunca troque a fonte de dados por outra (sempre os 2 links acima).
