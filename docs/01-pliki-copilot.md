# Pliki GitHub Copilot

Przegląd plików używanych przez GitHub Copilot – które są kluczowe dla efektywnej pracy, a które opcjonalne.

---

## Pliki wymagane / zalecane

### `.github/copilot-instructions.md`

**Najważniejszy plik.** Zawiera instrukcje na poziomie repozytorium – Copilot automatycznie dołącza je do każdego zapytania w chat i przy generowaniu kodu. Definiuje standardy projektu, styl kodu, ograniczenia i kontekst techniczny.

> Obsługiwane w: VS Code, IntelliJ/JetBrains (plugin ≥ 1.5), GitHub.com

**Przykładowy plik:** [`przyklady/.github/copilot-instructions.md`](../przyklady/.github/copilot-instructions.md)

```markdown
---
description: Instrukcje repozytoryjne dla GitHub Copilot (stack, standardy i bezpieczeństwo).
---

# Instrukcje Copilot

## Stack technologiczny
- TypeScript 5.x, Node.js 20, Express 4
- Testy: Vitest, Supertest

## Standardy kodu
- Używaj `async/await` zamiast `.then()`
- Eksportowane funkcje muszą mieć typowanie JSDoc lub TypeScript
- Nie używaj `any` bez komentarza z uzasadnieniem

## Konwencje nazewnictwa
- Pliki: `kebab-case.ts`
- Klasy: `PascalCase`
- Zmienne/funkcje: `camelCase`
```

---

## Pliki opcjonalne (warto mieć)

### `.github/instructions/*.instructions.md`

Instrukcje **path-specific** – stosowane tylko gdy Copilot pracuje na plikach pasujących do zdefiniowanej ścieżki. Pozwalają unikać przeładowania głównych instrukcji informacjami istotnymi tylko dla wybranych części projektu.

> Obsługiwane w: VS Code 1.99+, GitHub.com

**Przykładowy plik:** [`przyklady/.github/instructions/typescript.instructions.md`](../przyklady/.github/instructions/typescript.instructions.md)

```markdown
---
applyTo: "**/*.ts,**/*.tsx"
description: Zasady kodowania TypeScript dla plików TS/TSX.
---

# Instrukcje dla plików TypeScript

- Zawsze używaj ścisłego typowania – bez `any`
- Eksportowane funkcje muszą mieć typy parametrów i wartości zwracanej
- Używaj `interface` zamiast `type` dla obiektów danych
```

Plik `applyTo` określa glob pattern – instrukcje są dołączane tylko gdy edytowany plik pasuje do wzorca.

---

### `AGENTS.md` / `CLAUDE.md` / `GEMINI.md`

Instrukcje na poziomie projektu dla **agentów trzecich firm** (Claude, Gemini itp.) oraz Copilot coding agenta. Umieszczane w katalogu głównym repozytorium lub w podkatalogach – Copilot szuka ich hierarchicznie (instrukcje z podkatalogu mają wyższy priorytet w tym kontekście).

> Obsługiwane w: GitHub Copilot coding agent, Claude Code, Gemini CLI i inne

```markdown
# AGENTS.md

## Cel projektu
Aplikacja webowa do zarządzania zadaniami – TypeScript, Node.js, React.

## Zasady pracy agenta
- Nie modyfikuj plików produkcyjnych bez testów
- Uruchom `npm test` po każdej zmianie logiki
- Używaj Conventional Commits dla wiadomości commitów
- Nie commituj sekretów ani kluczy API
```

> `AGENTS.md` jest coraz powszechniej akceptowanym standardem między różnymi narzędziami AI. Copilot traktuje go podobnie do `.github/copilot-instructions.md`.

---

### `.github/prompts/*.prompt.md`

Szablony promptów wielokrotnego użytku. Dostępne w Copilot Chat przez menu `/` (VS Code) lub jako szybkie akcje. Każdy plik `.prompt.md` pojawia się jako oddzielna komenda.

> Obsługiwane w: VS Code 1.99+

**Przykładowe pliki:** [`przyklady/.github/prompts/`](../przyklady/.github/prompts/)

```
.github/prompts/
├── typescript.prompt.md      # Generowanie kodu TS zgodnie ze standardami
├── code-review.prompt.md     # Szablon przeglądu kodu
└── dokumentacja.prompt.md    # Generowanie dokumentacji JSDoc/OpenAPI
```

---

### `.github/mcp.json`

Konfiguracja serwerów MCP (Model Context Protocol) dla **Copilot coding agent** działającego w chmurze (GitHub Actions). Umożliwia agentowi dostęp do zewnętrznych narzędzi i API podczas wykonywania zadań.

> Obsługiwane w: GitHub Copilot coding agent (cloud), GitHub Enterprise

**Przykładowy plik:** [`przyklady/.github/mcp.json`](../przyklady/.github/mcp.json)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

### `.github/CODEOWNERS`

Definiuje właścicieli plików i katalogów w repozytorium. GitHub automatycznie dodaje właścicieli jako recenzentów w Pull Requestach dotyczących ich plików. Copilot respektuje ten plik przy przypisywaniu review.

> Obsługiwane w: GitHub (wszystkie plany)

**Przykładowa zawartość:**

```
# Właściciele całego repozytorium
*  @org/backend-team

# Pliki konfiguracyjne – wymaga zatwierdzenia DevOps
.github/  @org/devops

# Kod frontendowy
src/frontend/  @org/frontend-team
```

---

### `.github/workflows/copilot-setup-steps.yml`

Konfiguruje środowisko dla **Copilot coding agenta** działającego w chmurze. Kroki w tym pliku są wykonywane przed każdym zadaniem agenta – instalują zależności, konfigurują narzędzia i przygotowują środowisko. Zadanie musi mieć nazwę dokładnie `copilot-setup-steps`.

> Obsługiwane w: GitHub Copilot coding agent (cloud), GitHub Enterprise

**Przykładowy plik:** [`przyklady/.github/workflows/copilot-setup-steps.yml`](../przyklady/.github/workflows/copilot-setup-steps.yml)

```yaml
jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
```

---

### `.github/agents/*.agent.md`

Definicje **custom agentów** – wyspecjalizowanych agentów z własną tożsamością, zestawem narzędzi i instrukcjami. Każdy plik `.agent.md` tworzy osobnego agenta dostępnego z listy rozwijane w Copilot Chat i GitHub.com.

> Obsługiwane w: VS Code, JetBrains (preview), GitHub.com, Copilot CLI

**Przykładowy plik:** [`przyklady/.github/agents/test-specialist.agent.md`](../przyklady/.github/agents/test-specialist.agent.md)

```markdown
---
name: test-specialist
description: Specjalista od testów – analizuje pokrycie kodu i pisze testy jednostkowe oraz integracyjne
tools: ["read", "edit", "search", "shell"]
model: gpt-5.5
target: github-copilot
---

Jesteś specjalistą od testowania kodu. Twoje zadania:

- Analizuj istniejące testy i identyfikuj luki w pokryciu
- Pisz testy jednostkowe (Vitest) i integracyjne (Supertest)
- Nie modyfikuj kodu produkcyjnego bez wyraźnego polecenia
- Każdy test musi zawierać opis (`describe`/`it`) i asercje
```

**Pola YAML frontmatter:**

| Pole | Opis |
|---|---|
| `name` | Wyświetlana nazwa agenta |
| `description` | Krótki opis – widoczny w menu wyboru agenta (wymagany) |
| `tools` | Lista narzędzi (np. `read`, `edit`, `shell`, `search`); jeśli pominięty – dostęp do wszystkich |
| `mcp-servers` | Konfiguracja serwerów MCP dostępnych tylko dla tego agenta |
| `model` | Model AI (np. `claude-sonnet-4-5`); obsługiwane w VS Code i JetBrains |
| `target` | Ogranicz do środowiska: `vscode` lub `github-copilot` |

---

### `.github/skills/<nazwa>/SKILL.md`

**Agent skills** – foldery z instrukcjami, skryptami i zasobami które Copilot ładuje automatycznie gdy zadanie jest powiązane z daną umiejętnością. Pozwalają nauczyć Copilota powtarzalnych, wieloetapowych procesów.

> Obsługiwane w: GitHub Copilot cloud agent, Copilot CLI, VS Code (agent mode)

**Przykładowy plik:** [`przyklady/.github/skills/github-actions-debugging/SKILL.md`](../przyklady/.github/skills/github-actions-debugging/SKILL.md)

```markdown
---
name: github-actions-debugging
description: Instrukcja debugowania nieudanych workflow GitHub Actions. Używaj gdy prosisz o naprawienie błędów CI.
allowed-tools:
  - shell
---

Aby debugować nieudane workflow GitHub Actions:

1. Użyj narzędzia `list_workflow_runs` aby sprawdzić ostatnie uruchomienia
2. Użyj `summarize_job_log_failures` aby uzyskać podsumowanie błędów
3. Jeśli potrzebujesz więcej szczegółów – użyj `get_job_logs`
4. Spróbuj odtworzyć błąd lokalnie przed wprowadzeniem poprawek
5. Upewnij się że testy przechodzą przed commitem
```

**Struktura skill:**

```
.github/skills/
└── github-actions-debugging/
    ├── SKILL.md          ← wymagany plik główny
    └── debug-ci.sh       ← opcjonalny skrypt (może być wywołany przez skill)
```

Skill można też zainstalować poleceniem `gh skill install OWNER/REPO`.

---

### Subagenty (runtime)

**Subagenty** to osobne instancje agenta tworzone automatycznie przez głównego agenta Copilot w celu wykonania wyizolowanego podzadania (np. przeszukanie kodu, uruchomienie testów) i zwrócenia wyników. Nie są konfigurowane jako pliki – to mechanizm runtime.

> Obsługiwane w: GitHub Copilot cloud agent, VS Code (agent mode)

**Kiedy Copilot tworzy subagenta:**
- Gdy zadanie wymaga równoległej pracy w odizolowanym kontekście
- Przy złożonych analizach kodu (np. przeszukiwanie wielu plików jednocześnie)
- Gdy główny agent deleguje wyspecjalizowane zadanie

Subagenty kończące pracę wyzwalają hook `subagentStop` – zob. sekcja [`.github/hooks/*.json`](#githubhooksjson) powyżej.

---

### `.github/hooks/*.json`

**Lifecycle hooks** – skrypty wykonywane automatycznie w określonych punktach cyklu życia agenta. Pozwalają na zatwierdzanie lub blokowanie wywołań narzędzi, logowanie, skanowanie sekretów i inne zadania automatyczne.

> Obsługiwane w: GitHub Copilot cloud agent, Copilot CLI

**Przykładowy plik:** [`przyklady/.github/hooks/project-hooks.json`](../przyklady/.github/hooks/project-hooks.json)

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": "./scripts/check-secrets.sh",
        "timeoutSec": 15
      }
    ],
    "sessionStart": [
      {
        "type": "command",
        "bash": "echo \"Session started: $(date)\" >> logs/agent.log",
        "timeoutSec": 5
      }
    ]
  }
}
```

**Dostępne typy hooków:**

| Hook | Kiedy wywołany |
|---|---|
| `sessionStart` | Na początku sesji agenta |
| `sessionEnd` | Na końcu sesji agenta |
| `userPromptSubmitted` | Gdy użytkownik wysyła prompt |
| `preToolUse` | Przed każdym wywołaniem narzędzia (może **zatwierdzić lub zablokować**) |
| `postToolUse` | Po wykonaniu narzędzia |
| `agentStop` | Gdy główny agent kończy odpowiedź |
| `subagentStop` | Gdy subagent kończy pracę |
| `errorOccurred` | Gdy wystąpi błąd |

---

### `.vscode/settings.json`

Ustawienia VS Code specyficzne dla workspace. Pozwala skonfigurować zachowanie Copilot dla wszystkich developerów w projekcie (np. włączyć/wyłączyć dla danych języków, włączyć tryb agenta).

> Obsługiwane w: VS Code

**Przykładowy plik:** [`przyklady/.vscode/settings.json`](../przyklady/.vscode/settings.json)

---

### `.vscode/mcp.json`

Konfiguracja serwerów MCP dla **VS Code** (lokalnie, na poziomie workspace). Serwery MCP rozszerzają możliwości agenta Copilot o zewnętrzne narzędzia (np. dostęp do GitHub API, baz danych, narzędzi wyszukiwania).

> Obsługiwane w: VS Code 1.99+ (agent mode)

**Przykładowy plik:** [`przyklady/.vscode/mcp.json`](../przyklady/.vscode/mcp.json)

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${input:github_mcp_pat}"
      }
    }
  },
  "inputs": [
    {
      "type": "promptString",
      "id": "github_mcp_pat",
      "description": "GitHub Personal Access Token",
      "password": true
    }
  ]
}
```

---

### `.vscode/extensions.json`

Lista zalecanych rozszerzeń VS Code dla projektu. Gdy deweloper otworzy repozytorium, VS Code zaproponuje instalację tych rozszerzeń.

> Obsługiwane w: VS Code

**Przykładowy plik:** [`przyklady/.vscode/extensions.json`](../przyklady/.vscode/extensions.json)

```json
{
  "recommendations": [
    "GitHub.copilot",
    "GitHub.copilot-chat"
  ]
}
```

---

## Pliki globalne (poza repozytorium)

Poniższe pliki konfiguruje się lokalnie na maszynie dewelopera – nie są wersjonowane w repozytorium.

| Plik | System | Opis |
|---|---|---|
| `~/.config/github-copilot/intellij/global-copilot-instructions.md` | macOS/Linux | Globalne instrukcje dla IntelliJ – stosowane we wszystkich projektach |
| `%LOCALAPPDATA%\github-copilot\intellij\global-copilot-instructions.md` | Windows | Jak wyżej, dla Windows |
| `~/.config/github-copilot/vscode/global-copilot-instructions.md` | macOS/Linux | Globalne instrukcje dla VS Code |
| `%APPDATA%\Code\User\globalStorage\github.copilot-chat\global-copilot-instructions.md` | Windows | Globalne instrukcje dla VS Code (Windows) |

---

## Podsumowanie

| Plik | Priorytet | IDE / Środowisko |
|---|---|---|
| `.github/copilot-instructions.md` | ⭐ Kluczowy | VS Code, IntelliJ, GitHub.com |
| `.github/instructions/*.instructions.md` | ✅ Zalecany | VS Code, GitHub.com |
| `AGENTS.md` / `CLAUDE.md` | ✅ Zalecany | Copilot agent, Claude, Gemini |
| `.github/prompts/*.prompt.md` | ✅ Zalecany | VS Code |
| `.github/agents/*.agent.md` | ✅ Zalecany | VS Code, GitHub.com, Copilot CLI |
| `.github/skills/<name>/SKILL.md` | ✅ Zalecany | Copilot cloud agent, CLI, VS Code |
| `.github/hooks/*.json` | ✅ Zalecany | Copilot cloud agent, Copilot CLI |
| `.vscode/mcp.json` | ✅ Zalecany | VS Code |
| `.github/mcp.json` | ✅ Zalecany | GitHub cloud agent |
| `.github/workflows/copilot-setup-steps.yml` | ✅ Zalecany | GitHub cloud agent |
| `.github/CODEOWNERS` | ℹ️ Opcjonalny | GitHub (wszystkie plany) |
| `.vscode/settings.json` | ℹ️ Opcjonalny | VS Code |
| `.vscode/extensions.json` | ℹ️ Opcjonalny | VS Code |
