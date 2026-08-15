# Modelo de prompt — abrir uma fonte nova

> **Como usar:** copie este arquivo, preencha os `<<campos>>`, e cole o resultado inteiro
> como **primeira mensagem** de uma sessão nova do Claude Code (modelo Opus), rodando na
> raiz do repositório novo `fonte-<<slug-com-hifen>>`.
>
> Guarde a versão preenchida em `prompts/` **do repositório da fonte** — ela vira o registro
> de como aquela fonte foi construída.

## Antes de colar o prompt

1. Escolher a fonte em [`../catalogo/fontes-candidatas.md`](../catalogo/fontes-candidatas.md).
2. Criar a pasta/repositório `fonte-<<slug-com-hifen>>` com a estrutura de
   [`../GUIA-REPOSITORIOS.md`](../GUIA-REPOSITORIOS.md).
3. Copiar para lá, como ponto de partida, os arquivos de uma fonte já pronta que se pareça
   com a nova (API JSON → use `fonte-meios-pagamento`; arquivo XLSX/CSV → use
   `fonte-arrecadacao-federal`; muitas séries → use `fonte-credito-modalidade`).

## Princípio que faz esse padrão funcionar

**A sessão lê pouco.** O prompt aponta exatamente o que precisa ser lido, e nada além disso.
Nunca peça para ler o catálogo global inteiro nem o `CONTEXTO.md` de outras fontes — o
repositório da fonte nova já é autocontido, e é isso que mantém o contexto pequeno.

---

# ↓↓↓ A PARTIR DAQUI É O PROMPT — copie, preencha e cole ↓↓↓

# Nova fonte: <<Nome da fonte>>

## Contexto (leia antes de agir)

Este repositório é **um braço** do projeto *Dados Financeiros Abertos (Brasil)*: cada fonte
de dados tem seu próprio repositório, autocontido, com pipeline (Python) e dashboard
(HTML/CSS/JS + Chart.js publicado no GitHub Pages).

**Leia, nesta ordem, só o seguinte:**
1. `CONTEXTO.md` deste repositório — as Etapas 0→7, o contrato tidy e as convenções.
   (Se ele ainda não existir, use como modelo o `CONTEXTO.md` do repositório de referência
   citado abaixo e adapte para esta fonte.)
2. O repositório de referência **`<<fonte-meios-pagamento | fonte-arrecadacao-federal |
   fonte-credito-modalidade>>`**, como modelo de código, nomenclatura e estilo:
   `src/config.py`, `src/coleta/`, `src/tratamento/`, `src/analise/`, `src/publicacao/`,
   `run_pipeline.py`, `docs/index.html` e `docs/js/app.js`.

Não leia o repositório `controle-global` inteiro — se precisar de contexto global, o único
arquivo relevante é o `CONTEXTO.md` de lá (seções 5 a 7: Etapas, contrato de dados e
convenções).

## A fonte

- **Slug:** `<<slug_com_underscore>>`
- **Órgão:** <<BCB / RFB / Tesouro / IBGE / ...>>
- **Tema:** <<o que a série mede>>
- **Acesso:** <<API JSON (URL) | download de arquivo (URL) | API com N séries>>
- **Periodicidade:** <<mensal / trimestral / diária>>
- **Recorte geográfico:** <<nacional / UF / município>>
- **Ponto de partida no catálogo:** item <<A1 / B2 / ...>> de `fontes-candidatas.md`
  (cole aqui o trecho do item, para a sessão não precisar abrir o catálogo global)

<<Cole aqui o item do catálogo de candidatas.>>

## Objetivo desta sessão

Construir a fonte de ponta a ponta, Etapas 1→3, **Etapa E**, 4→7, com o mesmo padrão de qualidade
das fontes já implementadas. A Etapa E é um ponto de parada obrigatório: o painel das Etapas 4 e 6
só é desenhado depois que a pauta de exploração for discutida e aprovada.

### Etapa 1 — Confirmar a fonte e documentar
- Confirmar na origem: endpoint/arquivo exato, parâmetros, colunas, unidades, período
  coberto, licença. <<Liste aqui o que o catálogo marcou como "a verificar".>>
- Escrever `catalogo/fonte.md` deste repositório com o dicionário de dados completo
  (formato original + formato tidy resultante).

### Etapa 2 — Coleta (`src/coleta/<<slug>>.py`)
- Registrar a fonte em `src/config.py` (dicionário `FONTES`, um slug só neste repositório).
- Salvar o dado bruto **sem transformar** em `dados/brutos/`, com envelope de metadados
  (url consultada, data/hora da coleta, contagem de registros).
- Expor `coletar(slug)`.

### Etapa 3 — Tratamento (`src/tratamento/<<slug>>.py`)
- Converter para **tidy**: dimensões em linha, medidas em coluna, `ano_mes` (`YYYY-MM`) como
  primeira dimensão. Dimensões previstas: <<...>>. Medidas: <<...>>.
- Salvar CSV em `dados/processados/<<slug>>.csv`. Expor `tratar(slug)`.
- Se a fonte precisar de valores reais, deflacionar por **IPCA** (BCB/SGS série 433), com
  base no último mês da série — é a convenção do projeto.

### Etapa E — Exploração (`src/perfil/<<slug>>.py`, `docs/explorar.html`)
- **Pare aqui e me chame.** Antes de decidir métricas e gráficos, quero ver o que a fonte tem.
- Gerar `docs/dados/perfil_<<slug>>.json` (perfil medido de bruto + tidy + auxiliares) e escrever
  `docs/dados/notas_<<slug>>.json` (armadilhas, comparativos, contexto externo pesquisado,
  cruzamentos, pauta de visualizações). Publicar em `docs/explorar.html`.
- Especificação completa: `controle-global/prompts/modelo-pagina-exploracao.md`, seções 3 a 7.
- A pauta aprovada nessa conversa **é** o escopo das Etapas 4 e 6 abaixo — não invente cartões e
  gráficos antes dela.

### Etapa 4 — Análise (`src/analise/<<slug>>.py`)
- Estatística descritiva (média, mediana, desvio, coef. de variação, mín/máx), participação
  (%), crescimento YoY, CAGR, rankings. <<Métricas específicas desta fonte, se houver.>>

### Etapa 5 — Publicação (`src/publicacao/<<slug>>.py`)
- Consolidar dados + métricas em JSON compacto em `docs/dados/<<slug>>.json`, sempre com
  bloco `meta` (fonte, url, gerado_em, unidades, período). Expor `publicar(slug)`.

### Etapa 6 — Dashboard (`docs/`)
- `docs/index.html` + `docs/js/app.js` + `docs/css/estilo.css` (copiados do repositório de
  referência e adaptados). **Os cartões e gráficos são os aprovados na Etapa E** — o esqueleto
  (cabeçalho, paleta, componentes) é comum a todas as fontes, a escolha do que mostrar não é.
- Na navegação do cabeçalho, linkar `explorar.html` desta fonte e os painéis das outras fontes
  por URL absoluta (`https://gfvdata-web.github.io/fonte-<<outra>>/`), já que cada uma tem seu
  próprio site.
- Marcar como `no_painel`, em `docs/dados/notas_<<slug>>.json`, cada item da pauta construído.

### Etapa 7 — Documentação e deploy
- `README.md` (como rodar, estrutura, licença) e `CONTEXTO.md` deste repositório.
- `run_pipeline.py` rodando de ponta a ponta.
- Publicar via GitHub Pages (branch `main`, pasta `/docs`).

## Ao terminar

Me avise o que registrar no repositório `controle-global`:
- a linha para a tabela de `catalogo/fontes.md` e a seção-resumo da fonte;
- o que esta fonte trouxe de **novo** para o projeto (padrão inaugurado, verificação feita
  que não precisa ser refeita, limitação descoberta na origem);
- se algo aqui obrigou a **estender o contrato tidy** global (ex.: primeira dimensão
  geográfica), para eu atualizar o `CONTEXTO.md` global.
