# WhatsApp Prospecting Engine

![CI](https://github.com/Dimitrearaujo/whatsapp-prospecting-engine/actions/workflows/ci.yml/badge.svg)

Motor de prospeccao outbound automatizado — busca leads no Google Maps via Apify, dispara mensagens personalizadas pelo WhatsApp (Evolution API) e email (Resend), com rate limiter anti-bloqueio e agente IA BANT para qualificacao de respostas.

Em producao: 30 mensagens WhatsApp/dia e 200 emails/dia com delay aleatorio entre envios.

## Arquitetura

```
Apify (Google Maps Scraper)
         |
         v
    prospects.db (SQLite)
         |
    prospecting_engine.py
         |
    +----+----+
    |         |
Evolution   Resend
API (WA)   (Email)
    |         |
    +----+----+
         |
    agent_bant.py
    (responde via IA)
         |
    Dashboard HTML local
```

## Modulos

| Arquivo | Funcao |
|---|---|
| `prospecting_engine.py` | Motor principal — busca, disparo, dashboard |
| `agent_bant.py` | Agente IA que responde leads via WhatsApp (BANT + GPT) |
| `rate_limiter.py` | Controle de limite diario com delay aleatorio anti-bloqueio |
| `dispatcher.py` | Dispatcher multicanal (WhatsApp + Email) |
| `apify_scraper.py` | Scraping de leads no Google Maps via Apify |
| `copy_generator.py` | Gerador de mensagens personalizadas por segmento |
| `monitor.py` | Monitor 24/7 — avisa no WhatsApp se algo cair |
| `lib.py` | Utilitarios compartilhados, paths, config |

## Instalacao

```bash
git clone https://github.com/Dimitrearaujo/whatsapp-prospecting-engine
cd whatsapp-prospecting-engine
pip install -r requirements.txt

cp config.example.json config.json
# Edite config.json com suas chaves
```

**Pre-requisitos:**
- Evolution API rodando (WhatsApp conectado)
- Conta Resend (3.000 emails/mes gratis)
- Token Apify (scraping de leads — $5/mes free tier)
- Chave OpenAI (agente BANT)

## Uso

```bash
# Buscar novos leads (Apify scraping)
python prospecting_engine.py --search

# Disparar mensagens (WhatsApp + Email)
python prospecting_engine.py --send

# Simular sem enviar
python prospecting_engine.py --send --dry-run

# Marcar lead como respondido
python prospecting_engine.py --mark-responded 5585999990000

# Abrir dashboard no browser
python prospecting_engine.py --dashboard

# Rotina diaria completa (search + send)
python prospecting_engine.py --daily --limit 30
```

## Rate Limiter

O `rate_limiter.py` garante que o limite diario nunca seja ultrapassado e adiciona delay aleatorio entre envios para comportamento humano:

- WhatsApp: 30/dia, delay 60-120s entre mensagens
- Email: 200/dia, delay 15-30s entre emails
- Persistencia em JSON — resiste a reinicializacoes

## Segmentos suportados

Clinicas veterinarias, odontologicas e esteticas, academias, studios de pilates/yoga — mensagens personalizadas por segmento via `copy_generator.py`.

## Licenca

MIT
