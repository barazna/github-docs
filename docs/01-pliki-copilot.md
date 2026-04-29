# Pliki GitHub Copilot

Przegląd plików używanych przez GitHub Copilot – które są kluczowe dla efektywnej pracy, a które opcjonalne.

---

## Pliki wymagane / zalecane

### `.github/copilot-instructions.md`

**Najważniejszy plik.** Zawiera instrukcje na poziomie repozytorium – Copilot automatycznie dołącza je do każdego zapytania w chat i przy generowaniu kodu. Definiuje standardy projektu, styl kodu, ograniczenia i kontekst techniczny.

> Obsługiwane w: VS Code, IntelliJ/JetBrains (plugin ≥ 1.5), GitHub.com

**Przykładowy plik:** [`przyklady/.github/copilot-instructions.md`](../przyklady/.github/copilot-instructions.md)

```markdown
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

| Plik | Priorytet | IDE |
|---|---|---|
| `.github/copilot-instructions.md` | ⭐ Kluczowy | VS Code, IntelliJ, GitHub.com |
| `.github/prompts/*.prompt.md` | ✅ Zalecany | VS Code |
| `.vscode/mcp.json` | ✅ Zalecany | VS Code |
| `.github/mcp.json` | ✅ Zalecany | GitHub cloud agent |
| `.github/workflows/copilot-setup-steps.yml` | ✅ Zalecany | GitHub cloud agent |
| `.github/CODEOWNERS` | ℹ️ Opcjonalny | GitHub (wszystkie plany) |
| `.vscode/settings.json` | ℹ️ Opcjonalny | VS Code |
| `.vscode/extensions.json` | ℹ️ Opcjonalny | VS Code |
