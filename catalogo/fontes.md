# Catálogo de Fontes Implementadas

> Índice das fontes que já têm pipeline e dashboard. **O dicionário de dados completo de
> cada fonte vive no repositório dela**, em `catalogo/fonte.md` — este arquivo só diz o que
> cada fonte cobre e onde encontrá-la.
>
> Fontes ainda não implementadas: [`fontes-candidatas.md`](fontes-candidatas.md).

## Resumo

| Slug | Órgão | Tema | Período | Freq. | Geo | Acesso | Etapa E |
|------|-------|------|---------|-------|-----|--------|---------|
| `meios_pagamento_mensal` | BCB | Pagamentos | abr/2002 → | Mensal | ⚪ nacional | API Olinda/OData | ✅ [página](https://gfvdata-web.github.io/fonte-meios-pagamento/explorar.html) |
| `arrecadacao_federal` | RFB | Tributos | jan/1994 → | Mensal | ⚪ nacional | Download XLSX | ✅ [página](https://gfvdata-web.github.io/fonte-arrecadacao-federal/explorar.html) |
| `credito_modalidade` | BCB/SGS | Crédito | mar/2011 → | Mensal | ⚪ nacional | API SGS (61 séries) | ✅ [página](https://gfvdata-web.github.io/fonte-credito-modalidade/explorar.html) |

> **Etapa E** = página `explorar.html` (perfil das tabelas + pauta analítica). Especificação em
> [`../prompts/modelo-pagina-exploracao.md`](../prompts/modelo-pagina-exploracao.md); ordem de
> implementação na seção 8 de lá, que também registra o que o perfil de cada fonte revelou.

---

## `meios_pagamento_mensal` — BCB · Estatísticas de Meios de Pagamento (mensal)

- **Repositório:** `fonte-meios-pagamento`
- **Painel:** https://gfvdata-web.github.io/fonte-meios-pagamento/
- **Explorar (Etapa E):** https://gfvdata-web.github.io/fonte-meios-pagamento/explorar.html
- **Dicionário completo:** `fonte-meios-pagamento/catalogo/fonte.md`
- **O que cobre:** quantidade e valor de movimentação por mês e forma de pagamento
  (Pix, TED, TEC, Cheque, Boleto, DOC) no Brasil.
- **Tidy:** `ano_mes`, `forma_pagamento` → `quantidade` (milhares), `valor` (R$ milhões).
- **Licença:** Dados abertos — Banco Central do Brasil.
- **Papel no projeto:** fonte inicial e **modelo de referência** de código/estilo para as
  demais. Foi a partir dela que as Etapas 2→6 ganharam sua forma atual.

## `arrecadacao_federal` — RFB · Arrecadação das Receitas Federais (série histórica)

- **Repositório:** `fonte-arrecadacao-federal`
- **Painel:** https://gfvdata-web.github.io/fonte-arrecadacao-federal/
- **Explorar (Etapa E):** https://gfvdata-web.github.io/fonte-arrecadacao-federal/explorar.html
- **Dicionário completo:** `fonte-arrecadacao-federal/catalogo/fonte.md`
- **O que cobre:** valor arrecadado mensal por tributo no Brasil, 1994 →, a preços correntes
  e constantes.
- **Tidy:** `ano_mes`, `tributo` (+ `rotulo`, `tributo_pai`, `nivel`, `tipo`) →
  `valor`, `valor_constante` (R$ milhões).
- **Licença:** CC-BY-ND 3.0 — Receita Federal do Brasil.
- **Primeiras do projeto:** primeira fonte por **download de arquivo** (XLSX, não API) e
  primeira com **deflação por IPCA** (BCB/SGS 433) embutida no pipeline.
- **Atenção transversal:** a coluna `tipo` (`componente`/`detalhe`/`agregado`) evita dupla
  contagem — qualquer soma ou participação filtra por `tipo = componente`.

## `credito_modalidade` — BCB · Crédito por modalidade (SGS)

- **Repositório:** `fonte-credito-modalidade`
- **Painel:** https://gfvdata-web.github.io/fonte-credito-modalidade/
- **Explorar (Etapa E):** https://gfvdata-web.github.io/fonte-credito-modalidade/explorar.html
- **Dicionário completo:** `fonte-credito-modalidade/catalogo/fonte.md`
- **O que cobre:** saldo da carteira, taxa média de juros e spread das operações de crédito
  do SFN, por modalidade e segmento (PF/PJ), mar/2011 →.
- **Tidy:** `ano_mes`, `segmento`, `modalidade_credito` → `saldo` (R$ milhões),
  `taxa_juros_aa` (% a.a.), `spread_pp` (p.p. a.a., só nas linhas `Total`).
- **Licença:** Open Database License (ODbL) — Banco Central do Brasil.
- **Primeiras do projeto:** primeira fonte **multi-série** (61 códigos do SGS, uma
  requisição cada) e primeira com **duas dimensões** de recorte.
- **Verificação útil:** o SGS **não** publica crédito por UF por modalidade (só por porte de
  empresa) — por isso esta fonte é nacional. Registrado para não se reinvestigar.

---

## Fora do escopo financeiro

Projetos da conta `gfvdata-web` que seguem o mesmo modelo (repositório autocontido, pipeline
em Python, página estática no GitHub Pages), mas **não** são dados financeiros públicos — por
isso ficam fora da tabela **Resumo** e da Etapa E. Estão aqui só para constar no índice.

### `chess_tracking` — Chess.com · Histórico de partidas

- **Repositório:** [`chess-tracking`](https://github.com/gfvdata-web/chess-tracking)
- **Página:** https://gfvdata-web.github.io/chess-tracking/
- **O que cobre:** todas as partidas de uma conta do Chess.com (hoje `giggsmate`), jun/2018 →,
  com estatísticas de rating, resultado por cor, ritmo, abertura, horário, força do
  adversário, como as partidas terminam, sequências, atividade e gestão de tempo.
- **Tidy:** uma linha por partida — `data`, `hora`, `ritmo`, `cor`, `adversario` →
  `meu_rating`, `adv_rating`, `resultado`, `motivo`, `eco`/`abertura`, `lances`, `precisao`
  e tempos por fase (`t_abertura`, `t_meio`, `t_final`).
- **Acesso:** API pública do Chess.com (PubAPI), arquivos mensais por usuário.
- **Atualização:** GitHub Actions diário (06:17 em Brasília), coleta incremental — só os
  meses novos e o mês corrente; commit apenas se houver partida nova.
- **Monitoramento:** card no [painel de status](https://gfvdata-web.github.io/painel-status/).

---

## Como registrar uma fonte nova aqui

Ao concluir uma fonte no repositório dela, volte a este arquivo e adicione:
1. Uma linha na tabela **Resumo**.
2. Uma seção com repositório, painel, o que cobre, tidy, licença e o que ela trouxe de novo
   para o projeto (padrões inaugurados, verificações que não precisam ser refeitas).
3. Marque a fonte como implementada em [`fontes-candidatas.md`](fontes-candidatas.md) e
   atualize a tabela do [`README.md`](../README.md).
