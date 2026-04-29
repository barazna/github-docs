# GitHub Enterprise – Agenci Copilot i Code Review

Konfiguracja GitHub Copilot coding agent oraz automatycznego przeglądu kodu (code review) w środowisku GitHub Enterprise.

---

## Wymagania wstępne

- GitHub Enterprise Cloud lub Server (najnowsza wersja)
- Licencja **GitHub Copilot Enterprise** przypisana do organizacji
- Uprawnienia administratora organizacji lub repozytorium
- GitHub Actions włączone w organizacji

---

## 1. Włączenie Copilot dla organizacji

1. Przejdź do **Organization Settings** → **Copilot** → **Access**
2. Wybierz **Allow for all members** lub przypisz konkretne zespoły
3. Włącz **Copilot in github.com** (wymagane dla agenta i code review)
4. Zapisz zmiany

> Użytkownicy muszą mieć aktywną licencję Copilot Business lub Enterprise.

---

## 2. Copilot Code Review (przegląd kodu)

### Włączenie na poziomie repozytorium

1. Przejdź do **Repository Settings** → **Copilot** → **Code review**
2. Włącz **Enable Copilot code review**
3. Wybierz tryb wyzwalania:
   - **Automatic** – Copilot automatycznie recenzuje każdy nowy PR
   - **On request** – Copilot dodaje review tylko gdy zostanie poproszony przez reviewera

### Użycie w Pull Requeście

**Automatyczne:** Copilot automatycznie doda review po otwarciu PR.

**Na żądanie:** W otwartym PR w sekcji **Reviewers** kliknij `+` i wybierz **Copilot** jako reviewera.

Copilot:
- Przegląda zmienione pliki
- Dodaje komentarze inline z sugestiami
- Może sugerować konkretne poprawki kodu (jako `suggestion` block)
- Wystawia review: `APPROVED`, `REQUEST_CHANGES` lub `COMMENTED`

### Konfiguracja zasad review

W pliku `.github/copilot-instructions.md` możesz opisać co Copilot powinien sprawdzać:

```markdown
## Wytyczne do code review

- Sprawdzaj czy każda funkcja publiczna ma testy jednostkowe
- Weryfikuj obsługę błędów – brak try/catch lub walidacji wejścia to błąd
- Oznacz TODO jako warning jeśli nie mają przypisanego issue
- Sprawdzaj czy nie ma hardcoded secrets (klucze API, hasła)
- Weryfikuj zgodność z konwencjami nazewnictwa projektu
```

---

## 3. Copilot Coding Agent (zadania autonomiczne)

Agent Copilot może wykonywać całe zadania programistyczne – tworzy branch, pisze kod, otwiera PR.

### Włączenie agenta

1. **Organization Settings** → **Copilot** → **Coding agent**
2. Włącz **Enable Copilot coding agent**
3. Opcjonalnie: ogranicz do wybranych repozytoriów

### Przypisywanie zadań agentowi

**Przez Issues:**
1. Otwórz lub utwórz Issue
2. Kliknij **Assignees** → wybierz **Copilot** jako wykonawcę
3. Agent przeanalizuje Issue i automatycznie:
   - Stworzy nowy branch
   - Zaimplementuje rozwiązanie
   - Otworzy Pull Request

**Przez Copilot Chat na GitHub.com:**
```
@copilot Zaimplementuj funkcję walidacji adresu email zgodnie z RFC 5322
```

### Uprawnienia agenta

Agent działa z ograniczonymi uprawnieniami. Konfiguracja w **Repository Settings** → **Actions** → **General** → **Workflow permissions**:

```yaml
# .github/workflows/copilot-agent.yml (opcjonalnie dla zaawansowanych scenariuszy)
permissions:
  contents: write        # Tworzenie commitów i branchy
  pull-requests: write   # Otwieranie i aktualizowanie PR
  issues: read           # Odczyt treści Issue
```

---

## 4. Konfiguracja środowiska agenta (`copilot-setup-steps.yml`)

Plik `.github/workflows/copilot-setup-steps.yml` definiuje środowisko uruchomieniowe dla coding agenta. Kroki w tym pliku są wykonywane **przed każdym zadaniem agenta** – instalują zależności, konfigurują runtime i przygotowują projekt do pracy.

**Ważne:** Zadanie musi mieć nazwę dokładnie `copilot-setup-steps`. Plik musi być na domyślnym branchu repozytorium.

**Przykładowy plik:** [`przyklady/.github/workflows/copilot-setup-steps.yml`](../przyklady/.github/workflows/copilot-setup-steps.yml)

```yaml
name: Copilot Setup Steps

on:
  workflow_dispatch:

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
```

> Jeśli setup steps nie powiedzie się, agent i tak uruchomi zadanie – nie blokuje pracy, ale środowisko może być niekompletne.

---

## 5. Konfiguracja MCP dla agenta chmurowego

Plik `.github/mcp.json` definiuje serwery MCP dostępne dla agenta działającego w GitHub Actions.

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

> `${GITHUB_TOKEN}` jest automatycznie dostępny w środowisku GitHub Actions – nie wymaga ręcznej konfiguracji.

---

## 6. Polityki organizacji

### Zarządzanie dostępem do MCP

W **Organization Settings** → **Copilot** → **MCP** możesz:
- Zdefiniować allowlistę dozwolonych serwerów MCP
- Zablokować wszystkie zewnętrzne serwery MCP
- Wymagać zatwierdzenia administratora dla nowych serwerów

### Branch protection z Copilot

Aby wymagać przeglądu Copilot przed scaleniem PR:

1. **Repository Settings** → **Branches** → **Branch protection rules**
2. Włącz **Require a pull request before merging**
3. Opcjonalnie włącz **Require review from Code Owners**

W pliku `.github/CODEOWNERS`:
```
# Copilot automatycznie reviewuje wszystkie pliki TypeScript
*.ts @org/copilot-reviewers
src/ @org/backend-team
```

---

## 7. Workflow automatyczny (GitHub Actions)

Możesz wyzwolić agenta Copilot automatycznie przez GitHub Actions:

```yaml
# .github/workflows/copilot-review.yml
name: Copilot Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  copilot-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - name: Request Copilot Review
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.pulls.requestReviewers({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.payload.pull_request.number,
              reviewers: ['copilot-pull-request-reviewer[bot]']
            });
```

---

## 8. Monitorowanie i audyt

**Logi aktywności agenta:**
- **Organization Settings** → **Audit log** → filtruj po `copilot`
- Każda akcja agenta (commit, komentarz, PR) jest logowana

**Statystyki użycia:**
- **Organization Settings** → **Copilot** → **Usage** – przegląd aktywności per użytkownik i repozytorium

---

## Podsumowanie

| Funkcja | Gdzie włączyć | Wymagania |
|---|---|---|
| Code Review | Repo Settings → Copilot | Licencja Enterprise |
| Coding Agent | Org Settings → Copilot | Licencja Enterprise |
| Środowisko agenta | `.github/workflows/copilot-setup-steps.yml` | Coding Agent włączony |
| MCP (cloud) | `.github/mcp.json` | Coding Agent włączony |
| Polityki MCP | Org Settings → Copilot → MCP | Admin Org |
| Automatyczne workflow | `.github/workflows/` | GitHub Actions |
