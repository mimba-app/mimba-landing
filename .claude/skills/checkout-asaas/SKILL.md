---
name: checkout-asaas
description: Playbook da integração de checkout com o Asaas na landing do Mimba — sandbox, payment link vs. edge function, contrato do signup, e como guardar a chave sem nunca commitá-la. Use ao construir ou depurar o checkout.
---

# Checkout Asaas (landing → assinatura → provisiona a cabanha)

O checkout liga a landing ao provisionamento da cabanha. **Não existe MCP do Asaas** — é API REST. **A chave nunca vai no client.**

## Segredo — onde guardar (nunca no repo)
A landing é pública; a chave do Asaas não pode viver no `index.html` nem em nenhum arquivo versionado.
- **Sandbox (dev):** guarde a chave num arquivo **gitignored** (ex.: `.env` local, já coberto pelo `.gitignore`) ou peça ao usuário no momento de testar. Nunca a escreva num arquivo versionado.
- **Produção:** a chave vive como **secret no servidor** (Edge Function no lado da cabanha/Supabase), nunca na landing.
- Se precisar da chave e ela não estiver disponível localmente, **peça ao usuário** — não invente nem chute.

## Duas arquiteturas corretas (escolher com o `arquiteto`/`integrador-checkout`)
1. **Payment link / checkout hospedado do Asaas** — a landing redireciona para uma URL de cobrança do Asaas. Mais simples, menos código, sem tocar em chave no fluxo da landing. O webhook (`asaas-webhook`, lado cabanha) provisiona ao confirmar o pagamento.
2. **Edge Function intermediária** — a landing POSTa o form para uma função no lado da cabanha, que fala com o Asaas usando a chave-secret e insere o `signup`. Mais controle sobre UX/campos.

## Contrato do signup (alinhar com `../cabanha`)
Campos que o backend espera em `public.signups`: `nome_cabanha`, `email_admin`, `plano_id`, `cnpj_cpf`, `abccc_codigo`, `nome_exibicao`, `logo_url`, `asaas_*`, `status`. Planos em `public.planos` (Potro/Arreio/Tropilha). **Confira os campos e IDs reais em `../cabanha/HANDOFF.md`** — não invente.

## Fluxo de trabalho
1. **Sandbox primeiro.** Monte e valide o fluxo inteiro no sandbox do Asaas (criar cliente, criar assinatura/link, receber webhook simulado) antes de qualquer coisa em produção.
2. Testar payment link ou a chamada à edge function; verificar o `signup` sendo criado.
3. **Webhook em produção fica DESATIVADO até o go-live** (ver `HANDOFF.md` da cabanha) — não ative por conta própria.
4. Nunca colocar dado pessoal/pagamento em query string. Consentimento/LGPD conforme o plano do `growth-analytics`.

## Checklist de segurança antes de commitar
- [ ] Nenhuma chave/token do Asaas no diff.
- [ ] `index.html` não contém segredo (só ID público, se houver).
- [ ] Chave de sandbox está em arquivo gitignored, não versionado.
