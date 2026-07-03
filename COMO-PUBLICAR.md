# TPM Parceiros — PWA

App de campo pra registrar visitas a lojas de moto, gerir a rede de parceiros indicadores e o vale-compra.

## Arquivos
- `index.html` — o app inteiro (interface + lógica)
- `manifest.webmanifest` — identidade do app (nome, cor, ícones)
- `sw.js` — service worker (funciona offline)
- `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon-180.png` — ícones

> **Importante:** todos os arquivos precisam ficar na **mesma pasta** e ser servidos por **HTTPS**.
> O service worker NÃO funciona abrindo o `index.html` direto do computador (file://). Precisa estar hospedado.

## Publicar — opção GitHub Pages (recomendado, grátis)
1. Crie um repositório (ou use o `ecartsolucoes.github.io` que você já tem).
2. Suba todos os arquivos desta pasta na raiz (ou numa subpasta, ex: `/parceiros`).
3. Em **Settings → Pages**, ative o Pages na branch `main`.
4. Acesse `https://SEU-USUARIO.github.io/parceiros/` no celular.

## Publicar — opção cPanel / GoDaddy (você já usa)
1. Entre no Gerenciador de Arquivos.
2. Crie uma pasta, ex: `public_html/parceiros`.
3. Faça upload de todos os arquivos desta pasta para lá.
4. Confirme que o site tem SSL (cadeado). Acesse `https://seudominio.com.br/parceiros/`.

## Instalar no celular
- **Android (Chrome):** aparece o banner "Instalar na tela inicial" — toque em **Instalar**. (Ou menu ⋮ → Instalar app.)
- **iPhone (Safari):** toque em **Compartilhar** → **Adicionar à Tela de Início**. O app aparece com o ícone TPM e abre em tela cheia.

Depois de instalado, abre como app de verdade e funciona mesmo sem internet (os dados ficam no aparelho).

## Publicar uma atualização
Quando você editar o `index.html`, abra o `sw.js` e troque a versão:
`const VERSION = "tpm-parceiros-v1";` → `"tpm-parceiros-v2"`
Isso força o app a baixar a versão nova no próximo acesso.

## Onde os dados ficam (importante)
Hoje os dados de cada visita ficam **salvos só no aparelho** (no navegador do celular). Isso é ótimo pra protótipo e uso de um único usuário, mas:
- Não sincroniza entre o celular do Urso e o seu painel.
- Some se o app for desinstalado ou os dados do navegador forem limpos.

**Próximo passo de verdade:** plugar num backend pra você ver tudo em tempo real — uma planilha no SharePoint/Sheets, ou o seu Worker da Tiny (`tiny-proxy.diegocos.workers.dev`). Aí o Urso registra no campo e você acompanha do escritório.
