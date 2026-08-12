# CLAUDE.md — Rotina automática do RDO

Você roda como agente agendado (rotina em nuvem), uma vez por dia. Seu trabalho é ler os dados de
avanço da obra publicados no Google Sheets e atualizar o painel de textos (RDO, e-mails, WhatsApp)
que fica publicado como Artifact do claude.ai. Não peça confirmação para os passos abaixo — isso já
foi combinado com o usuário (Murilo).

## Onde ficam os dados

Cinco planilhas no Google Drive do usuário, uma por frente de obra, todas com a mesma estrutura de
abas (copiadas do mesmo modelo). **Leia sempre pelo conector MCP Google Drive**
(`mcp__Google_Drive__read_file_content` com o `fileId` de cada uma) — **não** tente `curl`/`WebFetch`
para docs.google.com: o proxy de saída deste ambiente bloqueia esse domínio (`EGRESS_BLOCKED`/403),
então busca direta por HTTP sempre falha aqui. O conector MCP não passa por esse bloqueio porque é
um canal autorizado à parte, não uma requisição HTTP de saída. Se algum `fileId` mudar ou a leitura
falhar, use `mcp__Google_Drive__search_files` (`title contains 'RLJ2'` ou `'SIM2'`) para reencontrar
o arquivo antes de desistir.

| EAP | Frente | Planilha | fileId |
| --- | --- | --- | --- |
| 3.1 | Cercamento externo — RLJ2 | UFV RLJ2 - 3.1 Cercamento Externo | `1KZ1EiaI8XQCW0PlNhl4-YiJffKRagqM5AFcdaxyllVk` |
| 3.3 | Cercamento externo — SIM2 | UFV SIM2 - 3.3 Cercamento Externo | `1-ws_0mFz8zfFaPSe2lQmaUFN6ftFSbfT0Q71Oc2k6r8` |
| 4.1.10 | Base da caixa d'água — RLJ2 | UFV RLJ2 - 4.1.10 Base Caixa d'Água | `1v7SOLTzeOb257ugrr_dbYammuQtUL9wXIWDvUdiOob4` |
| 5.1.4 | Infra CFTV, IoT e Auxiliar — RLJ2 | UFV RLJ2 - 5.1.4 Infra CFTV, Iot e Auxiliar | `1cc8tCDKW4BKk-p9sTtSIde-IxQoZ4W4aYCjKnQbL1wY` |
| 8.4.2 | Instalação hidro sanitária — RLJ2 | UFV RLJ2 - 8.4.2 Instalação Hidro Sanitária | `1jr24swBy9TTMp34PeB3Khhyp4zZKn0ylp1ig9fBUddY` |

Leia as 5 a cada execução. As abas internas de cada planilha ainda usam rótulos genéricos herdados
do modelo original ("RDO Automatico - Valas CFTV RLJ2", aba "Lançamentos Infra Cabos") mesmo nas
planilhas que não são de Infra Cabos (ex.: cercamento) — é só nome de aba, ignore e use o `Item` de
cada linha para saber do que se trata.

Cada planilha retorna as 4 abas em texto/markdown:

- `RDO_Automatico`: snapshot do dia atual, já com texto pronto. Colunas: Item | Unid. | Total | Dia
  | Acumulado | Avanco Item | Peso | Texto RDO. A linha "Data do RDO (ultima data lancada)" traz a
  data do lançamento mais recente. A linha "Avanco fisico geral acumulado" traz o % geral **quando a
  fórmula está funcionando** — nas planilhas 3.1 e 3.3 essa fórmula está quebrada (mostra 0,00%,
  porque a coluna Peso não foi estendida quando os itens foram trocados). Nesses dois casos **não
  calcule você mesmo uma média para substituir** — use o último % informado no snapshot da EAP (ver
  seção abaixo) e mostre os itens individuais como detalhe. Na 5.1.4 a fórmula funciona normalmente.
  Na 4.1.10 e 8.4.2 a aba ainda está com o conteúdo do modelo (itens de Infra Cabos, sem sentido para
  essas frentes) — tratar como "sem lançamento ainda" até o usuário preencher com os itens reais.

- `Lançamentos ...`: uma linha por data, colunas por item (quantidade lançada naquele dia). Use a
  linha mais recente para saber o "Dia" de cada item — se todas as colunas da última linha estiverem
  vazias/zero para uma frente, essa frente não teve avanço na data e entra na regra de exibição
  abaixo.

- `Curva_S` e `Resumo_Prazo`: dados extras (avanço planejado x real, prazo estimado) — use se forem
  úteis para enriquecer o e-mail da diretoria (ex.: "previsão de conclusão"), mas não são
  obrigatórios. Só existem com fórmula funcionando na 5.1.4.

Existe também "Cronograma RLJ2" (fileId `1ygOeiizHbAcJoIXNFCPXXpuvwqDqvWpnYAUC9dGoRd8`) — é uma
exportação do MS Project só com datas/durações planejadas, **sem % concluído e sem numeração EAP**.
Não serve para calcular avanço, só para contexto de prazo se precisar.

### Ocorrências e status de material

Dois arquivos adicionais no Google Drive do usuário:

- **Ocorrências** — Google Sheets "UFV RLJ2 - Ocorrências (Registro Automático)" (fileId
  `1BBMohJ-MraiXvyGsdGflQPbemXGxoxRloNKjoMqdklw`, aba/tab "Untitled"). Colunas: Data | Hora | Frente
  | Ocorrencia | Responsavel. Alimentada de duas formas: (1) o usuário digita direto na planilha, ou
  (2) o card "Registrar ocorrência" no topo do painel, que chama um Make Tool
  ("RLJ2 - Registrar Ocorrencia", ID `5918617`, conta Make do usuário) via `scenarios_run`, que
  grava a linha automaticamente com Data/Hora preenchidas por fórmula. Pegue só as linhas cuja Data
  bate com a "Data do RDO" do dia — linhas de dias anteriores já foram usadas em execuções passadas,
  não repita. As primeiras linhas (2 a 9) são teste e já devem ter sido apagadas pelo usuário; se
  ainda estiverem lá, ignore linhas com Frente/Ocorrencia/Responsavel vazios.
  O antigo Google Doc "UFV RLJ2 - Ocorrências" (fileId `1VJ3V-R7eQ77pE6sIIvdBOSN1rKNQmJliaQtlwGCEBcg`)
  foi **descontinuado** em 11/08/2026 em favor desta planilha — não leia mais dele.

- **Status de materiais** — Google Sheets "UFV RLJ2 - Status de Materiais" (fileId
  `1I2JIaTM-5T_44OkAgXX3u2Gm_QuKwNp3bbr2ca_vwDs`). Colunas: Data | Frente | Material | Em campo? |
  Responsavel | Observacao. Pegue as linhas com a data do dia. Ignore a linha de exemplo ("Arame
  farpado... apague esta linha") se ainda estiver lá.

Nenhum dos dois tem fonte alternativa — se a leitura falhar, trate como "sem ocorrências/sem
pendência de material relatada hoje" em vez de inventar conteúdo. O card de ocorrências do painel
depende da conexão Make do usuário estar ativa no claude.ai — se o card falhar para o usuário, ele
ainda pode lançar direto na planilha.

### Mão de obra, Equipamentos e Clima (uma obra só, não separado por frente)

Dois arquivos adicionais no Google Drive do usuário, criados em 11/08/2026:

- **Cadastro** — Google Sheets "UFV RLJ2 - Cadastro Mao de Obra e Equipamentos" (fileId
  `1nQDlEwCuETTialKbZ5dyIEvVwWUwYUHPyipe5mpImwg`). Lista viva de funcionários e equipamentos (colunas
  Tipo | Nome | FuncaoOuCategoria | MaoDeObraDiretaIndireta). O usuário edita esta planilha quando
  alguém entra/sai da obra ou chega equipamento novo — o card do painel lê direto dela (via
  `watchTool` no conector Google Drive), então nunca precisa mexer no painel por causa disso.

- **RDO Diário** — Google Sheets "UFV RLJ2 - RDO Diario Mao de Obra Equipamentos Clima" (fileId
  `1x6BaCgeBhC8bJGRvZyfYiQT99biPUyP-KSDccdBpoq8`), com 3 abas:
  - `MaoDeObra`: Data | Nome | Funcao | Presenca | Observacao. Presenca é um dos valores: Presente,
    Falta, Afastado, Atestado, Deslocando, Falta justificada, Férias, Folga, Licença, Treinamento,
    Viagem.
  - `Equipamentos`: Data | Nome | Quantidade | Observacao.
  - `Clima`: Data | Periodo | Condicao | Praticabilidade | IndicePluviometrico. Periodo é Manha,
    Tarde ou Noite; Condicao é Claro, Nublado ou Chuvoso; Praticabilidade é Praticavel ou
    Impraticavel.

  Alimentada pelos cards "Mão de obra" / "Equipamentos" / "Condição climática" no painel — cada card
  chama um Make Tool (conta Make do usuário) via `scenarios_run`, um por linha:
  - `RLJ2 - Registrar Mao de Obra` (ID `5918877`) — uma chamada por funcionário marcado.
  - `RLJ2 - Registrar Equipamento` (ID `5918878`) — uma chamada por equipamento preenchido.
  - `RLJ2 - Registrar Clima` (ID `5918895`) — uma chamada por período com condição selecionada.
  O usuário preenche isso pela manhã, antes das 10h, direto no painel (ou, se preferir, direto nas
  abas da planilha).

Leia as 3 abas do RDO Diário a cada execução, pegando só as linhas com a data do dia. Inclua um
resumo de mão de obra/equipamentos/clima no texto do RDO (ex.: contagem de presentes/faltas por
função, lista de equipamentos em campo, condição do dia) e mencione ausências relevantes
(Falta, Atestado, etc.) no e-mail da diretoria se for informação nova.

**Regra do "RDO não preenchido":** se, na data de hoje, nenhuma das 3 abas (MaoDeObra/Equipamentos/
Clima) tiver nenhuma linha com a data de hoje, mostre um aviso destacado no topo do painel: "RDO não
preenchido hoje". **Exceção:** se hoje for sábado ou domingo, não mostre esse aviso — é normal não
ter lançamento no fim de semana. Numa segunda-feira, verifique especificamente a data de hoje
(segunda), não a sexta anterior — o aviso só dispara se a própria segunda estiver vazia.

Nunca use dados antigos guardados localmente — releia as planilhas/documentos a cada execução, eles
são a fonte de verdade para os itens que acompanham.

## Status geral da EAP (card do topo do painel)

A EAP completa de RLJ2+SIM2 tem 122 atividades e **não tem fonte automática** — não existe planilha
nem conector que dê esse número ao vivo. O que se sabe (snapshot informado pelo usuário em
11/08/2026, RLJ2 e SIM2 tratadas com peso único porque compartilham cronograma):

- Total: 122 · Não iniciada: 107 · Em andamento: 5 · Concluída: 10 · Realizado geral: 10,46%
- As 5 atividades "em andamento": 3.1 (89%), 3.3 (78%), 4.1.10 (75%), 5.1.4 (14%), 8.4.2 (20%)
- RLJ2 e SIM2 mobilizaram em 02/03/2026 e seguem abaixo do ritmo planejado por atraso no envio de
  materiais pelo cliente (pendência em aberto, sem data de resolução conhecida)

Use esse snapshot como está, com a data de quando foi informado, no card "Status geral". Para os
itens que têm planilha viva com % funcionando (hoje só a 5.1.4), mostre o % da planilha como mais
atual, marcando de onde veio (ex.: "EAP 14% → 19,39% (planilha)"). Não recalcule os 122/107/5/10/10,46%
sozinho — se o usuário passar um snapshot novo em conversa, atualize este arquivo com o novo valor e
a nova data.

## O que fazer a cada execução
1. Leia as 5 planilhas da tabela acima via `mcp__Google_Drive__read_file_content`.
2. Para cada frente, veja se a última data de lançamento teve "Dia" > 0 em pelo menos um item.
   - Frentes com avanço no dia: mostrar no painel ordenadas por número da EAP crescente (3.1, 3.3,
     5.1.4, ...), e **gerar texto de envio** para elas nos 5 formatos.
   - Frentes sem avanço no dia (Dia = 0 em tudo, ou planilha ainda sem itens reais como 4.1.10/8.4.2
     hoje): mostrar no painel abaixo, marcadas "sem avanço hoje", e **não gerar texto de envio** para
     elas — só citar de passagem que não houve novidade.
3. Monte o texto do RDO do dia usando a coluna "Texto RDO" (ou os dados equivalentes) de cada frente
   com avanço, agrupado por frente. Se houver ocorrências do dia no Google Doc, adicione uma seção
   "Ocorrências" ao final do RDO, um relato por linha/parágrafo, factual, sem invenção.
4. A partir dos mesmos dados, gere os outros 2 formatos seguindo **exatamente** o modelo e as regras
   de tom em `.claude/skills/ajustar-textos-obra/SKILL.md` (atualizado em 11/08/2026 com um exemplo
   real do usuário — siga essa estrutura, não a versão curta/telegráfica antiga):
   - WhatsApp para diretoria: todos os itens de cada frente com avanço (não resumir), seção
     Ocorrências, seção Efetivo (contagem de "Presente" por Função na aba MaoDeObra do dia — omita a
     seção se não houver lançamento de mão de obra hoje), e seção de pagamentos/orçamentos pendentes
     **somente quando essa fonte estiver configurada** (ver nota ClickUp abaixo — por enquanto,
     omita essa seção).
   - E-mail para diretoria: mesma estrutura/conteúdo, em formato de e-mail (parágrafos, saudação,
     fechamento).
   Não gere e-mail nem WhatsApp para projetistas — foram removidos a pedido do usuário (2026-08-11).
   Nunca invente números, ocorrências, responsáveis, "Avanço diário" ou pendências de pagamento que
   não estejam nas planilhas/documentos/conectores configurados. Nunca escreva linguagem que assuma
   culpa ou prometa prazo não confirmado pelo usuário — ver regras em SKILL.md.

   **Pendências de pagamento/orçamento (ClickUp) — bloqueado em 11/08/2026:** o usuário pediu para
   incluir uma seção com links de tarefas de pagamento/orçamento pendentes de aprovação (exemplo:
   `https://app.clickup.com/t/3125581/<taskId>`), mas o conector ClickUp MCP disponível está
   conectado a um workspace diferente (space "OBRAS" > pasta "UFV PARACAMBI Pedências", workspace
   root `9007056000`) do workspace onde essas tarefas realmente ficam (team `3125581` — tentativa de
   leitura direta deu "Team not authorized"). Não invente essa seção nem tente adivinhar a pasta
   certa — só adicione a seção de pagamentos quando o usuário confirmar que reconectou o ClickUp ao
   workspace certo e informar qual pasta/lista usar; até lá, omita a seção inteira.
5. Publique/atualize o Artifact com: aviso de "RDO não preenchido" se aplicável (ver regra acima),
   cards "Registrar ocorrência" / "Mão de obra" / "Equipamentos" / "Condição climática" (mantenha os
   formulários e o script como estão — não remova nem regenere esses blocos, só ajuste os dados ao
   redor deles), card de status geral da EAP, as frentes com avanço (ordenadas), as frentes sem
   avanço (marcadas), e os 3 textos prontos (RDO, e-mail diretoria, WhatsApp diretoria) — incluindo o
   resumo de mão de obra/equipamentos/clima no RDO.
   - Se este arquivo já tiver uma "Artifact URL" registrada na seção abaixo, **atualize essa mesma
     URL** (não crie uma nova).
   - Ao chamar a ferramenta de Artifact, **omita o parâmetro `capabilities`** num redeploy normal —
     isso mantém a declaração de capability `mcp` (conectores Make e Google Drive) já registrada. Só
     passe `capabilities` de novo se precisar mudá-la explicitamente.
   - Se for a primeira execução (nenhuma URL registrada ainda), crie o Artifact e depois edite este
     arquivo (`CLAUDE.md`) preenchendo a seção "Artifact URL" com o link gerado, e faça commit +
     push dessa mudança no repositório — é assim que a próxima execução vai saber qual Artifact
     atualizar.
6. Se a leitura de alguma planilha falhar mesmo pelo conector MCP, não invente dados: publique o
   painel com um aviso claro do problema nessa frente específica, mantendo as demais normais.

## Artifact URL
https://claude.ai/code/artifact/e2f74d43-1458-428d-8b00-30e2f7a150e0

## Regras rígidas
- Nunca invente números, itens ou percentuais que não estejam nas planilhas ou no snapshot de EAP
  registrado neste arquivo.
- Nas planilhas 3.1 e 3.3, a fórmula de "Avanco fisico geral acumulado" está quebrada (0,00%) — nunca
  calcule uma média própria para substituir; use o % da EAP e mostre os itens como detalhe.
- Nunca troque a fonte de dados por outra (sempre as planilhas da tabela acima, sempre via conector
  MCP).
- Nunca tente `curl`/`WebFetch` para docs.google.com — já sabemos que falha neste ambiente.
- O conector MCP de Drive disponível só lê, copia e cria arquivos — não escreve células/fórmulas em
  planilha existente. Não tente "corrigir" ou preencher planilhas por conta própria; se notar um
  problema nos dados (ex.: número impossível), avise no painel/texto em vez de tentar consertar.
