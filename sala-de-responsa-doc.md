# Sala de Responsa — Documentação Completa

## Visão geral
Funcionalidade embutida no **Diário de Classe** que permite ao professor registrar **Responsas** (atos positivos) e **Vacilos** (atos negativos) por aluno, por disciplina, por data e horário de aula. Tudo auto-contido em um único HTML (`diario-de-classe-home.html`) — sem dependência externa.

---

## 1. Acesso à funcionalidade

### Card inicial (Home)
- Card **Sala de Responsa** na grade de cards da tela inicial, com ícone de pessoas em grupo, cor roxa (`#8E5CF7`)
- Ao clicar no card → `onclick="openSalaTab()"` → alterna para a aba da Sala de Responsa

### Menu lateral interno (inner-nav)
- Item **Sala de Responsa** na navegação lateral (inner-nav), abaixo de "Registro de Aulas", acima de "Registro de Projetos"
- Submenu expansível com duas opções (seta `▼` para expandir/recolher):
  - **Registrar** — ativo por padrão, mostra filtros + cards de turma
  - **Resumos** — placeholder "Em breve"

### Atalho de teclado
- **ESC** → volta um nível na hierarquia:
  1. Fecha modal de Marcar Todos
  2. Fecha modal de Visualizar por Data
  3. Fecha modal de Visualizar Registros do Aluno
  4. Fecha modal de Registro Individual
  5. Volta da tela de alunos para os cards com filtros

---

## 2. Tela principal — Filtros e cards

### Identificação da escola
- Ao lado de "Filtros": badges roxos com **URE: Leste 5** e **Escola: E. M. Professor João da Silva**
- Campos informativos (não editáveis)

### Filtros (painel branco com borda):
| Filtro | Tipo | Opções |
|--------|------|--------|
| Ano Letivo | Select | 2026, 2025, 2024 |
| Tipo de Ensino | Select | Ensino Fundamental, Ensino Médio, EJA |
| Turma | Select | 1º Ano A, 1º Ano B, 2º Ano A, 2º Ano B, 3º Ano A, 3º Ano B |
| Disciplina | Select | Matemática, Língua Portuguesa, Ciências, História, Geografia, Inglês, Educação Física, Arte, Todas |
| Pesquisar | Botão roxo | Aciona `filtrarTurmas()` |

### Comportamento dos filtros:
- Sem filtro de turma: exibe **cards de turma** agrupando disciplinas
- Com filtro de turma: exibe **cards de disciplina** daquela turma
- Resultados mostram contador no topo ("X turma(s) encontrada(s)" ou "X disciplina(s) encontrada(s)")
- Ao abrir a aba pela primeira vez, carrega automaticamente as 2 primeiras turmas com todas as disciplinas

### Cards de turma/disciplina:
- **Estrutura HTML** (classes exatas):
  ```html
  <div class="conteudoItens-azul item">
    <div class="card-turma" role="button" tabindex="0">
      <div class="card-turma-nome">4° Ano A – Matemática</div>
      <div class="card-turma-tipo-ensino">
        <div class="icon-info">i</div>
        <span>Ensino Fundamental de 9 Anos</span>
      </div>
      <div class="card-turma-extra">
        <i class="fa-solid fa-chalkboard-user"></i> Prof. Nome · <i class="fa-regular fa-clock"></i> Manhã
      </div>
    </div>
  </div>
  ```
- Estilo: card branco com `border: 2px solid #e5e5e5`, `border-radius: 8px`, `padding: 12px 15px`
- Disciplina aparece em roxo (`var(--purple)`) no card
- Ao clicar no card → abre tela de alunos (`openStudentView`)

### Dados cadastrados:
- **8 turmas** (1ºA–3ºB) distribuídas entre Fundamental, Médio, manhã, tarde e noite
- Cada turma com **4 disciplinas** (total de 32 combinações)
- **~90 alunos** distribuídos entre 6 turmas (15 alunos por turma)

---

## 3. Tela de alunos (student view)

### Layout em duas colunas flex:
```
┌──────────────────────────┬────────────┐
│  Lista de Alunos (Lado   │ Calendário │
│  principal, flex:1)      │ + Horário  │
│                          │ (260px)    │
│  ┌─ Busca ─────────────┐ │            │
│  │ 🔍 Buscar aluno...  │ │            │
│  └───────────────────────┘ │            │
│  ┌─ Tabela ──────────────┐│            │
│  │ Aluno │ 👁️ │Resp.│Vac.││            │
│  │ Nome  │ 👁️ │ ➕ │ ➖  ││            │
│  │ Nome  │ 👁️ │ ➕ │ ➖  ││            │
│  └───────────────────────┘│            │
│  └── Rodapé (totais) ────┘│            │
└──────────────────────────┴────────────┘
```

- **Lado esquerdo** (`order: 2` na flex): tabela de alunos com scroll (`max-height: calc(100vh - 160px)`, `scrollbar-width: none`)
- **Lado direito** (`order: 1`): calendário + horário, sticky (`position: sticky; top: 14px`)

### Cabeçalho:
- Botão **Voltar** roxo no topo → `backToCards()`
- Nome da turma + disciplina, professor, badge de turno (Manhã = verde, Tarde = laranja, Noite = azul)

### Coluna esquerda — Lista de alunos:
- **Busca**: campo de texto com filtragem em tempo real (`oninput="renderStudentView()"`)
- **Cabeçalho da tabela** (`.sv-th`):
  - "Aluno", ícone 👁️ (coluna "Ver"), botões **Resp.** (verde) e **Vac.** (laranja) para "Marcar Todos"
- **Linhas de aluno** (`.sv-row`):
  - Nome do aluno
  - Ícone 👁️ → `openViewModal(aluno)` — abre modal de consulta
  - Botão ➕ verde → `handleThumbClick(..., 'responsa')` — abre modal de registro individual
  - Botão ➖ laranja → `handleThumbClick(..., 'vacilo')` — abre modal de registro individual
  - Badge com contagem quando > 0 registros
- **Rodapé** (`.sv-footer`):
  - Totais: bolinha verde + número de Responsas, bolinha laranja + número de Vacilos, total de alunos

---

## 4. Calendário e Horário (painel direito)

### Calendário customizado:
- Navegação entre meses (botões `< >`)
- Dias clicáveis → seleciona a data
- Dia selecionado: fundo roxo (`var(--purple)`)
- Dias com registro: texto azul (`#1565D8`) com bolinha azul abaixo
- Ao clicar em dia com registro → seleciona a data e filtra

### Funcionalidades do calendário:
- Ao **mudar a data**:
  1. Horário é resetado para vazio
  2. Opções de horário com registro ficam em azul
  3. Abaixo aparece "Com registro: 1ª Aula, 3ª Aula" com ✅
  4. Lista de alunos é refiltrada
  5. "Dias com lançamento" é atualizado

### Seletor de horário:
- Opções: 1ª a 6ª Aula
- Horários com registro: texto azul + negrito no dropdown
- Abaixo do select: info "Com registro: 1ª Aula, 3ª Aula" (em azul)

### Informação da aula selecionada:
- "Aula selecionada: 12/06/2026 — 2ª Aula"
- Se data mas sem horário: "12/06/2026 (horário não selecionado)"
- Se nada: "Nenhuma aula selecionada"

### Dias com lançamento:
- Badges azuis clicáveis com as datas formatadas (pt-BR)
- Ao clicar em uma badge → abre modal de consulta por data (`openDateViewModal`)

### Bloqueio de registro:
- ➕ e ➖ só funcionam com **data E horário** selecionados
- Ao clicar sem seleção: toast "Selecione a data/horário da aula antes de registrar."
- Data e horário selecionados são **preenchidos automaticamente** no modal de registro (readonly)
- O usuário não pode alterar a data/horário no modal — só voltando à tela de alunos

---

## 5. Modal de registro individual (➕ ou ➖)

### Campos do modal:
| Campo | Tipo | Fonte |
|-------|------|-------|
| Aluno(a) | Texto (readonly) | Nome do aluno |
| Tipo | Badge verde/laranja | Responsa (➕) ou Vacilo (➖) |
| Data | Input date (readonly, desabilitado) | Data selecionada no calendário |
| Horário | Texto (readonly) | Horário selecionado |
| Comportamento observado | Select + campo "Outros" | Opções pré-definidas |
| O que você fez em relação a isso? | Textarea | Observação livre |

### Comportamentos observados:
**Responsa:**
1. Chega com material e preparado
2. Mantém pontualidade
3. Colabora em grupos
4. Entrega atividades e projetos
5. Outros (abre campo de texto livre)

**Vacilo:**
1. Não levou material
2. Conversa paralela
3. Não fez a tarefa
4. Atraso sem justificativa
5. Desrespeito ao professor
6. Outros (abre campo de texto livre)

### Validação:
- Se "Outros" selecionado sem preenchimento: "Descreva o comportamento observado."
- Se comportamento não selecionado: "Selecione um comportamento observado."
- Se data vazia: "Selecione uma data."

### Ao confirmar:
1. Cria registro com `id` único incremental
2. Salva no array `registros[]`
3. Fecha modal
4. Atualiza lista de alunos (contadores)
5. Atualiza calendário e horários
6. Mostra toast "Responsa registrada para Nome!" ou "Vacilo registrado para Nome!"

---

## 6. Modal de Marcar Todos (Resp./Vac. no cabeçalho)

### Abertura:
- Botão **Resp.** (verde) → `marcarTodos('responsa')`
- Botão **Vac.** (laranja) → `marcarTodos('vacilo')`

### Validações:
- Data e horário obrigatórios (mesmo bloqueio dos botões individuais)

### Lista de alunos:
- Cada aluno listado no mesmo formato do modal de consulta:
  ```
  ➕ Responsa
  Nome do Aluno
  ```
  Com badge circular (verde para Responsa, laranja para Vacilo) e nome em negrito

### Comportamento:
- Select de comportamento (com opção "Outros" + campo de texto)
- Mesmas opções do modal individual

### Ao confirmar (`confirmMarcarTodos`):
1. Pega todos os alunos filtrados (considerando busca)
2. Registra cada um com o comportamento selecionado
3. Obs padrão: "Marcado em lote"
4. Fecha modal
5. Atualiza tela
6. Toast: "Responsa registrada para X aluno(s)!"

---

## 7. Modal de Visualizar Registros do Aluno (👁️)

### Abertura:
- Clique no ícone 👁️ ao lado do nome do aluno (tanto na tela de alunos quanto no modal de lista)

### Conteúdo:
- Cabeçalho: nome do aluno, turma + disciplina
- Lista de registros ordenada por data decrescente
- Cada registro mostra:
  - Badge circular: ➕ verde (Responsa) ou ➖ laranja (Vacilo)
  - Tipo em negrito colorido
  - Data + horário
  - Comportamento (badge arredondado)
  - Observação (se houver, em fundo cinza)
  - Botão 🗑️ (lixeira, visível sempre, `opacity: .65`) → exclui o registro
- Rodapé: total de Responsas, Vacilos e registros

### Exclusão de registro:
- Botão de lixeira visível permanentemente (sem hover)
- Ao clicar: confirma e remove do array, atualiza tela
- Toast: "Registro excluído."

---

## 8. Modal de Visualizar por Data (clicar em data nos "Dias com lançamento")

### Abertura:
- Clique em qualquer badge de data na seção "Dias com lançamento"

### Conteúdo:
- Cabeçalho: turma + disciplina, data formatada
- Filtro de aluno (dropdown) — "Todos os alunos" ou nome específico
- Lista de registros daquela data, ordenados por nome do aluno
- Cada registro: badge, tipo, **nome do aluno**, horário, comportamento, observação, botão 🗑️
- Rodapé: totais

---

## 9. Dados armazenados

### Estrutura do registro:
```js
{
  id: Number,                  // único, incremental
  turma: String,               // ex: "1º Ano A"
  disciplinaId: String,        // ex: "matematica"
  aluno: String,               // nome completo
  tipo: String,                // "responsa" | "vacilo"
  data: String,                // "YYYY-MM-DD"
  horario: String,             // ex: "2ª Aula"
  comportamento: String,       // ex: "Colabora em grupos"
  obs: String                  // ex: "Marcado em lote" ou observação livre
}
```

### Persistência:
- Todos os registros ficam no array `registros[]` em memória durante a sessão
- Ao recarregar a página, os dados são perdidos (não há backend/salvamento ainda)

### Funções de consulta:
- `getRegistros(turma, discId, aluno, tipo)` — todos os registros
- `getRegistrosForDate(turma, discId, aluno, tipo, data)` — por data
- `getRegistrosForDateTime(turma, discId, aluno, tipo, data, horarioLabel)` — por data + horário
- `getDiasComRegistro(turma, discId)` — datas únicas com registros

---

## 10. Estrutura dos dados de turma

```js
const turmasData = [
  {
    nome: "1º Ano A",        // nome da turma
    ensino: "fundamental",   // "fundamental" | "medio"
    turno: "manha",          // "manha" | "tarde" | "noite"
    disciplinas: [
      { nome: "Matemática", id: "matematica", professor: "Prof. Carlos", alunos: 32 },
      ...
    ]
  },
  ...
];
```

### Alunos por turma:
```js
const alunosPorTurma = {
  "1º Ano A": ["Ana Beatriz Silva", "Bruno Oliveira", ...],  // 15 alunos
  // 6 turmas com 15 alunos cada = ~90 alunos
};
```

---

## 11. Regras de navegação e UX

### Hierarquia de telas:
1. **Home** (cards) → clica card Sala de Responsa
2. **Filtros + cards de turma** → clica em um card
3. **Tela de alunos** (com calendário) → clica ➕/➖ ou 👁️
4. **Modal de registro / visualização** → confirma ou fecha

### Navegação por ESC:
- Modal de Marcar Todos → ESC fecha modal
- Modal de Visualizar por Data → ESC fecha modal
- Modal de Visualizar Registros → ESC fecha modal
- Modal de Registro Individual → ESC fecha modal
- Tela de alunos → ESC volta para cards

### Navegação por clique fora:
- Todos os modais fecham ao clicar no fundo escuro (`onclick="if(event.target===this)close..."`)

### Abas (sub-sidebar):
- **Registrar** — mostra filtros + cards + tela de alunos
- **Resumos** — placeholder "Em breve" (ícone de gráfico)
- Ao clicar em "Registrar" pela primeira vez: carrega 2 turmas iniciais
- Ao clicar em "Início" na inner-nav: volta para Home (cards)

---

## 12. Cores do sistema

| Token | Cor | Uso |
|-------|-----|-----|
| `--purple` | `#8E5CF7` | Principal da Sala de Responsa, ícones, botão Pesquisar |
| `--pink` | `#E36496` | Card Frequência |
| `--yellow` | `#F7B100` | Card Registro de Aulas |
| `--navy` | `#183B56` | Card Avaliação |
| `--red` | `#EB5757` | Card Fechamento |
| `--cyan` | `#00C5D4` | Card Registro de Projetos |
| — | `#1565D8` | Dias com registro, links, nav-active |
| — | `#2e7d32` | Verde Responsa (badges, botões ➕) |
| — | `#e65100` | Laranja Vacilo (badges, botões ➖) |
| — | `#e8f5e9` | Fundo claro verde (Responsa) |
| — | `#fff8e1` | Fundo claro laranja (Vacilo) |
| — | `#e3f2fd` | Fundo azul claro (dias com registro) |
| — | `#17a34a` | Verde "Dias com lançamento" ✅ |

---

## 13. Tecnologia

- **HTML**: auto-contido, CSS + JS inline
- **CSS**: Custom properties, flexbox, grid, media queries (`@media(max-width:820px)`)
- **JS**: Vanilla JavaScript, DOM manipulation, sem dependências
- **Ícones**: Font Awesome 6 (`fa-solid`, `fa-regular`) + Bootstrap Icons (`bi`)
- **Fonte**: Open Sans (Google Fonts)
- **Arquivo único**: `diario-de-classe-home.html` (~1558 linhas)
- **Funciona**: 100% offline, abre direto no navegador, sem servidor

---

## 14. Regras de design

- **Cards de turma**: branco, `border: 2px solid #e5e5e5`, `border-radius: 8px`, sem sombra, padding de 12px 15px
- **Modais**: `border-radius: 16px`, animação slide-up (`modalIn`), overlay escuro 40%
- **Botões ➕/➖**: 34×34px, circulares, com contagem em badge flutuante quando > 0
- **Tabela de alunos**: linhas com hover sutil (`#fafbff`), sem scrollbar visível
- **Responsivo**: abaixo de 820px as duas colunas empilham (vertical)

---

## 15. Regras de comportamento

### Regras de negócio:
1. **Data e horário são obrigatórios** para qualquer registro — não pode registrar sem
2. **Data/horário bloqueados no modal** — não se pode alterar, só voltando
3. **Um registro por aluno/data/horário e comportamento** — mesmo aluno pode ter múltiplos registros no mesmo dia/horário (com comportamentos diferentes)
4. **Registro em lote**: usa comportamento padrão selecionado + obs "Marcado em lote"
5. **Exclusão**: remove do array e atualiza toda a UI (lista, calendário, horários)
6. **Contadores**: mostram apenas registros do filtro atual (data + horário selecionados)

### Regras de UI:
1. **Um único modal aberto por vez** — ESC fecha na ordem correta
2. **Calendário e horário sincronizados** — mudar data limpa horário
3. **Horários com registro marcados** — visual no dropdown + info abaixo
4. **Dias com lançamento** — clicáveis, abrem consulta por data com filtro de aluno
5. **Busca de aluno** — filtra em tempo real, afeta também "Marcar Todos"
6. **Toast** — mensagens de feedback no centro inferior, 2.5s de duração

---

## 16. Funcionalidades planejadas / não implementadas

- **Resumos**: aba placeholder "Em breve" (ícone de gráfico) — destino para dashboards e relatórios
- **Persistência**: registros estão apenas em memória (array `registros[]`) — sem localStorage ou backend
- **Exportação**: sem botão de exportar relatório
- **Edição de registro**: não implementada — apenas exclusão
