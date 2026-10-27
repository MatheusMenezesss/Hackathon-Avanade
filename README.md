# ADpilot — Copiloto Criativo

Plataforma de orquestração criativa em ciclo fechado para marketing de varejo farmacêutico. Automatiza o desdobramento multicanal de peças, aplica auditoria regulatória por categoria de produto e retém memória operacional entre campanhas.

---

## Visão geral

O ADpilot atua como uma camada de orquestração entre o briefing e a publicação — trabalhando em paralelo com Figma e Canva, sem substituí-los. O fluxo é dividido em cinco fases sequenciais com checkpoint de aprovação humana em cada transição.

```
F1 Briefing → F2 Marca → F3 Compositor → F4 Compliance Guard → F5 Memória
        ↑                                                              │
        └──────────────── ciclo fechado (Cartão de Produto) ──────────┘
```

---

## Estrutura do repositório

```
adpilot/
├── src/
│   ├── f1_briefing/          # Recepção de pedidos e estruturação via LLM
│   ├── f2_brand/             # Manual de Marca Sintético e Design Tokens
│   ├── f3_compositor/        # Motor de desdobramento multicanal por camadas
│   ├── f4_guard/             # Pipeline de auditoria regulatória
│   └── f5_memory/            # Cartão de Produto e base de conhecimento
├── data/
│   ├── catalog/              # Catálogo de SKUs com classificação regulatória
│   ├── rules/                # Regras ANVISA/CDC por categoria (JSON versionado)
│   └── brand/                # Manual de Marca Sintético parametrizado
├── templates/
│   ├── 1x1/                  # Templates JSON para formato Feed (1080×1080)
│   ├── 9x16/                 # Templates JSON para formato Stories (1080×1920)
│   └── 16x9/                 # Templates JSON para formato Banner (1920×1080)
├── tests/
│   ├── compliance/           # Conjunto ouro de 10 peças para validação do Guard
│   └── compositor/           # Testes de desdobramento com nomes curtos/médios/longos
├── docs/
│   ├── arquitetura.md        # Decisões técnicas e justificativas
│   ├── regulatorio.md        # Mapeamento RDC 96/2008, RDC 243/2018, CDC
│   └── lgpd.md               # Controles de privacidade implementados
├── assets/
│   └── packshots/            # Packshots PNG com fundo transparente por SKU
├── .env.example
├── requirements.txt
└── README.md
```

---

## Fases do sistema

### F1 — Briefing Inteligente
Recebe pedidos em texto, áudio ou webhook. LLM extrai SKU, preço e validade usando Structured Outputs — nunca infere dados ausentes. Cruza com o catálogo para identificar a classe regulatória e bloqueia medicamentos tarjados na origem.

**Stack:** Python · FastAPI · Pydantic · Whisper (STT)

### F2 — Manual de Marca Sintético
Gera Design Tokens no padrão W3C DTCG a partir do manual parametrizado em JSON. Exporta arquivo compatível com Figma via Token Studio.

**Stack:** Python · JSON Schema

### F3 — Compositor por Camadas
Recebe assets separados (fundo, packshot PNG transparente, textos) e os combina nas coordenadas definidas pelo Template Inteligente de cada formato. Renderiza via HTML/CSS headless para tipografia precisa.

**Stack:** Python · Pillow · Playwright · JSON templates

### F4 — Compliance Guard
Pipeline de auditoria automatizada que retorna um dos cinco estados: pré-check aprovado, pendência de revisão, não conformidade detectada, fora do escopo automatizado ou aprovação formal concluída.

Verificações:
- Presença e legibilidade do texto legal obrigatório por SKU (RapidFuzz)
- Contraste mínimo WCAG 2.1 AA — 4.5:1 (OpenCV)
- Zona de segurança do logo — 15% da largura (Google Cloud Vision)
- Preço na imagem vs. preço no briefing (OCR + comparação exata)

**Stack:** Python · Google Cloud Vision API · OpenCV · RapidFuzz

### F5 — Memória Operacional
Salva Cartão de Produto ao final de cada campanha aprovada. Pré-popula a Fase 1 em ciclos subsequentes do mesmo produto ou sazonalidade. Registra histórico de erros do Guard para antecipar falhas recorrentes.

**Stack:** Python · SQLite (MVP) · PostgreSQL (produção)

---



## Executando os testes

```bash
# Validação do Compliance Guard (conjunto ouro — 10 peças)
python -m pytest tests/compliance/ -v

# Teste do motor de desdobramento
python -m pytest tests/compositor/ -v
```

O conjunto ouro (`tests/compliance/`) contém 10 peças com erros plantados conhecidos. O critério de sucesso é recall de 7/7 peças irregulares bloqueadas com no máximo 1 falso positivo.

---

## Decisões técnicas relevantes

- **PNG achatado não é editável:** o Compositor nunca importa a peça fechada do Figma para reposicionar elementos. Opera exclusivamente com assets em camadas separadas + Template JSON.
- **LLM não infere dados comerciais:** preço, validade e classificação regulatória vêm do catálogo determinístico, nunca de inferência livre do modelo.
- **Regras regulatórias desacopladas do código:** o arquivo `data/rules/` é versionado separadamente e pode ser atualizado pelo time jurídico sem intervenção técnica.
- **Aprovação sempre humana:** o estado 🔵 Aprovação Formal só existe após clique explícito do responsável, com log imutável e hash SHA-256 da imagem.

---

## Status do MVP

| Módulo | Status |
|---|---|
| F1 — Briefing + catálogo SKU | Em desenvolvimento |
| F2 — Manual de Marca (JSON) | Em desenvolvimento |
| F3 — Compositor por camadas | Em desenvolvimento |
| F4 — Compliance Guard | Em desenvolvimento |
| F5 — Cartão de Produto | Em desenvolvimento |
| Via Rápida (Fast-Track) | Em desenvolvimento |
| Trend Radar / Mood Board Score | Fora do MVP — roadmap |
| Correlação criativo × performance | Fora do MVP — roadmap 6–12 meses |

---

## Licença

Proprietário — uso restrito à equipe do projeto.
