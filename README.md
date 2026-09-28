# Dados Financeiros Abertos — Brasil · Controle Global

Repositório-índice do projeto de exploração de **dados financeiros públicos brasileiros**
(Banco Central, Receita Federal e demais fontes oficiais abertas).

**Este repositório não tem código nem pipeline.** É só documentação: a visão consolidada de
todas as fontes, as convenções que valem para todas elas, e o radar de fontes ainda não
implementadas. Cada fonte de dados vive em **seu próprio repositório**, autocontido, com o
seu pipeline e o seu dashboard.

## Fontes implementadas

| Fonte | Repositório | Painel | Explorar | Órgão | Período | Geo |
|-------|-------------|--------|----------|-------|---------|-----|
| Meios de pagamento (mensal) | `fonte-meios-pagamento` | [painel](https://gfvdata-web.github.io/fonte-meios-pagamento/) | [dados](https://gfvdata-web.github.io/fonte-meios-pagamento/explorar.html) | BCB | abr/2002 → | ⚪ nacional |
| Arrecadação federal | `fonte-arrecadacao-federal` | [painel](https://gfvdata-web.github.io/fonte-arrecadacao-federal/) | [dados](https://gfvdata-web.github.io/fonte-arrecadacao-federal/explorar.html) | RFB | jan/1994 → | ⚪ nacional |
| Crédito por modalidade | `fonte-credito-modalidade` | [painel](https://gfvdata-web.github.io/fonte-credito-modalidade/) | [dados](https://gfvdata-web.github.io/fonte-credito-modalidade/explorar.html) | BCB/SGS | mar/2011 → | ⚪ nacional |

## Documentação

| Arquivo | O que é |
|---------|---------|
| [CONTEXTO.md](CONTEXTO.md) | Visão global: arquitetura de repositórios, etapas, contrato de dados e convenções que toda fonte segue |
| [catalogo/fontes.md](catalogo/fontes.md) | Índice das fontes implementadas — o que cada uma cobre e onde mora |
| [catalogo/fontes-candidatas.md](catalogo/fontes-candidatas.md) | Radar de 19 fontes oficiais ainda não implementadas, com priorização |
| [prompts/modelo-fonte-nova.md](prompts/modelo-fonte-nova.md) | Modelo de prompt para abrir a sessão de uma fonte nova |
| [prompts/modelo-pagina-exploracao.md](prompts/modelo-pagina-exploracao.md) | **Etapa E** — como construir, em qualquer fonte, a página `explorar.html`: perfil das tabelas + pauta analítica discutível |
| [GUIA-REPOSITORIOS.md](GUIA-REPOSITORIOS.md) | Como criar/organizar os repositórios e o que vai em cada pasta |
| [CLAUDE.md](CLAUDE.md) | Escopo deste repositório e regra de manutenção: toda mudança atualiza **todos** os documentos que a referenciam (checklist) |

## Como adicionar uma fonte nova

1. Escolher a fonte em [`catalogo/fontes-candidatas.md`](catalogo/fontes-candidatas.md).
2. Preencher o modelo em [`prompts/modelo-fonte-nova.md`](prompts/modelo-fonte-nova.md) com
   os dados dela.
3. Criar o repositório `fonte-<slug-com-hifen>` seguindo a estrutura de
   [`GUIA-REPOSITORIOS.md`](GUIA-REPOSITORIOS.md).
4. Abrir uma conversa nova, colar o prompt preenchido e construir as Etapas 1→3, **E**, 4→7 lá.
   A [Etapa E](prompts/modelo-pagina-exploracao.md) é ponto de parada: a pauta aprovada nela
   é que define os gráficos das Etapas 4 e 6.
5. Voltar aqui e registrar a fonte na tabela acima e em
   [`catalogo/fontes.md`](catalogo/fontes.md); marcar a candidata como implementada.

Cada fonte é independente: pode ser construída em paralelo, em sessões diferentes, sem uma
interferir na outra.
