# GitHub Copilot – Dokumentacja konfiguracji

Przewodnik opisujący pliki wymagane i opcjonalne dla GitHub Copilot, konfigurację środowisk IDE oraz ustawienia GitHub Enterprise dla agentów AI.

## Spis treści

| Dokument | Opis |
|---|---|
| [Pliki GitHub Copilot](docs/01-pliki-copilot.md) | Jakie pliki Copilot potrzebuje, które są wymagane, a które opcjonalne |
| [Konfiguracja VS Code](docs/02-konfiguracja-vscode.md) | Ustawienia VS Code – instrukcje, prompty, MCP, tryb agenta |
| [Konfiguracja IntelliJ](docs/03-konfiguracja-intellij.md) | Ustawienia JetBrains/IntelliJ – instrukcje, prompty, MCP |
| [GitHub Enterprise – Agenci](docs/04-github-enterprise.md) | Konfiguracja agentów Copilot i code review w GitHub Enterprise |

## Przykładowe pliki

Gotowe szablony do skopiowania do projektu znajdują się w katalogu [`przyklady/`](przyklady/).

```
przyklady/
├── .github/
│   ├── copilot-instructions.md   # Instrukcje repozytorium dla Copilota
│   ├── mcp.json                  # Konfiguracja MCP dla agenta chmurowego
│   ├── workflows/
│   │   └── copilot-setup-steps.yml  # Środowisko coding agenta
│   └── prompts/
│       ├── typescript.prompt.md  # Prompt dla kodu TypeScript
│       ├── code-review.prompt.md # Prompt do przeglądu kodu
│       └── dokumentacja.prompt.md # Prompt do generowania dokumentacji
└── .vscode/
    ├── settings.json             # Ustawienia VS Code dla Copilot
    ├── mcp.json                  # Konfiguracja serwerów MCP dla VS Code
    └── extensions.json           # Zalecane rozszerzenia
```

## Szybki start

1. Skopiuj `.github/copilot-instructions.md` do swojego repozytorium i dostosuj do projektu.
2. Dodaj `.vscode/settings.json` i `.vscode/mcp.json` dla konfiguracji VS Code.
3. Przejrzyj docs/ aby skonfigurować IDE oraz GitHub Enterprise.
