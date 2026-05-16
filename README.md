# Campus Player ID — Página de Resgate

Página de resgate (QR landing) da ativação **Campus Player ID** durante a **Campus Party Brasil 2026 (CPBR18)**, produzida pela [Real Games Studio](https://realgamesstudio.com.br).

O totem gera uma carta colecionável personalizada com IA, e o QR code aponta pra essa página, onde o visitante deixa email (opcional) e baixa a carta.

## Arquitetura

- **`share.html`** — página principal exibida ao escanear o QR. Mostra form LGPD + carta. O placeholder `{{media_placeholder}}` é substituído pelo Content Mover ao gerar a URL única de cada visitante. Em desenvolvimento, aceita `?img=URL` na query string como fallback de teste.
- **`politica_privacidade.html`** — Política de Privacidade LGPD-compliant, linkada no checkbox de consentimento.
- **`apps_script.gs`** — backend Google Apps Script (Web App) que recebe o POST do form e escreve em Google Sheet.
- **`SETUP.md`** — guia completo de setup (Sheet + Apps Script + deploy).

## Hospedagem

Esta página fica em GitHub Pages: <https://realgamesbr.github.io/campus-player-id/>

A versão de **produção** (com `{{media_placeholder}}` substituído pelo Content Mover) será hospedada na infra Real Games Studio quando a Fase 4 do projeto estiver pronta.

## Teste local

```
https://realgamesbr.github.io/campus-player-id/share.html?id=demo&img=https://example.com/sample-card.png
```

Substitui `img=` pela URL de uma imagem PNG/JPG de carta. O `?id=` vira o `photo_id` no Sheet.

## Conformidade LGPD

- Email é opcional (botão "Não quero deixar meu email")
- Consent de marketing é separado do de privacidade
- Política linkada visivelmente
- Política descreve coleta, uso, prazo, direitos do titular
- Apenas leads com `consent_marketing = TRUE` podem ser usados pra comunicação

Detalhes: ver `SETUP.md` seção "LGPD — Pontos importantes".

## Adaptado de

Padrão herdado de [`realgamesbr/jogo-da-pisa-foto`](https://github.com/realgamesbr/jogo-da-pisa-foto) (Vinhos de Portugal 2026), com visual e copy adaptados pro Campus Party.
