---
name: make-scenario-tools
description: >
  Como ligar um card interativo de um Artifact do claude.ai a uma automação no Make.com (ex.:
  gravar numa planilha Google Sheets), usando o conector MCP do Make. Use sempre que for construir
  ou mexer num formulário dentro de um Artifact que precisa escrever dado externo (Sheets, Docs,
  CRM etc.) através da conta Make do usuário, ou quando outro agente/sessão precisar entender por
  que os cards de Ocorrências/Mão de obra/Equipamentos/Clima do painel RLJ2 funcionam do jeito que
  funcionam.
---

# Ligar um card de Artifact a uma automação no Make

Descoberto/validado em 11/08/2026 construindo o painel RLJ2 (`RDO--main`). Resume o caminho que
funciona de primeira e os becos sem saída que custaram tempo e operações do Make.

## O padrão que funciona: Make "Tool", não "Scenario"

Não crie um Scenario normal com trigger de webhook nem com `scenarios_set-interface`. Os dois
caminhos parecem certos mas **não entregam o dado** de `scenarios_run` até o módulo final —
gastam operações do Make sem funcionar. O que funciona:

1. **`tools_create`** (não `scenarios_create`) — cria um recurso "Tool" com UM módulo de ação (ex.:
   `google-sheets:addRow`) e uma lista de `inputs` tipados.
2. Dentro do `mapper` do módulo, referencie os inputs como **`{{var.input.<nomeDoInput>}}`** — essa
   é a sintaxe que funciona (nem `{{nomeDoInput}}`, nem `{{1.nomeDoInput}}`, nem
   `{{input.nomeDoInput}}` funcionam — todos "têm sucesso" sem erro mas o valor chega vazio).
3. **Não precisa `scenarios_activate`** — Tools rodam direto.
4. Para testar, chame **`scenarios_run`** passando o `id` da Tool como `scenarioId` e os valores em
   `data` (com `responsive: true` pra ver o resultado na hora). Depois de rodar, confirme lendo a
   planilha/destino de verdade — não confie só no `status: 1` da resposta.
5. Se algo no módulo precisar ser corrigido depois, use `tools_update` (não `scenarios_update` —
   aquele é só para Scenarios normais, e além disso **zera qualquer interface configurada** se você
   usar por engano num Scenario).

Exemplo mínimo (Google Sheets "Add a Row"):

```json
{
  "teamId": <teamId>,
  "name": "Nome da Tool",
  "description": "...",
  "inputs": [
    {"name": "campo1", "type": "text", "required": true, "description": "..."}
  ],
  "module": {
    "module": "google-sheets:addRow",
    "version": 2,
    "mapper": {
      "mode": "map",
      "spreadsheetId": "...",
      "sheetId": "NomeDaAba",
      "tableFirstRow": "A1:Z1",
      "values": {"0": "{{formatDate(now; \"DD/MM/YYYY\")}}", "1": "{{var.input.campo1}}"},
      "insertUnformatted": false,
      "valueInputOption": "USER_ENTERED",
      "insertDataOption": "INSERT_ROWS"
    },
    "parameters": {"__IMTCONN__": <connectionId>},
    "metadata": {"expect": []}
  }
}
```

## Gotchas do Google Sheets no Make

- `sheetId` **não é consistente entre módulos**: `addRow`, `updateRow`, `getSheetContent` esperam o
  **nome da aba** (string, ex. `"MaoDeObra"`); já `renameSheet` espera o **gid numérico** da aba.
  Pra achar o gid: `rpc_execute` com `rpcName: "rpcSheet"` e `data: {..., ids: "true"}`.
- `clearValuesFromRange`: o parâmetro `range` precisa vir **prefixado com o nome da aba**
  (`"NomeDaAba!A1:D10"`). Um range sem prefixo (`"A1:D10"`) não dá erro — ele silenciosamente limpa
  a **primeira aba da planilha** (índice 0), que pode não ser a que você queria. Isso já apagou
  dado real numa aba errada nesta sessão. Depois de qualquer clear/write numa planilha com várias
  abas, confira com `getSheetContent` antes de seguir.
- Criar uma planilha nova (`create_file` do conector Drive) sempre nomeia a aba padrão
  `"Untitled"`, não `"Sheet1"`.
- Para adicionar mais abas numa planilha já criada: módulo `google-sheets:addSheet`
  (parâmetro `properties.title`). Para renomear a aba padrão: `renameSheet` (usa gid, ver acima).

## Ligar isso a um Artifact

No Artifact (ver skill `artifact-capabilities` para o contrato completo):

1. Declare a capability ao publicar: `capabilities: {"mcp": {"servers": [{"server":
   "claude_ai_Make", "tools": ["scenarios_run"]}]}}`. Se o card também precisa **ler** algo ao vivo
   (ex.: uma lista de cadastro), adicione outro server, ex. `{"server": "claude_ai_Google_Drive",
   "tools": ["read_file_content"]}`, e use `watchTool` (não `callTool`) pra exibir dado que deve se
   manter atualizado.
2. **Publicar com `mcp` exige que o Artifact não esteja compartilhado publicamente** — se dá erro
   "artifacts that use connectors can't be shared publicly", peça pro usuário desligar o
   compartilhamento público no menu de compartilhar da página, e publique de novo.
3. No JS da página, resolva o nome real do conector em runtime via `window.claude.mcp.listTools()`
   — não hardcode um "server" adivinhado, porque o nome de exibição pode não bater com o segmento
   usado na declaração de capability. Procure, na lista de servers retornada, aquele cujos `tools`
   incluam o nome da tool que você vai chamar (ex. `"scenarios_run"`).
4. Chame com `window.claude.mcp.callTool(server, 'scenarios_run', {scenarioId: <id da Tool>, data:
   {...}, responsive: true})`. Trate os códigos de erro (`not_granted`, `server_not_connected`,
   `needs_reauth`, `tool_error` etc.) com mensagens específicas — nunca um banner genérico.
5. Antes de publicar qualquer chamada MCP, você (o agente) precisa ter observado pelo menos uma
   chamada real dessa tool nesta sessão (regra da skill `artifact-capabilities`) — não adivinhe o
   formato de entrada/saída.

## Quando travar

Se depois de tentar o padrão acima (Tool + `var.input`) ainda não funcionar, **pare de tentar por
tentativa e erro** — isso consome operações reais da conta Make do usuário. Peça pro usuário
colar a pergunta pra IA do próprio Make (Maia, dentro da interface do Make) — ela tem acesso à
documentação interna do Make que este ambiente não tem, e resolveu em uma tentativa um problema
que o tentativa-e-erro aqui não resolveu (webhook sem "Determine data structure" capturada).

## Exemplo real deste projeto

Ver `CLAUDE.md` (raiz do repo) para os IDs concretos das 4 Make Tools do painel RLJ2 (Ocorrências,
Mão de obra, Equipamento, Clima) e das planilhas que elas alimentam — todas construídas com este
padrão.
