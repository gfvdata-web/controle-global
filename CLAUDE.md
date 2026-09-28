# CLAUDE.md — controle-global

Repositório só de documentação (sem código): o índice do projeto de dados financeiros
públicos e dos demais sites da conta `gfvdata-web`. Visão completa em [CONTEXTO.md](CONTEXTO.md).

## Regra permanente: concordância entre todos os documentos

**Toda mudança neste repositório atualiza, no mesmo commit, todos os documentos que
referenciam o que mudou.** Nenhum documento pode ficar contando uma versão antiga do projeto.
Antes de commitar, percorrer o checklist abaixo e fazer um `grep` pelo nome/slug/URL alterado
em todos os `.md` para achar referências esquecidas.

Pedido do Guilherme: commit e push das mudanças de documentação podem ser feitos sem pedir
autorização de novo, desde que nada seja quebrado (links, âncoras, tabelas).

### Onde cada informação aparece

| Informação | Documentos que precisam concordar |
|---|---|
| Fonte financeira (nova, renomeada, removida) | `README.md` (tabela "Fontes implementadas"), `catalogo/fontes.md` (Resumo + seção), `catalogo/fontes-candidatas.md` (marcar ✅), `CONTEXTO.md` (árvore da seção 3 e tabela de dimensões/medidas da seção 6), `GUIA-REPOSITORIOS.md` (mapa e "Estado atual"), `prompts/modelo-pagina-exploracao.md` seção 8 (se fez Etapa E) |
| Projeto fora do escopo financeiro (Bolão F1, Chess Tracking, Simulador, …) | `README.md` ("Outros painéis do gfvdata-web"), `catalogo/fontes.md` ("Fora do escopo financeiro"), `GUIA-REPOSITORIOS.md` (mapa "Fora do escopo"), `CONTEXTO.md` (parágrafo após a árvore da seção 3) |
| Links de Google Forms (Bolão e "Outros forms") | `README.md`, `catalogo/fontes.md` |
| Etapas, contrato de dados, convenções | `CONTEXTO.md` (fonte da verdade), `GUIA-REPOSITORIOS.md` (anatomia), `prompts/modelo-fonte-nova.md`, `prompts/modelo-pagina-exploracao.md` |
| Arquivo novo/renomeado neste repositório | árvore da seção 3 do `CONTEXTO.md` e tabela "Documentação" do `README.md` |
| Item do roadmap concluído | `CONTEXTO.md` seção 9 (marcar `[x]`) e o que mais citar o item como pendente |

### Fora deste repositório, mas que costuma andar junto

O [`painel-status`](https://github.com/gfvdata-web/painel-status) (pasta irmã
`../painel-status/`) monitora os sites. Quando um site entra, sai ou muda de nome/URL:
- `painel-status/src/config.py` (lista `SITES`) e a seção "Sites monitorados" do
  `painel-status/README.md`;
- atalhos de forms sem site: constante `OUTROS_FORMS` em `painel-status/docs/js/app.js`.

Mudanças no `painel-status` são commitadas **no repositório dele**; aqui só se atualiza a
descrição.

### Não quebrar

- Links relativos entre documentos (`../README.md` a partir de `catalogo/` e `prompts/`).
- URLs publicadas seguem `https://gfvdata-web.github.io/<repo>/` — conferir com `curl` antes
  de escrever uma URL nova.
- Não afirmar estado ("no ar", "arquivado", "concluído") sem verificar (GitHub API / `curl`).
