---
name: ajustar-textos-obra
description: >
  Ajusta e reescreve relatos de rotina de obra (anotações soltas, linhas técnicas de planilha de
  avanço, ou texto corrido) em três formatos possíveis: texto para RDO (relatório diário de
  obra), e-mail, ou mensagem de WhatsApp, cada um adaptado ao público (diretoria ou
  projetistas). Use sempre que o usuário mencionar RDO, relatório diário de obra, diário de
  obra, avanço da obra, pedir para ajustar texto para diretoria ou projetistas, transformar em
  email, mandar mensagem sobre a obra, ou colar dados de avanço (números de itens, percentuais,
  datas) pedindo para virar texto. Também use quando o usuário colar linhas técnicas geradas por
  planilha (formato "Item, Dia X un, Acumulado Y de Z un, Avanço W%") pedindo para comunicar
  isso a alguém.
---

# Ajustar textos de rotina de obra

Transforma o relato bruto do dia (anotações soltas, linhas técnicas geradas por planilha de
avanço, ou texto já corrido) em um dos três formatos de saída que o usuário usa na rotina de
gerente de projeto / coordenador de obra.

## Passo 1 — Entender o que o usuário quer

O usuário pode colar:
- Anotações soltas / bullet points do dia
- Linhas técnicas geradas automaticamente (ex.: planilha de avanço), no formato
  `Item: Dia X un | Acum. Y de Z un | Avanço W%`
- Um texto já corrido, só precisando ajustar tom/formato

**Se o usuário não disser qual saída quer, PERGUNTE antes de gerar** — não gere as três versões
de uma vez por padrão. As opções são:

1. Texto para **RDO** (relatório diário de obra)
2. **E-mail** — e para quem: diretoria ou projetistas?
3. **WhatsApp** — e para quem: diretoria ou projetistas?

O usuário pode pedir mais de um de uma vez explicitamente ("manda pro RDO e pro whatsapp da
diretoria") — nesse caso gere só os pedidos.

## Passo 2 — Extrair a informação relevante do relato bruto

Antes de escrever, identifique do texto/dados brutos:
- Quais itens/serviços tiveram avanço no dia (e quais não tiveremram, se relevante)
- Números-chave: quantidade do dia, acumulado, total previsto, % de avanço por item e geral
- Qualquer intercorrência, atraso, risco ou decisão pendente mencionada
- Datas relevantes (data do RDO, prazo previsto)

Nunca invente números ou itens que não estejam no relato — se faltar contexto (ex.: motivo de um
atraso), pergunte ao usuário ou deixe o campo genérico, sem supor.

## Passo 3 — Escrever no formato e tom certos

### RDO (relatório diário de obra)
- Objetivo, técnico, terceira pessoa, sem saudação/despedida
- Um item por linha (ou por bloco curto), formato compatível para colar direto num campo de
  relatório (site/app de diário de obra)
- Mantém unidades e jargão técnico de engenharia/obra
- Foco: o que foi executado no dia, acumulado, % de avanço. Intercorrências relevantes em linha
  separada, de forma factual
- Sem opinião, sem tom comercial

Exemplo de entrada crua (linha gerada por planilha):
`Baldrame: Dia 030 m | Acum. 369 de 727 m | Avanço 51%`

Exemplo de saída RDO:
`Baldrame: executados 30 m no dia, totalizando 369 m de 727,38 m previstos (51% de avanço).`

### E-mail para diretoria
- Tom formal/executivo, direto ao ponto
- Abre com o resultado principal (avanço geral, % do prazo, se está dentro do cronograma)
- Detalhes técnicos resumidos — diretoria quer impacto (prazo, custo, decisão necessária), não
  jargão de execução
- Se houver atraso ou risco, destaque isso e, se souber, a ação/decisão necessária
- Saudação e fechamento profissionais, mas enxutos

### E-mail para projetistas
- Tom técnico, pode manter jargão de engenharia/obra e detalhes de execução
- Foco em itens que afetam compatibilização de projeto, divergências de campo, dúvidas técnicas
  que exigem retorno do projetista
- Saudação e fechamento profissionais, mais direto que o e-mail de diretoria

### WhatsApp para diretoria
- Curto, direto, só os números/fatos que importam para quem decide
- Tom profissional mas conversacional — sem saudação longa nem assinatura formal
- Sem jargão técnico desnecessário

### WhatsApp para projetistas
- Curto, direto, pode ser técnico
- Tom profissional-informal, sem saudação longa

## Passo 4 — Confirmar antes de finalizar (quando fizer sentido)

Se o relato bruto tiver ambiguidade (ex.: um número que não bate, ou uma intercorrência pouco
clara), pergunte antes de escrever a versão final em vez de supor.
