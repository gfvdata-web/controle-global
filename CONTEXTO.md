# CONTEXTO GLOBAL — Dados Financeiros Abertos (Brasil)

> Arquivo-mestre do projeto como um todo. Define **o que vale para todas as fontes**:
> arquitetura de repositórios, as Etapas, o contrato de dados e as convenções.
>
> O que é específico de uma fonte (endpoint, colunas, particularidades) vive no
> `CONTEXTO.md` e no `catalogo/fonte.md` **do repositório daquela fonte** — nunca aqui.

---

## 1. Visão geral

Projeto de **exploração de dados financeiros públicos brasileiros**, com consumo via API
(ou download de arquivo), tratamento e **análise estatística**, culminando em **páginas
HTML + CSS** com dashboards analíticos.

- **Fontes:** Banco Central do Brasil (BCB), Receita Federal (RFB), Portal de Dados Abertos
  e demais fontes brasileiras oficiais, abertas e legalizadas.
- **Enfoque:** analítico e estatístico (séries temporais, participação, crescimento,
  distribuições, comparações).

## 2. Objetivos

1. Criar uma base reutilizável para incorporar **novas fontes** com o mínimo de atrito.
2. Manter, dentro de cada fonte, um **pipeline integrado**
   (coleta → tratamento → análise → publicação → visualização) em que cada etapa consome a
   saída padronizada da anterior.
3. Publicar **dashboards estáticos** (GitHub Pages) que consomem JSON gerado pelo pipeline.

## 3. Arquitetura: um repositório por fonte

```
controle-global/              ← este repositório (só documentação)
├── CONTEXTO.md               ← visão global, etapas, contrato, convenções
├── README.md                 ← índice das fontes + links dos painéis
├── GUIA-REPOSITORIOS.md      ← estrutura de pastas e o que vai em cada uma
├── catalogo/
│   ├── fontes.md             ← índice das fontes implementadas
│   └── fontes-candidatas.md  ← radar de fontes ainda não implementadas
└── prompts/
    └── modelo-fonte-nova.md  ← modelo de prompt para abrir uma fonte nova

fonte-meios-pagamento/        ← repositório próprio (pipeline + dashboard + Pages)
fonte-arrecadacao-federal/    ← repositório próprio
fonte-credito-modalidade/     ← repositório próprio
fonte-<proxima>/              ← cada fonte nova entra assim
```

**Por que assim:**
- Cada fonte é **autocontida**: pipeline, dados, dashboard e deploy próprios. Uma sessão
  trabalhando na fonte X nunca toca no repositório da fonte Y.
- Fontes podem ser construídas e atualizadas **em paralelo**, sem conflito.
- O contexto que uma sessão precisa ler é pequeno: o `CONTEXTO.md` daquela fonte, não a
  história de todas as outras.

**O custo aceito:** o CSS e a estrutura das páginas são **duplicados** entre os repositórios.
É uma cópia deliberada — cada painel pode divergir sem quebrar os outros. Se um dia a
divergência incomodar, a solução é versionar um tema comum, não voltar ao monorepo.

## 4. Stack tecnológica

| Camada | Tecnologia |
|--------|-----------|
| Coleta / tratamento / análise | **Python 3.13** (`requests`, `pandas`; `openpyxl` quando a fonte é XLSX) |
| Camada de publicação | JSON estático gerado pelo Python |
| Front-end / dashboards | **HTML + CSS + JavaScript** com **Chart.js** (via CDN) |
| Hospedagem | **GitHub Pages** (pasta `/docs` de cada repositório de fonte) |
| Versionamento | Git + GitHub (`gfvdata-web`) |

## 5. As Etapas

Cada etapa tem número e nome fixos, iguais em toda fonte. Use-os como referência nos prompts.

| Etapa | Nome | Onde vive | Entrada → Saída |
|-------|------|-----------|-----------------|
| 0 | Fundação & infraestrutura | raiz do repo | Estrutura de pastas, ambiente Python, `config.py`, git + Pages |
| 1 | Catálogo da fonte | `catalogo/fonte.md` | — → dicionário de dados (URL, colunas, unidades, licença) |
| 2 | Ingestão / coleta | `src/coleta/` | endpoint/arquivo → bruto em `dados/brutos/` |
| 3 | Tratamento & modelagem | `src/tratamento/` | bruto → **CSV tidy** em `dados/processados/` |
| 4 | Análise exploratória & estatística | `src/analise/` | CSV tidy → métricas (participação, YoY, CAGR, descritivas) |
| 5 | Camada de publicação | `src/publicacao/` | tidy + métricas → **JSON** em `docs/dados/` |
| 6 | Dashboards & visualização | `docs/` | JSON → site interativo (KPIs, gráficos, tabelas) |
| 7 | Documentação & deploy | `README.md`, GitHub Pages | — → site no ar e docs atualizadas |

## 6. Contrato de dados (vale para toda fonte)

```
[origem]  ──Etapa 2──▶  dados/brutos/<slug>.*
                             │
                        ──Etapa 3──▶  dados/processados/<slug>.csv   (tidy)
                             │
            ┌────────────────┴─────────────────┐
       ──Etapa 4──▶ métricas          ──Etapa 5──▶ docs/dados/<slug>.json
                                             │
                                        ──Etapa 6──▶ docs/index.html
```

**A forma do tidy é o contrato:** **dimensões em linha, medidas em coluna**, sempre com
`ano_mes` (`YYYY-MM`) como primeira dimensão. O que muda entre fontes são *quais* colunas
de dimensão e de medida existem — não a forma.

| Fonte | Dimensões | Medidas |
|-------|-----------|---------|
| `meios_pagamento_mensal` | `ano_mes`, `forma_pagamento` | `quantidade`, `valor` |
| `arrecadacao_federal` | `ano_mes`, `tributo` (+ hierarquia: `rotulo`, `tributo_pai`, `nivel`, `tipo`) | `valor`, `valor_constante` |
| `credito_modalidade` | `ano_mes`, `segmento`, `modalidade_credito` | `saldo`, `taxa_juros_aa`, `spread_pp` |

Nenhuma fonte implementada tem recorte geográfico ainda. Quando a primeira tiver (o Pix por
município é a candidata natural), a coluna `uf` — e possivelmente `cod_municipio` — entra
como mais uma dimensão, **sem afetar** as fontes que não a têm. Chave geográfica padrão do
projeto: **código IBGE de município (7 dígitos)**.

## 7. Convenções

- **Slug da fonte:** minúsculas com underscore, ex.: `meios_pagamento_mensal`. O mesmo slug
  nomeia os módulos em `src/`, o CSV em `dados/processados/` e o JSON em `docs/dados/`.
- **Nome do repositório:** `fonte-<slug-com-hifen>`, ex.: `fonte-meios-pagamento`.
- **Datas:** sempre normalizadas para string `YYYY-MM`.
- **JSON do front:** sempre com bloco `meta` (fonte, url, gerado_em, unidades, período).
- **Deflação:** quando uma fonte precisar de valores reais, o padrão é **IPCA** (BCB/SGS
  série 433), com base no último mês da série. Implementado hoje em `arrecadacao_federal`.
- **Idioma do código:** nomes de funções/variáveis e comentários em português.
- **Página do painel:** sempre `docs/index.html` + `docs/js/app.js` + `docs/css/estilo.css`
  (cada repositório tem um painel só, então não há nomes por fonte dentro de `docs/`).

## 8. Princípios de trabalho

- **Integração acima de tudo:** cada etapa tem contrato de entrada/saída definido (seção 6).
  Alterações devem respeitar esses contratos.
- **Não quebrar:** ao ajustar uma etapa, verificar as vizinhas. O `run_pipeline.py` da fonte
  deve continuar rodando de ponta a ponta.
- **Reprodutibilidade:** qualquer JSON publicado deve ser regenerável rodando o pipeline.
- **Escopo do repositório:** trabalho sobre uma fonte acontece **no repositório dela**. Este
  repositório só é tocado quando a mudança é global (uma convenção nova, uma fonte
  registrada, uma candidata promovida).

## 9. Roadmap global

- [ ] Implementar a **onda 1** de candidatas: Pix (BCB, com recorte municipal), SGS macro,
      Meios de Pagamento trimestral (cartões). Ver
      [`catalogo/fontes-candidatas.md`](catalogo/fontes-candidatas.md).
- [ ] Estender o contrato tidy com dimensão geográfica quando a primeira fonte com UF/município
      entrar (seção 6).
- [ ] Padrão de Etapa 4 mais rico em todas as fontes: sazonalidade, médias móveis, testes de
      tendência.
- [ ] Etapa 7: automação de atualização agendada (CI) por repositório.
- [ ] Avaliar um tema CSS versionado e compartilhado, se a divergência visual entre painéis
      começar a incomodar.
