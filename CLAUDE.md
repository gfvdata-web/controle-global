# CLAUDE.md — controle-global

Repositório só de documentação (sem código) do projeto de **dados financeiros públicos
brasileiros**. Visão completa em [CONTEXTO.md](CONTEXTO.md).

## Escopo: só as fontes financeiras

Este repositório documenta **apenas** as fontes financeiras e o que vale para elas. Hoje são
três, cada uma com seu repositório e seu painel:

- `fonte-meios-pagamento` — BCB, meios de pagamento
- `fonte-arrecadacao-federal` — RFB, arrecadação federal
- `fonte-credito-modalidade` — BCB/SGS, crédito por modalidade

Qualquer outro site ou projeto da conta `gfvdata-web` **não faz parte deste projeto e não entra
em nenhum documento daqui** — nem como link ou índice. O que for sobre eles é documentado no
repositório de cada um.

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
| Etapas, contrato de dados, convenções | `CONTEXTO.md` (fonte da verdade), `GUIA-REPOSITORIOS.md` (anatomia), `prompts/modelo-fonte-nova.md`, `prompts/modelo-pagina-exploracao.md` |
| Arquivo novo/renomeado neste repositório | árvore da seção 3 do `CONTEXTO.md` e tabela "Documentação" do `README.md` |
| Item do roadmap concluído | `CONTEXTO.md` seção 9 (marcar `[x]`) e o que mais citar o item como pendente |

### Não quebrar

- Links relativos entre documentos (`../README.md` a partir de `catalogo/` e `prompts/`).
- URLs publicadas seguem `https://gfvdata-web.github.io/<repo>/` — conferir com `curl` antes
  de escrever uma URL nova.
- Não afirmar estado ("no ar", "arquivado", "concluído") sem verificar (GitHub API / `curl`).
