# Konfiguracja VS Code dla GitHub Copilot

Jak skonfigurować Visual Studio Code, aby Copilot używał instrukcji, promptów i serwerów MCP.

---

## 1. Instalacja rozszerzenia

Zainstaluj oba rozszerzenia z Marketplace VS Code:

- **GitHub Copilot** (`GitHub.copilot`) – uzupełnienia inline
- **GitHub Copilot Chat** (`GitHub.copilot-chat`) – chat i tryb agenta

Lub przez terminal:

```bash
code --install-extension GitHub.copilot
code --install-extension GitHub.copilot-chat
```

Następnie zaloguj się na konto GitHub: `Ctrl+Shift+P` → **GitHub Copilot: Sign In**.

---

## 2. Instrukcje repozytorium

Utwórz plik `.github/copilot-instructions.md` w katalogu głównym repozytorium.  
Copilot automatycznie dołącza jego treść do każdego zapytania – nie wymaga żadnej dodatkowej konfiguracji.

Aby sprawdzić, czy plik jest rozpoznawany:
- Otwórz Copilot Chat
- Zapytaj: „Jakie są instrukcje tego repozytorium?"
- Copilot powinien odwołać się do zawartości pliku

> Plik działa zarówno w trybie inline, jak i w chat oraz agent mode.

---

## 3. Własne prompty (`.github/prompts/`)

Pliki `.prompt.md` w katalogu `.github/prompts/` stają się dostępne jako szybkie komendy w Copilot Chat.

**Użycie w VS Code:**
1. Otwórz Copilot Chat (`Ctrl+Alt+I`)
2. Wpisz `/` – zobaczysz listę dostępnych promptów z repozytorium
3. Wybierz prompt i uzupełnij parametry

**Struktura pliku `.prompt.md`:**

```markdown
---
mode: ask          # ask | edit | agent
description: Krótki opis widoczny w menu
---

Treść promptu z instrukcjami dla Copilota.
Możesz używać ${input:nazwa} dla parametrów dynamicznych.
```

---

## 4. Ustawienia `settings.json`

Plik `.vscode/settings.json` pozwala ujednolicić konfigurację Copilot dla całego zespołu.

**Przykładowy plik:** [`przyklady/.vscode/settings.json`](../przyklady/.vscode/settings.json)

Kluczowe ustawienia:

```json
{
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": true
  },
  "chat.agent.enabled": true,
  "github.copilot.chat.localeOverride": "pl",
  "github.copilot.renameSuggestions.triggerAutomatically": true
}
```

| Ustawienie | Opis |
|---|---|
| `github.copilot.enable` | Włącz/wyłącz Copilot per język (`"*"` = wszystkie) |
| `chat.agent.enabled` | Włącza tryb agenta w Copilot Chat |
| `github.copilot.chat.localeOverride` | Język odpowiedzi Copilota (np. `"pl"` dla polskiego) |
| `github.copilot.chat.codeGeneration.instructions` | Dodatkowe instrukcje dla generowania kodu – może wskazywać na plik (np. `.github/copilot-instructions.md`) |
| `github.copilot.chat.commitMessageGeneration.instructions` | Instrukcje do generowania wiadomości commit (np. format Conventional Commits) |

---

## 5. Konfiguracja MCP (`.vscode/mcp.json`)

MCP (Model Context Protocol) rozszerza możliwości agenta Copilot o zewnętrzne narzędzia.  
Plik `.vscode/mcp.json` definiuje serwery MCP dostępne w danym workspace.

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
    },
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
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

**Uruchomienie serwera MCP:**
1. Otwórz Copilot Chat i przełącz na tryb **Agent**
2. Kliknij ikonę narzędzi (🔧) – zobaczysz dostępne serwery MCP
3. Serwery uruchamiają się automatycznie przy pierwszym użyciu

> **Uwaga bezpieczeństwa:** Uruchamiaj tylko serwery MCP z zaufanych źródeł – mają dostęp do systemu plików i API.

---

## 6. Zalecane rozszerzenia (`.vscode/extensions.json`)

Plik `.vscode/extensions.json` sprawia, że VS Code proponuje instalację rozszerzeń nowym deweloperom.

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

## 7. Tryb agenta (Agent Mode)

Tryb agenta pozwala Copilotowi wykonywać wieloetapowe zadania autonomicznie (tworzenie plików, uruchamianie poleceń, edycja kodu).

**Włączenie:**
1. W `settings.json` ustaw `"chat.agent.enabled": true`
2. W Copilot Chat wybierz model z listy (np. GPT-4.1, Claude Sonnet)
3. Przełącz tryb na **Agent** (ikona w górnej części panelu chat)

**Przykładowe użycie:**
```
@workspace Utwórz nowy endpoint REST GET /users/{id} z walidacją i testami jednostkowymi
```

Agent samodzielnie:
- Przejrzy istniejący kod
- Utworzy nowe pliki
- Uruchomi testy
- Zaproponuje poprawki jeśli testy nie przejdą

---

## Podsumowanie konfiguracji

```
projekt/
├── .github/
│   ├── copilot-instructions.md          ← instrukcje repozytorium (auto-dołączane)
│   ├── mcp.json                         ← serwery MCP dla cloud agenta
│   ├── agents/
│   │   └── *.agent.md                   ← custom agenci (wybierane z dropdownu)
│   ├── hooks/
│   │   └── *.json                       ← lifecycle hooks agenta
│   ├── instructions/
│   │   └── *.instructions.md            ← instrukcje path-specific (applyTo)
│   ├── skills/
│   │   └── <nazwa>/SKILL.md             ← agent skills (ładowane automatycznie)
│   ├── workflows/
│   │   └── copilot-setup-steps.yml      ← środowisko cloud agenta
│   └── prompts/
│       └── *.prompt.md                  ← szablony promptów (menu /)
└── .vscode/
    ├── settings.json                    ← ustawienia Copilot dla workspace
    ├── mcp.json                         ← serwery MCP dla agenta lokalnego
    └── extensions.json                  ← zalecane rozszerzenia
```
