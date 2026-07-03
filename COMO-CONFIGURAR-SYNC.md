# Conectar o TPM Parceiros ao SharePoint (via N8N)

## O que isso faz
Toda vez que o Urso salva um check-in no app, o app manda os dados pro seu N8N, e o N8N grava uma linha na sua planilha do SharePoint. Se ele estiver sem internet, o app guarda e reenvia quando a conexão voltar — nada se perde.

## Passo 1 — Criar uma planilha NOVA e separada (não use a do DRE)
Motivo: o DRE é dado financeiro/fiscal com seu próprio pipeline; check-in de visita é dado comercial, com outro ritmo e outra audiência (pode crescer bastante e ser aberto pro Alexandre no futuro, por exemplo). Manter separado evita que um problema num afete o outro, e evita expor a planilha financeira pra quem só precisa ver parceiros.

1. No SharePoint, crie um **novo arquivo Excel**, por exemplo `TPM_PARCEIROS.xlsx`, numa biblioteca de documentos (pode ser a mesma biblioteca do DRE, só o arquivo é que é outro).
2. Dentro dele, crie uma aba chamada **`CheckinsParceiros`**.
3. Na primeira linha, coloque estes cabeçalhos, nesta ordem:

```
enviado_em | parceiro_id | parceiro_nome | parceiro_tipo | regiao | endereco | estagio | com_quem | papel | nota | material_cartoes | material_display | vale_qtd | vale_valor_unit | vale_total | proximo_passo | proximo_data | gps_lat | gps_lng | vendedor
```

4. Selecione essas células (cabeçalho + algumas linhas abaixo) e formate como **Tabela** (Ctrl+T / Inserir → Tabela). O N8N precisa que seja uma tabela formatada, não só células soltas.

## Passo 2 — Importar o fluxo no N8N (workflow próprio, não mexe no do DRE)
1. Abra seu N8N (`http://144.22.232.27:5678`).
2. Menu → **Import from File** → selecione `n8n-tpm-parceiros-checkin.json`.
3. Abra o node **"Gravar no Excel (SharePoint)"**:
   - Em **Credential**, selecione a mesma credencial que você já usa no pipeline de DRE (`n8n DRE TPM`) — é só autenticação, pode reaproveitar sem risco.
   - Em **Workbook**, selecione o arquivo **novo** `TPM_PARCEIROS.xlsx` (não o do DRE).
   - Em **Worksheet**, selecione **`CheckinsParceiros`**.
4. Clique em **Save**.
5. Ative o fluxo (toggle **Active** no canto superior direito). Ele fica completamente independente do fluxo de DRE — pode ativar, pausar ou editar um sem afetar o outro.
6. Abra o node **"Webhook Check-in"** e copie a **Production URL** (algo como `http://144.22.232.27:5678/webhook/tpm-parceiros-checkin`).

## Passo 3 — Configurar essa URL no app
1. Abra o app TPM Parceiros no celular ou no navegador.
2. Na tela **Painel**, role até o final e toque em **"⚙︎ Configurar sincronização com SharePoint"**.
3. Cole a URL do webhook copiada no Passo 2.
4. Toque em **Salvar**.

Pronto. A partir daí, cada check-in salvo aparece como uma nova linha na aba `CheckinsParceiros` da sua planilha — geralmente em poucos segundos.

## Como saber se está funcionando
- No app, se aparecer uma faixa amarela dizendo **"X check-in(s) pendente(s)"**, é porque o envio ainda não confirmou (sem internet, ou URL errada). O app tenta de novo sozinho.
- Se não aparecer nenhuma faixa, está tudo sincronizado.
- Você também pode tocar na faixa (ou no botão de configurar) e usar **"Tentar enviar agora"** pra forçar uma nova tentativa.

## Se quiser ver os dados em tempo real
Abra o arquivo `TPM_PARCEIROS.xlsx` pelo navegador ou pelo Excel — os check-ins do Urso vão aparecer como novas linhas na aba `CheckinsParceiros`, sem precisar de nenhum outro passo. Dá pra montar um dashboard (tabela dinâmica, gráfico) em cima dessa aba, separado do seu DRE.

---

## Via inversa — cadastrar leads pela planilha (Excel → App)

Além do Urso mandar check-ins pro Excel, você (ou outro chefe) pode cadastrar lojas direto na planilha e elas aparecem sozinhas no celular dele.

### Passo 1 — Copiar a aba `LeadsNovos` pra dentro do seu workbook
1. Abra o arquivo `LeadsNovos_template.xlsx` que preenchi pra você.
2. Abra também o seu `TPM_PARCEIROS.xlsx` (o mesmo do check-in).
3. Com os dois arquivos abertos no Excel, clique com o botão direito na aba **`LeadsNovos`** (embaixo, no rodapé da planilha) → **Mover ou Copiar** → em "Para o livro", escolha `TPM_PARCEIROS.xlsx` → marque **"Criar uma cópia"** → OK.
4. Agora seu `TPM_PARCEIROS.xlsx` tem 3 abas: `CheckinsParceiros`, `ResumoParceiros` e `LeadsNovos`.
5. Suba esse arquivo atualizado de volta pro SharePoint (substituindo o antigo).

### Passo 2 — Preencher leads
Na aba `LeadsNovos`, você só precisa preencher a coluna **`nome`** (o resto é opcional, mas ajuda o Urso). O `lead_id` se preenche sozinho. Tem dropdown pronto nas colunas `tipo` e `prioridade`.

### Passo 3 — Importar o segundo fluxo no N8N
1. No N8N, importe o arquivo `n8n-tpm-parceiros-leads.json` (mesmo processo do fluxo de check-in: menu → Import from File).
2. Abra o node **"Ler Leads (SharePoint)"**:
   - **Credential:** a mesma de sempre (`n8n DRE TPM`).
   - **Workbook:** `TPM_PARCEIROS.xlsx`.
   - **Worksheet:** `LeadsNovos`.
3. Salve e ative o fluxo.
4. Abra o node **"Webhook Buscar Leads"** e copie a **Production URL**.

### Passo 4 — Colar a URL no app
No app → Painel → **⚙︎ Configurar sincronização** → cole essa segunda URL no campo **"URL do webhook — buscar leads novos"** → Salvar.

### Como funciona no dia a dia
- O app busca leads novos sozinho toda vez que abre, e de novo quando a internet volta.
- O Urso também pode forçar uma busca manual: na aba **Parceiros**, tem um botão **"🔄 Leads"**.
- Cada lead vira um parceiro novo no app, no estágio **"Novo"**, já com a prioridade e observação visíveis na ficha dele.
- Se você reeditar a mesma linha na planilha (mesmo `lead_id`), ela **não duplica** no app — o `lead_id` é a chave que evita repetição.

---

## Login do time (usuário e senha pela planilha)

Em vez de cada vendedor só digitar o nome, dá pra exigir usuário e senha — validados contra uma aba da própria planilha.

> **Importante:** isso é uma trava simples de identificação, não segurança forte. A senha fica em texto puro na planilha e trafega pelo webhook sem criptografia extra. Serve pra saber quem fez cada check-in, não pra proteger dados sensíveis. Troque a senha de quem sair da equipe.

### Passo 1 — Copiar a aba `Usuarios` pra dentro do seu workbook
1. Abra o arquivo `Usuarios_template.xlsx` que preenchi pra você (já vem com 2 linhas de exemplo).
2. Abra também o seu `TPM_PARCEIROS.xlsx`.
3. Clique com o botão direito na aba **`Usuarios`** → **Mover ou Copiar** → escolha `TPM_PARCEIROS.xlsx` → marque **"Criar uma cópia"** → OK.
4. Preencha uma linha por pessoa: `usuario`, `senha`, `nome` (o nome completo é o que aparece nos check-ins dela).
5. Apague as 2 linhas de exemplo (gabriel/nagila) ou substitua pelos usuários reais.
6. Suba o arquivo atualizado de volta pro SharePoint.

### Passo 2 — Importar o terceiro fluxo no N8N
1. Importe `n8n-tpm-parceiros-login.json` (mesmo processo dos outros dois fluxos).
2. Abra o node **"Ler Usuários (SharePoint)"**: mesma credencial (`n8n DRE TPM`), workbook `TPM_PARCEIROS.xlsx`, aba `Usuarios`.
3. Salve e ative o fluxo.
4. Copie a **Production URL** do node **"Webhook Login"**.

### Passo 3 — Colar no app
No app → Painel → **⚙︎ Configurar sincronização** → cole essa URL no campo **"URL do webhook — login do time"** → Salvar.

### Como funciona
- Assim que essa URL é configurada, o app passa a pedir **usuário e senha** em vez do nome simples.
- Se a pessoa não quiser entrar, tem o botão **"Lançar como anônimo"** — o check-in vai gravado como `Anônimo` na planilha, mas continua funcionando normalmente.
- Se o login falhar (usuário ou senha errados), aparece um aviso claro e a pessoa pode tentar de novo ou lançar anônimo.
- Enquanto essa URL não estiver configurada, o app continua funcionando com o fluxo simples de nome (não trava ninguém esperando você configurar isso).
