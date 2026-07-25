# Mimba — Landing (contexto do projeto)

> Contexto sempre carregado. Este repo é **só a landing** (`mimba.com.br`). O app/produto e o backend vivem em `projetos/cabanha` (leia o `HANDOFF.md` de lá para o estado do produto).

## O que é
Landing page de marketing/conversão do **Mimba** — SaaS multi-tenant de gestão para cabanhas de cavalo crioulo (razão social: Mimba Tech; posicionamento v1: "Gestão Crioulos", integra ABCCC). Objetivo da página: **apresentar o produto e converter em assinatura** (checkout → Asaas → provisiona a cabanha no backend).

- **Landing:** `mimba.com.br` (este repo, `mimba-app/mimba-landing`).
- **App:** `app.mimba.com.br` (repo separado — a cabanha).

## Arquitetura
- **Frontend:** `index.html` **único, HTML puro** — CSS inline/`<style>` + um `<script>` de vanilla JS (fade-in via IntersectionObserver). **Sem framework, sem bundler, sem package.json** (mesma filosofia do app da cabanha). O `index.html` **é a fonte de verdade** — edita direto e faz push. ~43 KB.
- Hospedado no **GitHub Pages** (push na `main` → publica; workflow `versionar.yml` arquiva as últimas 10 versões em `versions/`).
- **Histórico:** até 2026-07-25 o deploy era um *bundle* gerado (React/base64, 615 KB, `<title>` "Bundled Page", com `image-slot`s **vazios**). Foi substituído pela versão HTML puro (source `uploads/landing.html`), que bate com a convenção do projeto. O pipeline dc/bundle foi descartado.
- **Marca Mimba** via **variáveis CSS** em `:root` (`--terra`, `--ouro`, `--campo`, `--creme`…), **consistente com o app**: verde campo `#2D5A3D` (ação), ouro `#B8860B` (acento). Fontes **DM Sans** (corpo), **Playfair Display** (títulos), **DM Mono**. Não hardcode cor/fonte fora das variáveis.

## Checkout (Asaas)
- A integração de pagamento é do **Asaas** (não há MCP; é API REST).
- **A chave do Asaas NUNCA vai no client-side** de uma página estática. O checkout ou **redireciona para um payment link / checkout hospedado** do Asaas, ou **POSTa para uma Edge Function** (que vive no lado da cabanha/Supabase e insere o `signup`).
- Chave **sandbox** para desenvolvimento fica **local e gitignored** — nunca commitada. Ver a skill `checkout-asaas`.

## Convenções
- **`index.html` é a fonte de verdade e o que é servido** — edita direto, sem build step. Conferir o staged antes do commit de produção.
- **Deploy** é pela skill `deploy` (valida, commita só o `index.html`, push, confere o build do Pages).
- Commits terminam com `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- **GitHub via `gh` CLI** (já autenticado, WRITE em `mimba-app/mimba-landing`) — não precisa de MCP do GitHub.

## Regras que nunca podem ser quebradas
- **Nunca commitar segredos** — chave/token do Asaas, PATs, service_role. Nada de secret client-side.
- **Nunca expor a chave do Asaas no `index.html`** (é público). Pagamento passa por payment link ou edge function.
- **Só commitar o `index.html`** em produção — conferir `git diff --cached --name-only` antes do push. Se vier outra coisa, **PARE**.
- **Não quebrar a marca** — cor/fonte vêm das variáveis CSS da paleta Mimba, não hardcoded fora do sistema.
- Manter `<title>`, `meta description`, `lang` e SEO reais — não regredir para placeholder.

## Extensões deste projeto (`.claude/`)
- **Subagentes** (`.claude/agents/`):
  - `designer` — design visual & animações (marca, motion, responsivo, acessibilidade).
  - `engenheiro-frontend` — implementação na landing (HTML puro, performance, SEO técnico, acessibilidade).
  - `copywriter-cro` — copy & conversão (headline, CTA, oferta, prova social, SEO de conteúdo).
  - `growth-analytics` — funil, GTM/GA4, plano de tracking, UTM, captação/enriquecimento de leads.
  - `integrador-checkout` — fluxo de checkout Asaas (form → payment link/edge function → signup).
- **Skills** (`.claude/skills/`):
  - `deploy` — deploy seguro da landing (só `index.html` → Pages).
  - `checkout-asaas` — playbook da integração de pagamento (sandbox, sem segredo no repo).

## Relação com a cabanha
Backend (Supabase, provisionamento, Asaas webhook, `signups`/`planos`) e o app estão em `projetos/cabanha`. O checkout desta landing **alimenta** aquele fluxo (cria `signup` + assinatura no Asaas → webhook → provisiona a cabanha). Para detalhes do backend, leia `../cabanha/HANDOFF.md` e `../cabanha/CLAUDE.md`.
