# fable5skill

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/fable5skill/guia/**

Skills para [Claude Code](https://claude.com/claude-code) que geram sites **scroll-film** — páginas onde a rolagem inteira funciona como um plano-sequência cinematográfico contínuo, que se dissolve no conteúdo normal do site ao final.

## scroll-film-studio

A skill principal deste repositório. É um **processo, não um template**: entrevista, pitcha 2-3 conceitos criativos nomeados, faz a direção de arte (paleta, tipografia, motion) e constrói a página do zero — HTML/CSS/JS self-contained.

Instalar:
```bash
cp -r scroll-film-studio ~/.claude/skills/scroll-film-studio
```

### As três lanes (caminhos de construção)

| Lane | O que é | Custo | Setup |
|---|---|---|---|
| **A — Pure-code** (padrão) | GSAP + Lenis: cenas fixadas, parallax, reveals | grátis | zero — só CDN |
| **B — Footage (Higgsfield/Kie.ai/fal/Replicate)** | Vídeo real gerado por IA, encadeado shot-a-shot (last frame → next start image). Testada em produção com **Kie.ai Seedance** (`bytedance/v1-lite-image-to-video`, ~22 créditos/clipe 720p) | créditos do engine | conta própria no serviço |
| **C — Footage via Agnes AI** | Mesmo contrato da Lane B, adaptado pro Agnes (`agnes-video-v2.0`, modo `keyframes`, interpola par de imagens em vez de gerar a partir de 1 start image) | **US$0** | `AGNES_API_KEY` |

A Lane A funciona pra qualquer pessoa que baixe a skill, sem contas nem custo. As Lanes B/C são opt-in — só entram quando o usuário já tem (ou quer) um engine de vídeo por IA.

### Estrutura

```
scroll-film-studio/
├── SKILL.md                    # o processo (entrevista → pitch → build → verify → deploy)
├── references/
│   ├── engine.md                # receitas de build: canvas scrub engine (Lane B/C) + motion vocabulary (Lane A)
│   ├── playbook.md               # a lei da Lane B (Higgsfield/Kie.ai/fal/Replicate)
│   └── lane-c-agnes.md            # a lei da Lane C (Agnes AI) — par de keyframes por capítulo, custo zero
└── scripts/
    ├── chain-step.sh              # gera 1 clipe encadeado (Lane B) + gate de SSIM na emenda
    ├── chain-step-agnes.sh         # mesmo contrato, engine Agnes (Lane C)
    ├── assemble.sh                 # concatena os clipes, extrai frames pro scrubber, amostra a cor da emenda
    └── verify.js                    # puppeteer-core: teste de jank (rAF) + screenshots via `?jump=`
```

### A regra de ouro

Toda decisão de **gosto** (conceito, direção de arte, paleta, tipografia, motion, copy, o código da página) é feita pelo Claude que roda a skill — nunca delegada a outro modelo. Trabalho mecânico (ffmpeg, SSIM, extração de frames, deploy) vai para scripts puros, sem modelo nenhum. Um segundo CLI frontier (Codex, Gemini) pode opcionalmente ser usado só pra *criticar* os conceitos antes da escolha — nunca pra escrever design ou código.

### Verificação

Todo build implementa um contrato de dev: `?jump=<scrollY>` pra pousar pré-rolado e `window.__ready` pra sinalizar que a página assentou. `scripts/verify.js` usa isso pra tirar screenshot em qualquer posição de scroll e medir jank real (p95/max de deltas de `requestAnimationFrame`), nunca FPS médio.

### Exemplo construído com esta skill — as 3 lanes lado a lado

**Sinal** ([repo](https://github.com/inematds/inema-sitefilm)) — o scroll-film do INEMA.CLUB, mesma jornada (Ruído → Convergência → Núcleo → Rede → Portal) construída pelas três lanes, com menu de versões no topo:

| Versão | Lane / Engine | Link |
|---|---|---|
| Filme I | Lane C — Agnes AI (US$0, keyframes interpolados) | [ver](https://inematds.github.io/inema-sitefilm/) |
| Filme II — Director's Cut | Lane C (mesma footage; filme dentro das letras, dots de capítulo, timeline de scrub) | [ver](https://inematds.github.io/inema-sitefilm/index2.html) |
| Filme III | **Lane B — Kie.ai Seedance Lite** (cadeia last-frame→start real, ~112 créditos) | [ver](https://inematds.github.io/inema-sitefilm/index3.html) |
| Code I | Lane A — GSAP+Lenis vertical (jank max 17ms) | [ver](https://inematds.github.io/inema-sitefilm/lane-a.html) |
| Code II | Lane A — corrida horizontal pinada, âmbar | [ver](https://inematds.github.io/inema-sitefilm/lane-a2.html) |

Os runners usados na Lane B do exemplo (`kie-chain.sh` pro Kie.ai e `fal-chain.sh` pro fal.ai — upload → createTask → poll → download → extração do último frame → gate SSIM) estão no [repo do exemplo](https://github.com/inematds/inema-sitefilm) e seguem o contrato de `scripts/chain-step.sh`.

**Notas de campo do Kie.ai** (medidas na prática): modelo `bytedance/v1-lite-image-to-video` via `POST /api/v1/jobs/createTask` + poll em `GET /api/v1/jobs/recordInfo?taskId=`; **data URI não é aceito** — subir a imagem antes em `https://kieai.redpandaai.co/api/file-base64-upload` e passar a `downloadUrl`; saldo em `GET /api/v1/chat/credit`; ~22,5 créditos por clipe 720p/5s.

## Guardrails

- Zero dados pessoais ou API keys no repositório — cada usuário traz sua própria conta de engine de vídeo (Lane B/C) e de deploy (Vercel/GitHub Pages).
- Respeita `prefers-reduced-motion` em todo build.
- Nunca sobe intermediários de build (clipes brutos, keyframes de referência) pro deploy final — só o `index.html` + os frames realmente usados pelo scrubber.
