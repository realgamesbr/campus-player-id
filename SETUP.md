# Setup — Página de Resgate Campus Player ID

Pacote: HTML + Apps Script + Política. Adaptado do padrão Vinhos de Portugal, mesma estrutura — só muda visual, copy e `game_id`.

## Visão geral

```
[Visitante escaneia QR no totem]
        ↓
[Abre share.html no celular] → vê form de captura
        ↓
[Digita email + checks] OU [clica "Não quero deixar meu email"]
        ↓
[Form posta JSON pro Apps Script Web App]
        ↓
[Apps Script salva linha na Google Sheet]
        ↓
[HTML mostra a carta + botão Baixar Carta]
```

## Arquivos do pacote (`web/`)

- `share.html` — página principal exibida ao escanear o QR (placeholder `{{media_placeholder}}` é a URL do PNG da carta final, substituído pelo Content Mover)
- `politica_privacidade.html` — política linkada no checkbox de consentimento
- `apps_script.gs` — código pra colar no Google Apps Script (backend)
- `SETUP.md` — este guia

---

## Passo 1 — Google Sheet (~3 min)

1. Acesse [sheets.google.com](https://sheets.google.com) → **+ Em branco**
2. Renomeia a planilha (canto superior esquerdo) pra **"Campus Player ID CPBR18 — Eventos"**
3. Renomeia a aba (canto inferior) de "Página1" pra **`Eventos`** (case-sensitive!)
4. Pode deixar a planilha vazia — o Apps Script cria os headers automaticamente na primeira chamada. Mas se quiser preparar, cola na linha 1:

   `timestamp | event | game_id | email | consent_marketing | consent_privacy | photo_id | user_agent`

   Cada item em uma coluna. Negrito + congelar (Exibir > Congelar > 1 linha) é opcional.

## Passo 2 — Apps Script (~5 min)

1. Na Sheet, **Extensões > Apps Script**
2. Apaga o código padrão `function myFunction() {}`
3. Abre o arquivo `apps_script.gs` deste pacote, copia TUDO, cola no editor do Apps Script
4. Salva (`Ctrl+S` ou disquete) → nomeia o projeto: **"Campus Player ID — Captura Leads"**
5. Clica **Implantar > Nova implantação**
6. Engrenagem ao lado de "Selecionar tipo" → **Aplicativo da Web**
7. Configura:
   - **Descrição:** `Captura de leads Campus Player ID CPBR18`
   - **Executar como:** `Eu (seu-email@gmail.com)` (default)
   - **Quem pode acessar:** **`Qualquer pessoa`** ← crítico, senão dá 403
8. Clica **Implantar**
9. Autoriza quando o Google pedir:
   - Vai mostrar "Google não verificou este app"
   - Clica **Avançado > Acessar (nome do projeto) (não seguro)**
   - Permite acesso à sua Sheet
10. **Copia a URL do Web App** que aparece — formato:
    ```
    https://script.google.com/macros/s/AKfycb...../exec
    ```

## Passo 3 — Colar URL no HTML (~30s)

Abre `share.html` (este pacote) e localiza esta linha (perto do `// CONFIGURACAO`):

```javascript
const APPS_SCRIPT_URL = 'COLE_AQUI_A_URL_DO_APPS_SCRIPT_WEB_APP';
```

Substitui pela URL do Passo 2.10.

## Passo 4 — Teste rápido (~2 min)

1. Abre a URL do Apps Script (passo 2.10) **direto no navegador** — deve retornar:
   ```json
   {"ok":true,"msg":"Endpoint ativo — Campus Player ID","expected_game_id":"campus_player_id_cpbr18"}
   ```
   Se aparecer isso, o backend está vivo.

2. Pra testar o HTML localmente sem o Content Mover, edita uma cópia do `share.html` e troca `{{media_placeholder}}` por uma URL de imagem qualquer (ex: uma das `card-a.jpg` do projeto: `https://i.imgur.com/algumacoisa.png`). Abre no navegador.

3. Preenche email + marca privacy → clica "Ver Minha Carta" → deve mostrar a carta. Volta na Sheet → deve ter uma linha nova.

4. Recarrega a página com a mesma URL → deve ir direto pra carta (localStorage lembra).

5. Em outra aba (URL diferente, ex: `?id=test2`), clica "Não quero deixar meu email" → não vai pra Sheet com email, mas registra evento `skip`.

## Passo 5 — Política de Privacidade (~1 min)

A `politica_privacidade.html` já está preenchida com os dados do Real Studio LTDA. Se algo precisar mudar (data, CNPJ, email), edita os campos no arquivo.

A política é linkada via path relativo no HTML (`./politica_privacidade.html`), então hospeda os dois na mesma pasta no servidor final.

## Passo 6 — Hospedar e Gerar QR Codes (Fase 4 — depende do Content Mover)

Quando o Content Mover estiver integrado:
1. Totem gera a carta (PNG composto) e faz upload pro Content Mover
2. Content Mover hospeda o PNG numa CDN (ex: `realgamesstudio.com.br/cpid/cards/00042.png`)
3. Content Mover gera uma cópia única do `share.html` substituindo `{{media_placeholder}}` pela URL do PNG
4. Hospeda em URL pública (ex: `realgamesstudio.com.br/cpid/00042.html`)
5. QR code no totem aponta pra essa URL

---

## LGPD — Pontos importantes

- **Email é opcional** — botão "Não quero deixar meu email" funciona sempre
- **Consent marketing é separado** do consent privacy — só envia marketing pra quem marcou marketing
- **Quem deixou email mas NÃO marcou marketing** fica registrado (evidência do consentimento) mas **não pode receber comunicação posterior**
- **Exportar pra ferramenta de email**: filtrar `consent_marketing = TRUE`
- **Descadastro**: marca linha como excluída ou apaga — mantém log

## Métricas (fórmulas pra colocar em aba "Métricas")

| Métrica | Fórmula |
|---|---|
| Total de cartas visualizadas | `=COUNTA(Eventos!A2:A)` |
| Leads capturados | `=COUNTIF(Eventos!B2:B,"lead")` |
| Skips (não deixou email) | `=COUNTIF(Eventos!B2:B,"skip")` |
| Taxa de conversão | `=COUNTIF(Eventos!B2:B,"lead")/COUNTA(Eventos!A2:A)` |
| Leads c/ opt-in marketing | `=COUNTIFS(Eventos!B2:B,"lead",Eventos!E2:E,TRUE)` |
| Taxa opt-in marketing | `=COUNTIFS(Eventos!B2:B,"lead",Eventos!E2:E,TRUE)/COUNTIF(Eventos!B2:B,"lead")` |

## Troubleshooting

| Sintoma | Causa provável |
|---|---|
| 403 ao chamar a URL do Apps Script | Não marcou "Qualquer pessoa" no deploy. Implanta de novo |
| Lead não aparece na Sheet | URL do Apps Script errada no HTML; ou Sheet não tem aba `Eventos` |
| "Email inválido" mesmo válido | Não passou no regex (`/^[^\s@]+@[^\s@]+\.[^\s@]+$/`) — tem espaço/char estranho? |
| Botão "Ver Minha Carta" sempre desabilitado | Falta checkbox de privacy marcado, OU email não passa no regex |
| Volta na URL e não pede email | Comportamento esperado — localStorage lembra. Pra resetar: `localStorage.clear()` |

## Reaproveitar pra outra edição do evento (CPBR19, etc)

1. Duplica a Sheet (`Arquivo > Fazer uma cópia`)
2. Abre Apps Script da cópia → muda `EXPECTED_GAME_ID` pra novo valor (ex: `'campus_player_id_cpbr19'`)
3. Reimplanta como **Nova implantação > Web App > Qualquer pessoa** → nova URL
4. Cola nova URL no `share.html` da nova edição
5. Atualiza `GAME_ID` no `share.html` pra bater
