# Konfiguracja IntelliJ / JetBrains dla GitHub Copilot

Jak skonfigurować IntelliJ IDEA i inne IDE JetBrains, aby Copilot używał instrukcji, promptów i serwerów MCP.

---

## 1. Instalacja pluginu

1. Otwórz **Settings** (`Ctrl+Alt+S`) → **Plugins**
2. Wyszukaj `GitHub Copilot`
3. Kliknij **Install**, następnie **Restart IDE**

Alternatywnie zainstaluj z [JetBrains Marketplace](https://plugins.jetbrains.com/plugin/17718-github-copilot).

**Wymagania:**
- IntelliJ IDEA 2023.1 lub nowszy
- Plugin GitHub Copilot ≥ 1.5 (dla obsługi custom instructions)

**Logowanie:**
- Po instalacji kliknij ikonę Copilot w prawym dolnym rogu
- Wybierz **Sign in to GitHub** i zakończ autoryzację OAuth

---

## 2. Instrukcje repozytorium

Plik `.github/copilot-instructions.md` w katalogu głównym projektu jest automatycznie rozpoznawany przez plugin Copilot w JetBrains.

Nie wymaga dodatkowej konfiguracji – Copilot dołącza go do każdego zapytania chat.

**Weryfikacja działania:**
1. Otwórz Copilot Chat (ikonka w prawym pasku lub `Alt+Shift+C`)
2. Zapytaj: „Jakie są instrukcje tego projektu?"
3. Odpowiedź powinna odwoływać się do zawartości pliku

**Przykładowy plik:** [`przyklady/.github/copilot-instructions.md`](../przyklady/.github/copilot-instructions.md)

---

## 3. Globalne instrukcje (poza repozytorium)

Instrukcje globalne stosowane są we **wszystkich projektach** na danej maszynie.

| System | Ścieżka pliku |
|---|---|
| macOS / Linux | `~/.config/github-copilot/intellij/global-copilot-instructions.md` |
| Windows | `%LOCALAPPDATA%\github-copilot\intellij\global-copilot-instructions.md` |

**Przykładowa zawartość:**

```markdown
# Globalne instrukcje Copilot

- Preferuj czytelność kodu nad zwięzłością
- Używaj języka polskiego w komentarzach
- Zawsze dodawaj obsługę błędów (try/catch lub Result type)
- Nie generuj kodu produkcyjnego z `console.log` / `System.out.println`
```

> Instrukcje globalne są łączone z instrukcjami repozytorium (`.github/copilot-instructions.md`).

---

## 4. Własne prompty

W JetBrains prompty z katalogu `.github/prompts/` są dostępne w Copilot Chat.

**Użycie:**
1. Otwórz Copilot Chat
2. Kliknij ikonę `/` lub wpisz `/` w polu czatu
3. Wybierz prompt z listy

> Obsługa promptów dostępna od wersji pluginu 1.5+. Upewnij się, że używasz aktualnej wersji.

---

## 5. Konfiguracja MCP

MCP (Model Context Protocol) w JetBrains 2025.1+ umożliwia Copilotowi korzystanie z zewnętrznych narzędzi.

### Konfiguracja przez Settings (GUI)

1. Otwórz **Settings** (`Ctrl+Alt+S`) → **Tools** → **GitHub Copilot** → **MCP Servers**
2. Kliknij `+` aby dodać nowy serwer
3. Wypełnij formularz:

| Pole | Przykład |
|---|---|
| Name | `github` |
| Type | `http` lub `stdio` |
| URL / Command | `https://api.githubcopilot.com/mcp/` |

### Konfiguracja przez plik JSON

Plik konfiguracji MCP dla JetBrains jest przechowywany lokalnie:

| System | Ścieżka |
|---|---|
| macOS / Linux | `~/.config/github-copilot/jetbrains/mcp-servers.json` |
| Windows | `%APPDATA%\github-copilot\jetbrains\mcp-servers.json` |

**Przykładowa zawartość:**

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer <TWÓJ_PERSONAL_ACCESS_TOKEN>"
      }
    },
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/ścieżka/do/projektu"]
    }
  }
}
```

> Zastąp `<TWÓJ_PERSONAL_ACCESS_TOKEN>` rzeczywistym tokenem PAT z wymaganymi zakresami: `repo`, `read:org`.

### Użycie MCP w Copilot Chat

1. Otwórz Copilot Chat i przełącz na tryb **Agent**
2. Kliknij ikonę narzędzi – zobaczysz dostępne serwery MCP
3. Serwery uruchamiają się automatycznie przy pierwszym wywołaniu narzędzia

**Przykład użycia GitHub MCP:**
```
Pobierz listę otwartych Pull Requestów w tym repozytorium i podsumuj co zawierają
```

---

## 6. Ustawienia pluginu

Dostęp do ustawień: **Settings** → **Tools** → **GitHub Copilot**

| Ustawienie | Opis |
|---|---|
| **Enable Copilot** | Włącz/wyłącz globalnie lub per język |
| **Enable for current language** | Szybkie przełączenie dla aktualnego pliku |
| **Show completions automatically** | Podpowiedzi inline automatycznie lub na żądanie (`Alt+\`) |
| **Enable GitHub Copilot Chat** | Włącz panel chat |

**Skróty klawiszowe:**

| Akcja | Skrót |
|---|---|
| Zaakceptuj sugestię | `Tab` |
| Odrzuć sugestię | `Esc` |
| Następna sugestia | `Alt+]` |
| Poprzednia sugestia | `Alt+[` |
| Otwórz chat | `Alt+Shift+C` |
| Podpowiedź na żądanie | `Alt+\` |

---

## 7. Weryfikacja konfiguracji

Po skonfigurowaniu sprawdź czy wszystko działa poprawnie:

```
✅ Plugin zainstalowany i zalogowany (ikona w prawym dolnym rogu świeci)
✅ Copilot Chat dostępny (ikona na pasku bocznym)
✅ Instrukcje repozytorium rozpoznane (zapytaj Copilota o instrukcje projektu)
✅ Serwery MCP widoczne w trybie Agent (lista narzędzi)
```

---

## Podsumowanie

```
Repozytorium:
  .github/copilot-instructions.md            ← instrukcje projektu (auto-dołączane)
  .github/prompts/*.prompt.md                ← szablony promptów
  .github/workflows/copilot-setup-steps.yml  ← środowisko cloud agenta

Lokalnie na maszynie:
  ~/.config/github-copilot/intellij/
    global-copilot-instructions.md  ← globalne instrukcje
    mcp-servers.json                ← konfiguracja serwerów MCP
```
