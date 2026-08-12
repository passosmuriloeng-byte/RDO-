# RDO

Painel automático de textos de obra (RDO / e-mail diretoria / WhatsApp diretoria), gerado por uma
rotina agendada na nuvem (Claude Code) a partir
da planilha de avanço da obra — sem custo de API paga.

- `CLAUDE.md` — instruções da rotina automática (onde ler os dados, o que gerar, onde publicar).
- `.claude/skills/ajustar-textos-obra/SKILL.md` — regras de tom por formato/público.

O painel fica publicado como Artifact do claude.ai (URL registrada em `CLAUDE.md` após a primeira
execução da rotina). A rotina roda 1x por dia, 10h horário de Brasília.

Projeto irmão do painel original (`projeto-painel-obra/`), que usava geração via API paga da
Anthropic direto do navegador — abordagem descartada em favor deste modelo.
