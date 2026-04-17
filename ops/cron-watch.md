# Cron Watch — Monitoramento Zulip

> SPIKE roda a cada 1 minuto. Le o Zulip. Detecta. Age.

## Fluxo

```
EVERY 1 MINUTE:
  1. Ler ultimo timestamp processado (banco ou arquivo local)
  2. GET /api/v1/messages?anchor=newest&num_after=0&num_before=50
     - Filtrar: timestamp > ultimo_processado
  3. Pra cada mensagem nova:
     a. Tem tag [INICIATIVA]? → registrar_iniciativa()
     b. Tem tag [MONITOR]? → checar_report_semanal()
     c. Tem tag [REVIEW]? → checar_decisao()
     d. Eh resposta a topico monitorado? → atualizar_participante()
  4. Checar iniciativas abertas:
     - Decisao pendente >4h? → cobrar()
     - Pesquisa pendente >48h? → cobrar()
     - Report semanal atrasado? → cobrar()
  5. Atualizar ultimo_processado
  6. Se 09h BRT → postar_resumo_diario()
```

## Implementacao

**Opcao A — Script shell + curl (minimo)**
```bash
#!/bin/bash
# spike-watch.sh — roda via cron a cada 1min

ZULIP_API="https://dis.abckx.com.br/api/v1"
LAST_ID_FILE="$HOME/.spike_last_id"

# Buscar mensagens novas
LAST_ID=$(cat "$LAST_ID_FILE" 2>/dev/null || echo "0")
MESSAGES=$(curl -s -u "$ZULIP_BOT_EMAIL:$ZULIP_API_KEY" \
  "$ZULIP_API/messages?anchor=$LAST_ID&num_after=50&narrow=[{\"operator\":\"stream\",\"operand\":\"DIRETORIA\"}]")

# Processar com LLM gratuita (Groq/Ollama)
# Passar mensagens pro LLM classificar: tem tag? precisa cobrar?
# LLM responde com acoes a tomar
# Executar acoes via zulip-send

# Atualizar ultimo ID
echo "$NEW_LAST_ID" > "$LAST_ID_FILE"
```

**Opcao B — Claude Code com prompt (recomendado)**
```bash
# cron: * * * * * claude --print -p "RUN-SPIKE-WATCH" --model haiku
```

Prompt RUN-SPIKE-WATCH:
1. Ler ultimas mensagens do Zulip (usar MCP zulip ou curl)
2. Classificar: tem tag [INICIATIVA/MONITOR/REVIEW]?
3. Checar prazos baseado na matriz de KPIs
4. Se precisa cobrar → zulip-send com mensagem padrao
5. Se 09h → postar resumo diario
6. Registrar no banco

## LLM Gratuita pra Classificacao

**Groq (gratis, rapido):**
- Modelo: llama-3.1-8b ou mixtral-8x7b
- Usar pra: classificar mensagem (tem tag? qual fase? quem foi marcado?)
- Nao usar pra: gerar texto longo ou decisoes

**Ollama (self-hosted):**
- Se CTO tiver rodando Ollama na infra
- Mesmo uso: classificacao rapida

**Haiku (barato):**
- Se nao tiver gratis disponivel, Haiku custa centavos
- Ideal pra: ler mensagem + decidir acao + gerar cobranca curta

## Crontab

```
# SPIKE watch — every minute
* * * * * /home/spike/ops/spike-watch.sh >> /home/spike/logs/spike.log 2>&1

# SPIKE resumo diario — 09h BRT (12h UTC)
0 12 * * * /home/spike/ops/spike-daily.sh >> /home/spike/logs/spike.log 2>&1
```

## Streams Monitorados

| Stream | O que monitorar |
|--------|----------------|
| DIRETORIA | [INICIATIVA], [REVIEW] |
| GERAL | [ANUNCIO] |
| WORKSPACE | [INICIATIVA], [ALINHAMENTO] |
| mentoringbase | [INICIATIVA] |
| MARKETING | [INICIATIVA] |

---

**Owner:** CTO configura o cron. SPIKE roda.
