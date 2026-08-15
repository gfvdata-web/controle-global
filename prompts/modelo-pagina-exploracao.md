# Etapa E — Página de exploração da fonte

> **O que é:** o processo e a especificação para criar, em **qualquer** fonte do projeto, uma
> página irmã do painel — `docs/explorar.html` — que mostra **as tabelas como elas são** e uma
> **pauta analítica** discutível.
>
> **Para que serve:** essa página é o material de trabalho entre o Guilherme e a sessão do
> Claude *antes* de desenhar o painel definitivo. Ela responde duas perguntas, nessa ordem:
> 1. **O que existe aqui?** — colunas, tipos, chaves, joins, cobertura, buracos, amostra real.
> 2. **O que dá para perguntar a esses dados?** — comparativos, contexto externo, cruzamentos
>    com as outras fontes, e a lista de visualizações candidatas com status.
>
> **Como usar:** este arquivo é ao mesmo tempo a especificação e o prompt. Para rodar em uma
> fonte, copie a partir de "↓↓↓ A PARTIR DAQUI É O PROMPT ↓↓↓", preencha os `<<campos>>` e cole
> como primeira mensagem de uma sessão nova rodando **na raiz do repositório daquela fonte**.

---

## 1. Onde a Etapa E entra

Ela roda **depois da Etapa 3** (o tidy precisa existir) e **antes da Etapa 4**. Para as fontes
já construídas, ela é acrescentada por cima — sem tocar em nada que já funciona.

```
Etapa 2 (coleta) ──▶ Etapa 3 (tidy) ──▶ ETAPA E ──▶ [discussão] ──▶ Etapa 4 → 5 → 6
```

Recebe letra em vez de número (não é "3.5") para que a numeração 0→7 já escrita em todo
`CONTEXTO.md`, `README.md` e prompt existente continue valendo sem renumeração.

## 2. A regra que não se quebra

**A Etapa E só adiciona.** Ela cria arquivos novos e acrescenta *um* link à navegação do
`index.html`. Ela **não** reescreve o painel, o pipeline, o CSS existente, o tidy nem o JSON de
publicação. Se algo na Etapa E exigir mudar um arquivo já publicado além desse link, **pare e
pergunte** — não decida sozinho.

## 3. Os artefatos

| Arquivo | Quem escreve | Sobrescrito pelo pipeline? |
|---|---|---|
| `src/perfil/nucleo.py` | motor genérico, **idêntico nas três fontes** | não |
| `src/perfil/<slug>.py` → `docs/dados/perfil_<slug>.json` | **código** (pandas) | **sim**, toda rodada |
| `docs/dados/notas_<slug>.json` | **Claude, à mão** (curadoria) | **nunca** |
| `docs/explorar.html` + `docs/js/explorar.js` | Claude, uma vez | não |

`nucleo.py` faz toda a medição (tipos, nulos, distintos, estatísticas, chave primária,
duplicatas, lacunas, órfãos, amostra) e `<slug>.py` só diz **quais** tabelas existem e o que
cada coluna significa. `explorar.js` é igualmente idêntico entre as fontes — o slug vem de
`<body data-slug="...">`. Ao corrigir o núcleo, replique nas outras fontes; é a mesma
duplicação deliberada do `estilo.css`.

A Etapa E entra no `run_pipeline.py` logo depois da Etapa 3, com a flag `--sem-perfil` para
pulá-la.

A separação é o ponto central do desenho: o **perfil** é estrutura viva — envelhece junto com o
dado e se corrige sozinho a cada rodada do pipeline. As **notas** são leitura humana — pesquisa,
hipótese, decisão — e nenhum script tem permissão de apagá-las. A página junta as duas coisas no
navegador.

> **Detalhe que importa:** `dados/brutos/` não é versionado. Isso faz do `perfil_<slug>.json` o
> **único registro em git da forma do dado bruto** — o que a origem entregou, com quantas colunas
> e que tipos, na data da coleta. Trate-o como documentação, não como cache.

---

## 4. Especificação do `perfil_<slug>.json`

Gerado por `src/perfil/<slug>.py`, expondo `perfilar(slug)`, chamado pelo `run_pipeline.py`
logo após a Etapa 3.

```jsonc
{
  "meta": {
    "slug": "meios_pagamento_mensal",
    "fonte": "BCB · Estatísticas de Meios de Pagamento",
    "gerado_em": "2026-08-15T14:02:11-03:00",
    "versao_perfil": 1
  },

  "tabelas": [
    {
      "id": "bruto",                       // 'bruto' | 'tidy' | 'aux_<nome>'
      "camada": "bruto",                   // bruto | tidy | auxiliar
      "nome": "Resposta da API Olinda (OData)",
      "arquivo": "dados/brutos/meios_pagamento_mensal.json",
      "formato": "JSON (OData v4)",
      "origem": "https://olinda.bcb.gov.br/...",   // URL, ou o passo que produziu a tabela
      "n_linhas": 1746,
      "n_colunas": 4,
      "granularidade": "uma linha por mês × forma de pagamento",
      "chave_primaria": ["datainicial", "formapagamento"],
      "chave_primaria_valida": true,       // testado de fato: sem duplicatas na combinação
      "duplicatas": 0,
      "colunas": [ /* ver abaixo */ ],
      "amostra": {
        "cabecalho": ["datainicial", "formapagamento", "quantidade", "valor"],
        "inicio": [ ["2002-04-01", "TED", 12345.0, 678.9], /* … 25 linhas */ ],
        "fim":    [ /* … últimas 25 linhas */ ]
      },
      "alertas": [
        "coluna `valor` só passa a ser preenchida a partir de 2003-01 (10 meses nulos no início)"
      ]
    }
  ],

  "relacionamentos": [
    {
      "de": "tidy.ano_mes",
      "para": "aux_ipca.ano_mes",
      "cardinalidade": "N:1",
      "cobertura_pct": 100.0,
      "orfaos": 0,                          // linhas de `de` sem correspondente em `para`
      "uso": "deflação: valor_constante = valor × (ipca_base / ipca_mes)"
    }
  ],

  "cobertura_temporal": {
    "inicio": "2002-04",
    "fim": "2026-06",
    "meses_esperados": 291,
    "meses_presentes": 291,
    "lacunas": []                            // ex.: ["2013-07", "2013-08"]
  }
}
```

### Objeto coluna

```jsonc
{
  "nome": "quantidade",
  "tipo": "float64",                    // dtype do pandas, como está
  "papel": "medida",                    // medida | dimensao | data | chave | texto  (declarado, ver 4.1)
  "unidade": "milhares de transações",  // null quando não se aplica  (declarado)
  "descricao": "Volume de transações no mês",                        // (declarado)
  "nulos": 0,
  "pct_nulos": 0.0,
  "distintos": 1740,
  "armazenado_como_texto": true,        // só aparece quando a medida vem como string (SGS faz isso)

  // se numérica:
  "min": 0.4, "max": 6412330.1, "media": 118233.7, "mediana": 4120.5,
  "desvio": 512004.2, "p05": 12.3, "p95": 601233.0, "zeros": 3, "negativos": 0,

  // se categórica / texto:
  "top_valores": [ { "valor": "Pix", "n": 291, "pct": 16.7 }, /* até 10 */ ],
  "tamanho_min": 3, "tamanho_max": 7,

  // se data:
  "min_data": "2002-04", "max_data": "2026-06"
}
```

### 4.1 O que é calculado e o que é declarado

O módulo **calcula** tudo que se lê do arquivo: tipos, contagens, nulos, distintos,
estatísticas, duplicatas, amostra, lacunas de mês, órfãos de join.

O módulo **não adivinha** semântica. `papel`, `unidade` e `descricao` vêm de um dicionário
declarado no topo do próprio `src/perfil/<slug>.py`:

```python
DECLARACAO = {
    "tidy": {
        "granularidade": "uma linha por mês × forma de pagamento",
        "chave_primaria": ["ano_mes", "forma_pagamento"],
        "colunas": {
            "ano_mes":         ("data",     None,                      "Mês de referência"),
            "forma_pagamento": ("dimensao", None,                      "Instrumento de pagamento"),
            "quantidade":      ("medida",   "milhares de transações",  "Volume no mês"),
            "valor":           ("medida",   "R$ milhões",              "Montante no mês"),
        },
    },
}
```

Coluna que aparece no arquivo e **não** está na `DECLARACAO` entra no JSON com
`"papel": "nao_declarado"` e gera um alerta na tabela. Isso é proposital: quando a origem
adiciona uma coluna nova, a página avisa em vez de esconder.

### 4.2 Quais tabelas perfilar

**Bruto + tidy + auxiliares** — as três camadas.

- **bruto** — o arquivo como veio. Se a origem entrega várias tabelas (várias abas de XLSX,
  vários endpoints, várias séries), **cada uma é uma entrada** em `tabelas`. Quando são muitas
  do mesmo formato (ex.: 61 séries do SGS), perfile a estrutura de **uma** representativa e
  acrescente uma tabela auxiliar com o **inventário** das demais (código, título, unidade,
  período, nº de observações) — é isso que tem valor analítico, não 61 perfis idênticos.
- **tidy** — `dados/processados/<slug>.csv`, sempre.
- **auxiliares** — toda tabela de apoio que o pipeline usa ou constrói: o IPCA da deflação, o
  mapa código SGS → segmento × modalidade, a hierarquia de tributos (`tributo_pai`, `nivel`,
  `tipo`), tabelas de-para. **É aqui que moram os joins reais** — sem elas a seção de
  relacionamentos fica vazia e a página perde metade da graça.

### 4.3 Amostra: início + fim

25 primeiras e 25 últimas linhas, na ordem natural da tabela (para o tidy, ordenado por
`ano_mes` e depois pelas dimensões). Mostrar as duas pontas revela o que o `limit 50` puro
esconde: coluna que só passa a existir depois, categoria descontinuada, mudança de escala,
troca de metodologia no meio da série.

Limites: célula de texto truncada em 80 caracteres (com `…`); float arredondado para 4 casas no
JSON; **arquivo inteiro sob ~300 KB**. Se estourar, corte primeiro os `top_valores` para 5 e
depois a amostra para 15+15 — nunca o dicionário de colunas.

---

## 5. Especificação do `notas_<slug>.json`

Escrito à mão. Nenhum script escreve nele. É o registro da conversa entre o Guilherme e o
Claude sobre esta fonte.

```jsonc
{
  "meta": { "slug": "…", "atualizado_em": "2026-08-15" },

  "leitura": [
    { "tabela_id": "tidy", "texto": "…o que salta aos olhos nessa tabela…" }
  ],

  "armadilhas": [
    {
      "titulo": "Dupla contagem por `tipo`",
      "texto": "Somar todas as linhas infla o total: `agregado` já contém `componente`. Qualquer soma ou participação filtra `tipo = 'componente'`.",
      "impacto": "alto"                       // alto | medio | baixo
    }
  ],

  "comparativos": [
    {
      "id": "C1",
      "titulo": "Substituição TED → Pix",
      "pergunta": "Quanto do volume de TED migrou para Pix desde nov/2020?",
      "colunas": ["ano_mes", "forma_pagamento", "valor", "quantidade"],
      "o_que_esperar": "…hipótese explícita, para o gráfico poder confirmá-la ou desmenti-la…"
    }
  ],

  "contexto_externo": [
    {
      "titulo": "Pix entra em operação plena",
      "resumo": "…1 a 3 frases…",
      "numero": "16/11/2020",                 // o dado citável, ou null
      "fonte": "Banco Central do Brasil",
      "url": "https://…",
      "consultado_em": "2026-08-15",
      "por_que_importa": "marca a quebra estrutural das séries de TED e boleto"
    }
  ],

  "cruzamentos": [
    {
      "com_fonte": "credito_modalidade",
      "chave": "ano_mes",
      "pergunta": "Volume de Pix acompanha o saldo de crédito PF ou anda independente?",
      "cuidado": "períodos diferentes: crédito começa em 2011-03; recortar o início"
    }
  ],

  "pauta": [
    {
      "id": "V1",
      "titulo": "Participação das formas ao longo do tempo",
      "pergunta": "Em que mês o Pix passou o TED em valor? E em quantidade?",
      "tabela": "tidy",
      "colunas": ["ano_mes", "forma_pagamento", "valor"],
      "tipo_grafico": "área empilhada 100%",
      "por_que_esse_tipo": "a pergunta é sobre fatia, não sobre nível — empilhada 100% lê a substituição direto, sem o leitor ter que dividir de cabeça",
      "status": "proposto",                   // proposto | aprovado | descartado | no_painel
      "prioridade": "alta",                   // alta | media | baixa
      "esforco": "baixo",                     // baixo | medio | alto
      "comentario": "",                       // por que foi aprovado/descartado
      "decidido_em": null                     // "2026-08-20" quando sair de 'proposto'
    }
  ],

  "perguntas_abertas": [
    "A quebra de 2013 na série de TEC é metodológica ou real? Não achei nota técnica."
  ]
}
```

### 5.1 Regras da curadoria

**Contexto externo.** Pesquise de fato na web antes de escrever — não escreva de memória.
Prioridade das fontes: **origem oficial** (BCB, RFB, IBGE, Tesouro) → nota técnica / relatório
→ imprensa econômica estabelecida. Toda entrada carrega `url` e `consultado_em`. Se um número
não tem link, ele **não entra**. Se a pesquisa não confirmar algo, isso vira item em
`perguntas_abertas`, não vira afirmação com ressalva.

**Comparativos.** Cada um declara uma **hipótese** em `o_que_esperar`. Um comparativo cujo
resultado não muda nada no entendimento não vale um cartão na página.

**Pauta.** Todo item começa em `proposto`. Só o Guilherme move para `aprovado` ou `descartado`,
e a mudança é registrada com `comentario` e `decidido_em`. `no_painel` marca o que já foi
construído na Etapa 6 — é assim que a página vira o backlog visível do painel definitivo, e não
uma lista de ideias soltas que ninguém sabe se valem.

**Cruzamentos.** A chave comum entre todas as fontes do projeto é `ano_mes`. Todo cruzamento
declara o `cuidado` — período diferente, unidade diferente, nominal vs. real, nível vs. fluxo.

---

## 6. Especificação da página `docs/explorar.html`

Mesmo CSS, mesmo cabeçalho, mesma linguagem visual do painel. O que muda é o conteúdo — e é
isso que **precisa** mudar por fonte: o esqueleto é comum, os cartões não.

**Navegação.** O `nav-paineis` do cabeçalho ganha um link para cada painel *e* o par
Painel/Explorar da fonte atual:

```html
<nav class="nav-paineis" aria-label="Painéis do projeto">
  <a href="index.html">Painel</a>
  <a href="explorar.html" aria-current="page">Explorar dados</a>
  <span class="nav-sep"></span>
  <a href="https://gfvdata-web.github.io/fonte-arrecadacao-federal/">Arrecadação federal</a>
  <a href="https://gfvdata-web.github.io/fonte-credito-modalidade/">Crédito por modalidade</a>
</nav>
```

No `index.html`, a **única** alteração permitida é acrescentar `<a href="explorar.html">Explorar
dados</a>` a esse mesmo `nav` (e o `aria-current` passa a ficar no "Painel").

### Seções, na ordem

1. **Resumo da fonte** — grade de `.kpi`: nº de tabelas perfiladas, período coberto, linhas no
   tidy, periodicidade, nº de dimensões × medidas, data da última coleta, licença.
2. **Mapa das tabelas** — um cartão com as tabelas agrupadas em três colunas
   (bruto → tidy → auxiliar) e, abaixo, a tabela de **relacionamentos**: de, para,
   cardinalidade, cobertura %, órfãos, uso. Chave primária validada ganha selo verde;
   `chave_primaria_valida: false` ganha selo vermelho e vai para o topo.
3. **Cobertura temporal** — barra do período com as lacunas marcadas, ou a frase
   "291 de 291 meses presentes, sem lacunas".
4. **Uma seção por tabela** (`<details>` fechado por padrão, exceto o tidy, aberto):
   - linha de metadados: formato, arquivo, linhas × colunas, granularidade, chave primária;
   - **dicionário de colunas** — `.tabela` com nome (+ selo de papel), tipo, unidade, % nulos,
     distintos, e a estatística conforme o tipo (min/mediana/máx para número; top-3 valores para
     categoria; período para data);
   - **amostra** — `.tabela` dentro de `.tabela-wrap`, com as 25 primeiras linhas, uma linha
     separadora `⋯ N linhas omitidas ⋯` e as 25 últimas. Cabeçalho com o nome da coluna e o selo
     de tipo. Números alinhados à direita (`.num`), nulos como `—` em `.nulo`;
   - **alertas** da tabela, se houver;
   - **leitura** curada daquela tabela, vinda das notas.
5. **Armadilhas** — cartões ordenados por impacto. Só aparece se `armadilhas` não estiver vazio.
6. **Comparativos possíveis** — um cartão por item: pergunta em destaque, colunas envolvidas como
   selos, hipótese em `o_que_esperar`.
7. **Contexto externo** — lista com título, resumo, número citável, e o link com a data de
   consulta. Todo item é clicável até a origem.
8. **Cruzamento com outras fontes** — um cartão por fonte relacionada, com a pergunta e o
   cuidado. Linka o painel da outra fonte.
9. **Pauta de visualizações** — o núcleo discutível. `.tabela` com id, título, pergunta, tipo de
   gráfico, prioridade, esforço e **status como selo colorido**. Filtro por status no topo
   (`.alternador` com Todos / Proposto / Aprovado / Descartado / No painel), reaproveitando o
   componente que já existe no CSS.
10. **Perguntas abertas** — lista simples. É o que ainda não se sabe.
11. **Rodapé** — data de geração do perfil, data de atualização das notas, licença da fonte.

### Regras de implementação

- `docs/js/explorar.js` carrega os **dois** JSON com `Promise.all` e degrada com elegância: se
  `notas_<slug>.json` não existir ou estiver vazio, as seções 5→10 não são renderizadas e a
  página segue funcionando só com o perfil. O contrário não vale — sem perfil, a página mostra
  o erro.
- **Sem Chart.js aqui.** Esta página é sobre a estrutura do dado; gráfico é assunto do painel.
  A única exceção é a barra de cobertura temporal, que é HTML/CSS puro.
- CSS novo vai **anexado ao fim** de `docs/css/estilo.css`, sob o comentário
  `/* ===== Etapa E — página de exploração ===== */`. Nada acima disso é editado. Classes novas
  esperadas: `.selo`, `.selo-papel`, `.selo-status`, `.nav-sep`, `.mapa-tabelas`, `.barra-cobertura`,
  `.linha-omitida`, `.cartao-pauta`.
- Reaproveite o que já existe: `.conteiner`, `.cabecalho`, `.nav-paineis`, `.kpi`, `.cartao`,
  `.cartao-cabecalho`, `.tabela`, `.tabela-wrap`, `.num`, `.nulo`, `.alternador`, `.alt-btn`,
  `.rodape`. Os tokens de cor (`--acento`, `--positivo`, `--negativo`, `--texto-suave`) já
  cobrem tema claro e escuro — use-os em vez de cravar cor.
- Português em toda a interface. Números no formato pt-BR (`Intl.NumberFormat('pt-BR')`).

---

## 7. O ciclo de discussão

A página não é um entregável que termina — é onde a conversa fica registrada.

1. A sessão gera o perfil e escreve a **primeira versão** das notas, com toda a pauta em
   `proposto`.
2. O Guilherme lê a página publicada e responde: aprova, descarta, corrige, acrescenta.
3. A sessão edita **só** `notas_<slug>.json` — muda `status`, preenche `comentario` e
   `decidido_em`, adiciona itens novos. Nunca apaga um item descartado: `descartado` com o motivo
   escrito vale mais do que a ausência, porque impede que a mesma ideia volte daqui a três meses.
4. Quando a Etapa 6 constrói um item, ele vira `no_painel`.

Ao fim de um ciclo, o que estiver em `aprovado` **é** a especificação do painel definitivo.

---

## 8. As três fontes já feitas — o que cada uma ensinou

Feitas em 15/08/2026, nesta ordem. Para uma fonte nova, copie o `explorar.html`, o
`explorar.js`, o bloco de CSS e o `nucleo.py` da que mais se parecer com ela.

| Fonte | Tabelas | Joins | O que o perfil revelou |
|---|---|---|---|
| [`fonte-meios-pagamento`](https://gfvdata-web.github.io/fonte-meios-pagamento/explorar.html) | 3 | 2 | TEC e DOC sem movimento desde fev/2024; Pix presente em 68 dos 291 meses. A **referência**: é dela que as outras copiaram a página. |
| [`fonte-arrecadacao-federal`](https://gfvdata-web.github.io/fonte-arrecadacao-federal/explorar.html) | 5 | 4 | O rótulo do XLSX **não é chave** — 'ENTIDADES FINANCEIRAS' e 'DEMAIS EMPRESAS' aparecem 3× cada. Auto-relacionamento `tributo_pai`→`tributo` íntegro. Série para em dez/2025 **por limitação da RFB**, verificado na origem. |
| [`fonte-credito-modalidade`](https://gfvdata-web.github.io/fonte-credito-modalidade/explorar.html) | 4 | 3 | `valor` vem do SGS como texto; `segmento`/`modalidade`/`medida` **não vêm da API** (são anotação da Etapa 2); as 61 séries têm cobertura idêntica, sem viés de janela. |

Três coisas que só apareceram ao implementar e que valem para a próxima fonte:

1. **Nulo estrutural é a regra, não a exceção.** As três fontes têm colunas com nulos ou zeros
   que significam "não existia" / "não se aplica", não "faltou dado". O perfil os reporta como
   percentual e a nota é quem explica — essa divisão de trabalho funcionou e deve ser mantida.
2. **A chave primária tem que ser testada, não declarada.** A única PK inválida das três fontes
   estava no bruto da arrecadação, e ninguém teria notado sem medir.
3. **Junte a pesquisa externa ao que o perfil mediu.** O caso exemplar: o perfil detectou zeros
   em TEC/DOC a partir de mar/2024 e a pesquisa confirmou a data do encerramento na Febraban.
   Medida + fonte externa transforma uma anomalia numa explicação.

Em qualquer retrofit, o `index.html`, o `app.js`, o tidy e o JSON de publicação ficam **exatamente
como estão**, exceto o link novo na navegação.

---

# ↓↓↓ A PARTIR DAQUI É O PROMPT — copie, preencha e cole ↓↓↓

# Etapa E — Página de exploração: `<<slug>>`

## Contexto

Este repositório é um braço do projeto *Dados Financeiros Abertos (Brasil)*: uma fonte por
repositório, com pipeline em Python e painel estático em GitHub Pages.

Vou pedir a **Etapa E**: uma página irmã do painel, `docs/explorar.html`, que mostra as tabelas
desta fonte como elas são e propõe uma pauta analítica. Ela **só adiciona** — não reescreve nada
do que já existe.

**Leia, nesta ordem, só isto:**
1. `CONTEXTO.md` deste repositório — etapas, contrato tidy, convenções.
2. `catalogo/fonte.md` — o dicionário de dados já escrito. **Não o duplique**: a página de
   exploração mostra o dado medido; o `fonte.md` documenta a origem. Onde os dois divergirem,
   o dado medido manda, e a divergência vira alerta.
3. `src/config.py`, `src/tratamento/<<slug>>.py` e `run_pipeline.py` — para saber o que a Etapa 3
   faz e quais tabelas auxiliares existem.
4. `docs/index.html` e `docs/css/estilo.css` — para reaproveitar cabeçalho, navegação e classes.
5. `<<se já houver uma fonte com explorar.html pronto: docs/explorar.html e docs/js/explorar.js
   dela, como modelo>>`

Não leia o repositório `controle-global` inteiro. A especificação completa desta etapa está em
`controle-global/prompts/modelo-pagina-exploracao.md`, seções 3 a 7 — **leia essas seções**; elas
definem o formato exato dos dois JSON e as seções da página.

## O que fazer

### E.1 — Perfilar (código)
Criar `src/perfil/__init__.py` e `src/perfil/<<slug>>.py`, expondo `perfilar(slug)`, que lê o
bruto, o tidy e as auxiliares e gera `docs/dados/perfil_<<slug>>.json` no formato da seção 4.
Declarar `papel`, `unidade` e `descricao` de cada coluna no dicionário `DECLARACAO` do módulo
(seção 4.1). Registrar a chamada no `run_pipeline.py`, **depois** da Etapa 3.

Tabelas a perfilar nesta fonte: `<<bruto: … | tidy | auxiliares: …>>`

Rodar e conferir: chave primária validada de verdade, lacunas de mês detectadas, órfãos de join
contados, arquivo abaixo de 300 KB.

### E.2 — Curar (à mão)
Escrever `docs/dados/notas_<<slug>>.json` no formato da seção 5. Antes de escrever
`contexto_externo`, **pesquise na web** — origem oficial primeiro, todo número com `url` e
`consultado_em`; o que não confirmar vira `perguntas_abertas`. Toda a `pauta` nasce em `proposto`.

Ângulos que já sei que me interessam nesta fonte: `<<ex.: crescimento do Pix e substituição do
TED; efeito da Selic sobre o spread; sazonalidade de dezembro na arrecadação>>`

### E.3 — Publicar
`docs/explorar.html` + `docs/js/explorar.js` conforme a seção 6, com o CSS novo **anexado ao fim**
de `docs/css/estilo.css`. Acrescentar o link "Explorar dados" ao `nav-paineis` do `index.html` —
essa é a única alteração permitida em arquivo existente.

### E.4 — Documentar
Registrar a Etapa E no `CONTEXTO.md` e no `README.md` deste repositório.

## Ao terminar

Me mostre a página rodando local e me diga:
- o que o perfil revelou que **não estava** no `catalogo/fonte.md` (coluna a mais, nulo estrutural,
  lacuna, chave que não é única, divergência de unidade);
- os itens da `pauta`, numerados, para eu aprovar ou descartar um a um;
- o que ficou em `perguntas_abertas` e o que você tentou para resolvê-las.
