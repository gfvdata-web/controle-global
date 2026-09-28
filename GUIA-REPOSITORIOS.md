# Guia de repositórios — o que criar e o que vai em cada pasta

> Como o projeto está organizado no disco e no GitHub, e o passo a passo para publicar os
> repositórios novos. Referência prática; a justificativa da arquitetura está na seção 3 do
> [CONTEXTO.md](CONTEXTO.md).

## O mapa

Tudo mora em `C:\Users\Guilherme\Documents\ClaudeCode\`, um repositório Git por pasta:

| Pasta local | Repositório GitHub | O que é | GitHub Pages |
|---|---|---|---|
| `controle-global/` | `controle-global` | Só documentação: visão global, catálogos, modelo de prompt | não |
| `fonte-meios-pagamento/` | `fonte-meios-pagamento` | Pipeline + painel — BCB, meios de pagamento | sim (`/docs`) |
| `fonte-arrecadacao-federal/` | `fonte-arrecadacao-federal` | Pipeline + painel — RFB, arrecadação | sim (`/docs`) |
| `fonte-credito-modalidade/` | `fonte-credito-modalidade` | Pipeline + painel — BCB/SGS, crédito | sim (`/docs`) |
| `DadosFinanceirosBancoCentral/` | idem | **Repositório antigo**, monolítico — **arquivado** (só-leitura); ver "O que fazer com o antigo" | sim (congelado) |

## Anatomia de um repositório de fonte

Todos os três seguem exatamente esta forma — e toda fonte nova deve segui-la também:

```
fonte-<slug-com-hifen>/
├── CONTEXTO.md              # etapas, contrato e convenções DESTA fonte
├── README.md                # como rodar, estrutura, link do painel, licença
├── requirements.txt         # só o que esta fonte usa
├── run_pipeline.py          # orquestra Etapas 2→5 (sem argumento de slug: é uma fonte só)
├── .gitignore
├── .claude/launch.json      # config do preview local (python -m http.server em docs/)
├── catalogo/
│   └── fonte.md             # Etapa 1 — dicionário de dados desta fonte
├── src/
│   ├── __init__.py
│   ├── config.py            # caminhos + registro da fonte (uma entrada em FONTES)
│   ├── coleta/<slug>.py         # Etapa 2
│   ├── tratamento/<slug>.py     # Etapa 3
│   ├── perfil/<slug>.py         # Etapa E — perfila bruto + tidy + auxiliares
│   ├── analise/<slug>.py        # Etapa 4
│   └── publicacao/<slug>.py     # Etapa 5
├── dados/
│   ├── brutos/.gitkeep      # conteúdo NÃO versionado (regenerável pela Etapa 2)
│   └── processados/<slug>.csv
├── docs/                    # Etapas 6 e E — o que o GitHub Pages publica
│   ├── index.html           # sempre index.html (um painel por repositório)
│   ├── explorar.html        # Etapa E — perfil das tabelas + pauta analítica
│   ├── css/estilo.css
│   ├── js/app.js            # sempre app.js
│   ├── js/explorar.js       # Etapa E
│   └── dados/
│       ├── <slug>.json          # Etapa 5 — dados do painel
│       ├── perfil_<slug>.json   # Etapa E — GERADO pelo pipeline, sobrescrito
│       └── notas_<slug>.json    # Etapa E — ESCRITO À MÃO, nunca sobrescrito
└── prompts/                 # opcional: o prompt que originou esta fonte (registro histórico)
```

**Quatro regras que valem sempre:**
1. Dentro de `docs/`, o painel é sempre `index.html` + `js/app.js`. Nomes por fonte
   (`credito-modalidade.html`) só faziam sentido no repositório monolítico.
2. `dados/brutos/` nunca é versionado — o `.gitignore` já cobre. O `.gitkeep` existe só para
   a pasta sobreviver ao clone.
3. `src/config.py` registra **uma** fonte. O formato de dicionário por slug foi mantido para
   que o código das etapas não precisasse mudar.
4. `docs/dados/notas_<slug>.json` é o único arquivo dentro de `docs/dados/` que **não** é gerado
   por código. Nenhum script escreve nele — é onde a curadoria da Etapa E fica registrada.

## Estado atual: tudo publicado

Os três repositórios de fonte e o `controle-global` estão no GitHub, e os três painéis estão
no ar com a página de exploração (Etapa E):

| Fonte | Painel | Explorar |
|---|---|---|
| `fonte-meios-pagamento` | https://gfvdata-web.github.io/fonte-meios-pagamento/ | [explorar.html](https://gfvdata-web.github.io/fonte-meios-pagamento/explorar.html) |
| `fonte-arrecadacao-federal` | https://gfvdata-web.github.io/fonte-arrecadacao-federal/ | [explorar.html](https://gfvdata-web.github.io/fonte-arrecadacao-federal/explorar.html) |
| `fonte-credito-modalidade` | https://gfvdata-web.github.io/fonte-credito-modalidade/ | [explorar.html](https://gfvdata-web.github.io/fonte-credito-modalidade/explorar.html) |

Nenhum dos três tem atualização agendada ainda (roadmap, Etapa 7 no
[CONTEXTO.md](CONTEXTO.md)) — os dados são atualizados rodando o pipeline localmente.

## Passo a passo para publicar um repositório novo

Na raiz da pasta nova:

```bash
git init -b main
git add -A
git commit -m "Estrutura inicial do repositorio"
```

Depois, criar o repositório remoto e enviar (o `gh` cria e faz push de uma vez):

```bash
gh repo create gfvdata-web/<nome-do-repo> --public --source=. --push
```

Para um **repositório de fonte**, ligar o GitHub Pages na pasta `/docs`:

```bash
gh api -X POST repos/gfvdata-web/<nome-do-repo>/pages -f source[branch]=main -f source[path]=/docs
```

O painel fica em `https://gfvdata-web.github.io/<nome-do-repo>/` — que é exatamente a URL
escrita nos `README.md` e na navegação entre painéis. Se você mudar o nome de algum
repositório, esses links precisam ser atualizados junto.

`controle-global` **não** precisa de Pages: é documentação lida no GitHub mesmo.

## Ambiente Python por repositório

Cada repositório tem o seu venv (não compartilhe o venv antigo entre eles):

```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

`fonte-arrecadacao-federal` é o único que precisa de `openpyxl` — está no `requirements.txt`
dele.

## O que fazer com o repositório antigo

`DadosFinanceirosBancoCentral` continha as três fontes juntas. Todo o conteúdo dele foi
migrado para os repositórios de fonte, e ele **já está arquivado** no GitHub (só-leitura:
o histórico continua acessível e ninguém commita nele por engano). O site antigo
(https://gfvdata-web.github.io/DadosFinanceirosBancoCentral/) continua respondendo, congelado.

Pendência opcional: como o repositório arquivado não aceita commit, trocar o `docs/index.html`
antigo por uma página apontando para os três painéis novos exige desarquivar, commitar e
arquivar de novo. **Apagar** o repositório continua sendo irreversível — só se não quiser mais
o histórico anterior à divisão.

## Onde cada coisa passou a morar

Referência rápida de onde foi parar o conteúdo do repositório antigo:

| Antes (repositório antigo) | Agora |
|---|---|
| `CONTEXTO.md` (seções globais: etapas, contrato, convenções) | `controle-global/CONTEXTO.md` |
| `CONTEXTO.md` (seção 8: lista de fontes) | `controle-global/catalogo/fontes.md` |
| `catalogo/fontes.md` (dicionário, uma seção por fonte) | dividido em `catalogo/fonte.md` de cada repositório de fonte |
| `catalogo/fontes-candidatas.md` | `controle-global/catalogo/fontes-candidatas.md` |
| `CONTEXTO.md` (seção 12: padrão de prompt) | `controle-global/prompts/modelo-fonte-nova.md` |
| `prompts/fonte-06-*.md`, `prompts/fonte-08-*.md` | `prompts/` do repositório da respectiva fonte |
| `src/config.py` (3 fontes num dicionário) | dividido: um `src/config.py` por repositório, com uma fonte cada |
| `run_pipeline.py` (com registro `PIPELINES` e argumento de slug) | um `run_pipeline.py` por repositório, sem argumento de slug |
| `src/<etapa>/<slug>.py` | mesmo caminho, no repositório da fonte |
| `docs/index.html` | `fonte-meios-pagamento/docs/index.html` |
| `docs/arrecadacao-federal.html` | `fonte-arrecadacao-federal/docs/index.html` |
| `docs/credito-modalidade.html` | `fonte-credito-modalidade/docs/index.html` |
| `docs/js/app.js` | `fonte-meios-pagamento/docs/js/app.js` |
| `docs/js/arrecadacao-federal.js` | `fonte-arrecadacao-federal/docs/js/app.js` |
| `docs/js/credito-modalidade.js` | `fonte-credito-modalidade/docs/js/app.js` |
| `docs/css/estilo.css` | copiado nos três (duplicação deliberada — ver CONTEXTO.md, seção 3) |
| nav entre painéis (links relativos) | links absolutos para `gfvdata-web.github.io/fonte-*/` |

> Os arquivos em `prompts/` das fontes são **registro histórico**: eles descrevem o
> repositório monolítico (falam em `docs/credito-modalidade.html`, em adicionar entradas ao
> `PIPELINES`, etc.). Ficaram como estão de propósito — para fontes novas, use
> [`prompts/modelo-fonte-nova.md`](prompts/modelo-fonte-nova.md), que já reflete a
> arquitetura atual.

## Adicionando a próxima fonte

1. Escolher em [`catalogo/fontes-candidatas.md`](catalogo/fontes-candidatas.md).
2. Criar `fonte-<slug-com-hifen>/` com a anatomia acima, copiando de uma fonte parecida:
   API JSON → `fonte-meios-pagamento`; arquivo XLSX/CSV → `fonte-arrecadacao-federal`;
   muitas séries → `fonte-credito-modalidade`.
3. Preencher [`prompts/modelo-fonte-nova.md`](prompts/modelo-fonte-nova.md) e rodar a sessão
   dentro do repositório novo.
4. Ao terminar: registrar em `catalogo/fontes.md`, marcar ✅ em `fontes-candidatas.md`,
   adicionar a linha no `README.md` e o link na navegação dos painéis existentes — e seguir o
   checklist completo do [`CLAUDE.md`](CLAUDE.md) (inclui o mapa acima).
