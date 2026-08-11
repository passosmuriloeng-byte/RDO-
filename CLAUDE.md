# CLAUDE.md — Rotina automática do RDO

Você roda como agente agendado (rotina em nuvem), uma vez por dia. Seu trabalho é ler os dados de
avanço da obra publicados no Google Sheets e atualizar o painel de textos (RDO, e-mails, WhatsApp)
que fica publicado como Artifact do claude.ai. Não peça confirmação para os passos abaixo — isso já
foi combinado com o usuário (Murilo).

## Onde ficam os dados
A planilha "UFV RLJ2 - INFRA CABOS" no Google Drive do usuário (fileId
`1cc8tCDKW4BKk-p9sTtSIde-IxQoZ4W4aYCjKnQbL1wY`). **Leia sempre pelo conector MCP Google Drive**
(`mcp__Google_Drive__read_file_content` com esse `fileId`) — **não** tente `curl`/`WebFetch` para
docs.google.com: o proxy de saída deste ambiente bloqueia esse domínio
(`EGRESS_BLOCKED`/403), então busca direta por HTTP sempre falha aqui. O conector MCP não passa
por esse bloqueio porque é um canal autorizado à parte, não uma requisição HTTP de saída.
Se o `fileId` mudar ou a leitura falhar, use `mcp__Google_Drive__search_files` com
`title contains 'RLJ2'` para reencontrar o arquivo antes de desistir.

A leitura retorna as 4 abas da planilha em texto/markdown:

- `RDO_Automatico` (bloco "RDO Automatico - Valas CFTV RLJ2"): snapshot do dia atual, já com texto
  pronto. Colunas: Item | Unid. | Total | Dia | Acumulado | Avanco Item | Peso | Texto RDO. A linha
  "Data do RDO (ultima data lancada)" traz a data do lançamento mais recente, e a linha "Avanco
  fisico geral acumulado" traz o % geral. Há também um bloco extra depois ("Caixa | Medida | Tipo")
  com lista de caixas de passagem — não faz parte do RDO, ignore esse bloco.

- `Lançamentos Infra Cabos`: uma linha por data, colunas por item (quantidade lançada naquele dia).
  Sem acumulado por item naquela data — só o incremento do dia. Use para contexto histórico se
  precisar, mas a fonte principal do dia atual é `RDO_Automatico`.

- `Curva_S` e `Resumo_Prazo`: dados extras (avanço planejado x real, prazo estimado) — use se forem
  úteis para enriquecer o e-mail da diretoria (ex.: "previsão de conclusão"), mas não são
  obrigatórios nos 5 textos.

Nunca use dados antigos guardados localmente — releia a planilha a cada execução, ela é a fonte de
verdade.

## O que fazer a cada execução
1. Leia a planilha via `mcp__Google_Drive__read_file_content` (fileId acima).
2. Monte o texto do RDO do dia usando a coluna "Texto RDO" do bloco `RDO_Automatico` (já vem
   pronta, um item por linha) e o avanço físico geral acumulado (linha "Avanco fisico geral
   acumulado").
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
5. Se a leitura da planilha falhar mesmo pelo conector MCP, não invente dados: publique o painel
   com um aviso claro do problema em vez de números incorretos.

## Artifact URL
https://claude.ai/code/artifact/e2f74d43-1458-428d-8b00-30e2f7a150e0

## Regras rígidas
- Nunca invente números, itens ou percentuais que não estejam na planilha.
- Nunca troque a fonte de dados por outra (sempre a planilha acima, sempre via conector MCP).
- Nunca tente `curl`/`WebFetch` para docs.google.com — já sabemos que falha neste ambiente.
