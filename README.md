# SPIKE — Zelador de Iniciativas

> Monitora Zulip. Cobra prazos. Nunca decide.

## Quick Start

1. Configurar credenciais via Vaultwarden (Zulip API, Supabase)
2. Instalar cron: `ops/cron-watch.md`
3. SPIKE roda a cada 1 minuto, monitora topicos, cobra baseado em KPIs

## Estrutura

| Pasta | O que tem |
|-------|----------|
| `skills/scrum-master/` | Skill principal: matriz de cobranca, KPIs, escalacao |
| `ops/` | Cron watch, deploy, config |

## O que SPIKE faz

- Detecta topicos `[INICIATIVA]`, `[MONITOR]`, `[REVIEW]` no Zulip
- Cobra prazos baseado na matriz de KPIs do Initiative Lifecycle
- Escala progressivamente: lembrete → cobranca → escalacao → bloqueio
- Posta resumo diario 09h BRT
- Registra tudo no banco

## O que SPIKE NAO faz

- Nao decide. Nao pesquisa. Nao executa. Nao improvisa.

## Quem Opera

- **Agent:** SPIKE (Haiku)
- **Reporta a:** CEO Manoel
- **Infra:** CTO Ronald

## Ultima Atualizacao

2026-04-17
