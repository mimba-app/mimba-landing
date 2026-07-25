---
name: deploy
description: Deploy seguro da landing do Mimba — valida o HTML, commita só o index.html, faz push e confere que o GitHub Pages publicou em mimba.com.br. Use ao subir uma mudança para produção.
---

# Deploy da landing (index.html → GitHub Pages → mimba.com.br)

Push na `main` publica no Pages; o workflow `versionar.yml` arquiva a versão anterior em `versions/`. **Só o `index.html`** deve ir no commit de produção.

## Passos
1. **Sanidade — o JS inline compila?**
   `node -e "const s=require('fs').readFileSync('index.html','utf8'); const m=s.split('<script>').pop().split('</script>')[0]; new Function(m); console.log('ok')"`
2. **Stage só o index.html** e confirme:
   `git add index.html && git diff --cached --name-only` → tem que ser exatamente `index.html`. Se vier outra coisa (ex.: um segredo, um arquivo do source), **PARE**.
3. **Sem segredos:** confirme que não há chave/token do Asaas, PAT ou service_role no diff. A landing é pública.
4. **Commit** — mensagem clara + trailer `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
5. **Rebase + push:** `git fetch -q origin main && git pull --rebase -q origin main && git push origin main`
   (o `versionar.yml` costuma criar um commit no remoto arquivando a versão anterior — o rebase resolve; conflito é raro porque a mudança dele é em `versions/`).
6. **Verificar o build do Pages:**
   `gh api repos/mimba-app/mimba-landing/pages/builds/latest --jq '{status,error:.error.message}'` → `built`, sem erro.
7. **Conferir ao vivo** (com cache-buster): baixe `https://mimba.com.br/?cb=<timestamp>` e cheque um marcador da mudança (uma string nova que você adicionou). Enquanto o DNS de `mimba.com.br` não publicar, use `https://mimba-app.github.io/mimba-landing/`.

## Regras
- Nunca commitar segredos nem outros arquivos junto do `index.html`.
- Build `errored`? Investigue antes de qualquer novo push.
- `index.html` é HTML puro e é a fonte de verdade — edições vão direto nele (ver `CLAUDE.md`).
