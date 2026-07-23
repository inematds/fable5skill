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
| **B — Footage (Higgsfield/Kie.ai/fal/Replicate)** | Vídeo real gerado por IA, encadeado shot-a-shot (last frame → next start image) | créditos do engine | conta própria no serviço |
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

### Exemplo construído com esta skill

[Sinal — o scroll-film do INEMA.CLUB](https://inematds.github.io/inema-sitefilm/) — Lane C, gerado com Agnes AI, 5 capítulos indo do ruído de dados dispersos até o acesso centralizado do portal.

## Guardrails

- Zero dados pessoais ou API keys no repositório — cada usuário traz sua própria conta de engine de vídeo (Lane B/C) e de deploy (Vercel/GitHub Pages).
- Respeita `prefers-reduced-motion` em todo build.
- Nunca sobe intermediários de build (clipes brutos, keyframes de referência) pro deploy final — só o `index.html` + os frames realmente usados pelo scrubber.
