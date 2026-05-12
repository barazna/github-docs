# Optymalizacja tokenów i przyspieszenie pracy agenta AI

Zbiór wskazówek, dobrych praktyk i technik pozwalających zmniejszyć zużycie tokenów, przyspieszyć odpowiedzi Copilota i usprawnić pracę agenta AI w codziennym developmencie.

---

## Dlaczego warto optymalizować tokeny?

Każde zapytanie do Copilota (i każda iteracja agenta) konsumuje kontekst – okno tokenów ma ograniczoną pojemność. Zbyt duży lub nieistotny kontekst powoduje:

- **Wolniejsze odpowiedzi** – model musi przetworzyć więcej danych przed wygenerowaniem odpowiedzi
- **Gorszą jakość** – ważne informacje „toną" w szumie nieistotnych danych
- **Wyższe koszty** – w płatnych planach i API bezpośrednio przekłada się na rachunek
- **Utratę kontekstu** – przy bardzo długich sesjach wcześniejsze informacje są wycinane

---

## 1. Zwięzłe instrukcje repozytorium

### Problem

`copilot-instructions.md` jest dołączany do **każdego** zapytania. Zbyt rozbudowany plik podnosi koszt wszystkich interakcji.

### Dobre praktyki

- **Skup się na tym, czego Copilot nie wie** – nie opisuj rzeczy oczywistych (np. „używaj if/else do warunków")
- **Maksymalnie 100–200 linii** – dłuższy plik rzadko przekłada się na lepsze wyniki
- **Konkret zamiast prozy** – listy wypunktowane zamiast długich paragrafów

```markdown
❌ ŹLE – zbyt ogólne i obszerne:
Nasz projekt używa TypeScript, który jest nadzbiorem JavaScriptu i dodaje
statyczne typowanie. Staramy się pisać czysty kod zgodny z dobrymi
praktykami inżynierii oprogramowania...

✅ DOBRZE – konkretne reguły:
- Stack: TypeScript 5, Node.js 20, Express 4, Vitest
- Styl: async/await, bez `any`, `interface` zamiast `type`
- Testy: Vitest dla jednostkowych, Supertest dla integracyjnych
- Konwencje: kebab-case pliki, PascalCase klasy, camelCase zmienne
```

### Czego unikać w `copilot-instructions.md`

- Długich przykładów kodu – lepiej odwołać się do istniejących plików projektu
- Historii projektu lub tła biznesowego nieistotnego dla generowania kodu
- Szczegółów konfiguracji środowiska – te powinny być w `copilot-setup-steps.yml`

---

## 2. Instrukcje path-specific zamiast globalnych

### Problem

Instrukcje dotyczące Reacta nie są potrzebne gdy Copilot pracuje nad plikami backendowymi – niepotrzebnie zwiększają rozmiar kontekstu.

### Rozwiązanie: `applyTo` w `.github/instructions/`

Przenieś specyficzne reguły do osobnych plików z `applyTo`:

```markdown
<!-- .github/instructions/react.instructions.md -->
---
applyTo: "src/frontend/**/*.{tsx,jsx}"
---

- Komponenty tylko funkcyjne (hooks zamiast klas)
- Props z interfejsem TypeScript, nie inline type
- Stylowanie przez CSS Modules
```

```markdown
<!-- .github/instructions/backend.instructions.md -->
---
applyTo: "src/api/**/*.ts"
---

- Każdy endpoint musi walidować input przez Zod
- Zwracaj `Result<T, Error>` zamiast rzucać wyjątki
- Logi przez Pino, nie console.log
```

**Efekt:** Copilot ładuje tylko instrukcje pasujące do aktualnie edytowanego pliku – znacząco mniej tokenów przy każdej interakcji.

---

## 3. Precyzyjne dołączanie kontekstu

### Używaj `#file` i `#symbol` zamiast `@workspace`

| Zmienna | Zasięg | Koszt tokenów | Kiedy używać |
|---|---|---|---|
| `@workspace` | Całe repozytorium (indeks) | Wysoki | Tylko przy wyszukiwaniu cross-repo |
| `#file:ścieżka` | Jeden plik | Niski | Gdy Copilot potrzebuje konkretnego pliku |
| `#symbol:nazwa` | Jeden symbol | Bardzo niski | Gdy potrzebujesz definicji funkcji/klasy |
| `#selection` | Zaznaczony fragment | Minimalny | Przy pracy z konkretnym kodem |

```
❌ WOLNE: @workspace znajdź jak walidujemy email

✅ SZYBKIE: #file:src/validators/email.ts jak działa walidacja?

✅ SZYBKIE: #symbol:validateEmail sprawdź czy obsługuje plus w adresie
```

### Wskazuj konkretne pliki jako kontekst

Przed wysłaniem zadania do agenta dołącz tylko pliki bezpośrednio związane z zadaniem:

```
Popraw błąd w walidacji – kontekst: #file:src/validators/email.ts #file:tests/email.test.ts
```

---

## 4. Plik `.copilotignore`

Utwórz plik `.copilotignore` w katalogu głównym, aby wykluczyć z indeksu pliki nieistotne dla Copilota:

```
# .copilotignore

# Artefakty budowania i zależności
node_modules/
dist/
build/
.next/
coverage/

# Wygenerowane pliki
*.generated.ts
*.min.js
*.lock

# Dane i zasoby
*.png
*.jpg
*.svg
*.pdf
data/fixtures/
migrations/

# Pliki konfiguracyjne rzadko potrzebne Copilotowi
.env*
*.pem
```

**Efekt:** Mniejszy indeks = szybsze przeszukiwanie, mniej szumu w `@workspace`.

---

## 5. Formułowanie efektywnych promptów

### Zasada jednej odpowiedzialności

Jeden prompt = jedno dobrze zdefiniowane zadanie.

```
❌ ZŁE – zbyt szeroki zakres:
"Zrób żeby aplikacja działała lepiej"

✅ DOBRE – jeden konkretny cel:
"Zrefaktoruj funkcję `processOrder` w src/orders/processor.ts
żeby używała async/await zamiast Promise chain"
```

### Podaj oczekiwany format wyjścia

Model generuje mniej iteracji gdy wie dokładnie co ma zwrócić:

```
Napisz test jednostkowy dla funkcji `calculateTax`.
Format: Vitest, jeden blok describe z 3 przypadkami:
1. podatek 23% dla kwoty > 0
2. brak podatku dla kwoty = 0
3. rzucenie błędu dla kwoty ujemnej
```

### Technika „najpierw plan"

Dla złożonych zadań poproś o plan zanim zacznie kodować – unikasz przerabiania dużych fragmentów kodu:

```
Zanim napiszesz kod, opisz krótko (3–5 kroków) jak zaimplementujesz
autoryzację JWT middleware dla Express. Zatwierdź plan, a potem zacznij kodować.
```

---

## 6. Efektywna praca w trybie agenta

### Zawężaj uprawnienia narzędzi

W plikach `.agent.md` podaj tylko narzędzia potrzebne do zadania:

```markdown
<!-- .github/agents/tester.agent.md -->
---
name: tester
description: Pisze i uruchamia testy jednostkowe
tools: ["read", "edit", "shell"]   ← NIE dawaj "browser", "search" gdy niepotrzebne
---
```

### Używaj hooks do early-stopping

Hook `preToolUse` może zablokować kosztowne operacje przed ich wykonaniem:

```json
{
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": "[ \"$TOOL_NAME\" != 'search_web' ] || exit 1",
        "timeoutSec": 5
      }
    ]
  }
}
```

### Rozbijaj duże zadania na mniejsze

Agent iterujący nad jednym dużym zadaniem zużywa znacznie więcej tokenów niż seria małych, skupionych zadań:

```
❌ ZŁE – jedno ogromne zadanie:
"Dodaj system autoryzacji z rolami, JWT, refresh tokenami,
wylogowaniem i obsługą wygasłych tokenów"

✅ DOBRE – seria małych kroków:
Krok 1: "Dodaj JWT middleware dla Express – tylko weryfikacja tokena"
Krok 2: "Dodaj endpoint /auth/refresh – używa refreshToken z cookie"
Krok 3: "Dodaj role-based access control do istniejącego middleware"
```

---

## 7. Zarządzanie kontekstem chat

### Rozpoczynaj nową sesję przy zmianie tematu

Długie rozmowy akumulują kontekst. Gdy zmieniasz temat (np. z debugowania na nowe feature) – otwórz nowe okno czatu.

### Streszczaj kontekst zamiast powtarzać

Zamiast przewijać i cytować wcześniejsze fragmenty rozmowy:

```
❌ ZŁE: "[przekleja 20 linii kodu z poprzedniej wiadomości] a teraz zmień to tak że..."

✅ DOBRE: "W funkcji sendEmail którą już widziałeś – dodaj obsługę błędu timeout"
```

### Wyczyść kontekst przed skomplikowanym zadaniem

Przed zadaniem wymagającym dużo kontekstu (np. refaktor całego modułu) – zacznij od świeżego chatu z jednym skoncentrowanym promptem zawierającym tylko niezbędne pliki.

---

## 8. Wybór modelu do zadania

Cięższy model ≠ lepszy wynik dla każdego zadania. Dopasuj model do złożoności:

| Zadanie | Rekomendowany model | Dlaczego |
|---|---|---|
| Uzupełnienie kodu inline | GPT-4o mini / szybki | Prostsze zadanie, liczy się latencja |
| Wyjaśnienie błędu | GPT-4.1 | Wystarczający do analizy |
| Refaktor modułu | Claude Sonnet / GPT-4.1 | Dobry balans jakości i szybkości |
| Architektura systemu | Claude Opus / o3 | Złożone rozumowanie |
| Pisanie testów | GPT-4.1 | Powtarzalne zadanie z wyraźnym wzorcem |

W `.github/agents/*.agent.md` ustaw model per agent:

```markdown
---
name: quick-fixer
model: gpt-4o-mini   ← lżejszy model dla szybkich poprawek
tools: ["read", "edit"]
---
```

---

## 9. Optymalizacja `copilot-setup-steps.yml`

Długi setup = długi czas oczekiwania na start agenta chmurowego.

### Cachowanie zależności

```yaml
jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'          ← cachuje node_modules między uruchomieniami
      - run: npm ci
```

### Instaluj tylko to co potrzebne

```yaml
# Zamiast: npm ci  (instaluje WSZYSTKIE zależności łącznie z dev)
# Gdy agent tylko czyta i edytuje kod:
- run: npm ci --omit=dev    ← pomija devDependencies jeśli nie są potrzebne agentowi
```

### Ograniczaj niepotrzebne kroki setupu

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 1          ← płytki clone zamiast pełnej historii git
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'
  - run: npm ci
  # NIE dodawaj kroków budowania/lintowania – agent sam je wykona gdy potrzeba
```

---

## 10. Struktura projektu przyjazna agentowi

Dobrze zorganizowany projekt = mniej pytań = mniej iteracji = mniej tokenów.

### Samodokumentujące nazwy plików i katalogów

```
❌ src/utils/h.ts, src/utils/misc.ts, src/utils/stuff.ts

✅ src/validators/email-validator.ts
   src/formatters/currency-formatter.ts
   src/handlers/auth-error-handler.ts
```

### Barrel files (index.ts) dla modułów

Agent nie musi przeszukiwać całego katalogu gdy istnieje jasny punkt wejścia:

```typescript
// src/validators/index.ts
export { validateEmail } from './email-validator';
export { validatePhone } from './phone-validator';
export { validatePostalCode } from './postal-code-validator';
```

### Komentarze JSDoc przy publicznych API

Agent rozumie interfejs modułu bez czytania implementacji:

```typescript
/**
 * Waliduje adres email zgodnie z RFC 5322.
 * @param email - Adres do sprawdzenia
 * @returns true jeśli email jest prawidłowy
 * @throws {ValidationError} gdy email ma nieprawidłowy format
 */
export function validateEmail(email: string): boolean { ... }
```

---

## Podsumowanie – checklista optymalizacji

| Praktyka | Oszczędność tokenów | Trudność wdrożenia |
|---|---|---|
| Zwięzłe `copilot-instructions.md` (< 200 linii) | ⭐⭐⭐ Wysoka | Niska |
| Instrukcje path-specific (`applyTo`) | ⭐⭐⭐ Wysoka | Niska |
| `.copilotignore` – wykluczenie artefaktów | ⭐⭐ Średnia | Bardzo niska |
| `#file` zamiast `@workspace` | ⭐⭐⭐ Wysoka | Bardzo niska |
| Precyzyjne prompty z oczekiwanym formatem | ⭐⭐⭐ Wysoka | Niska |
| Rozbijanie dużych zadań agenta | ⭐⭐⭐ Wysoka | Niska |
| Cache w `copilot-setup-steps.yml` | – (czas, nie tokeny) | Bardzo niska |
| Lżejszy model dla prostych zadań | ⭐⭐ Średnia | Niska |
| Nowa sesja chat przy zmianie tematu | ⭐⭐ Średnia | Bardzo niska |
| JSDoc przy publicznych API | ⭐⭐ Średnia | Średnia |
