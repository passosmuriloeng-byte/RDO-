# RDO

Painel automático de textos de obra (RDO / e-mail diretoria / WhatsApp diretoria), gerado por uma
rotina agendada na nuvem (Claude Code) a partir
da planilha de avanço da obra — sem custo de API paga.

- `CLAUDE.md` — instruções da rotina automática (planilhas, fileIds, Make Tools, regras de geração,
  Artifact URL). É a referência completa da arquitetura atual: 5 planilhas de avanço físico, EAP
  geral, Ocorrências, Mão de obra/Equipamentos/Clima, e os cards interativos do painel.
- `.claude/skills/ajustar-textos-obra/SKILL.md` — regras de tom por formato/público (RDO, e-mail,
  WhatsApp), incluindo o modelo de texto completo da diretoria e as regras de linguagem que não
  comprometem o autor.
- `.claude/skills/make-scenario-tools/SKILL.md` — como os cards do painel gravam em planilhas do
  Google via Make (padrão técnico, reutilizável em outros projetos que precisem da mesma ligação
  Artifact → Make).

O painel fica publicado como Artifact do claude.ai (URL registrada em `CLAUDE.md` após a primeira
execução da rotina). A rotina roda 1x por dia, 10h horário de Brasília.

Projeto irmão do painel original (`projeto-painel-obra/`), que usava geração via API paga da
Anthropic direto do navegador — abordagem descartada em favor deste modelo.
