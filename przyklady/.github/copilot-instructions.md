# Instrukcje GitHub Copilot

## Stack technologiczny

- **Język:** TypeScript 5.x
- **Runtime:** Node.js 20 LTS
- **Framework:** Express 4.x
- **Baza danych:** PostgreSQL 15 (Prisma ORM)
- **Testy:** Vitest, Supertest
- **Linter:** ESLint + Prettier

## Standardy kodu

- Używaj `async/await` zamiast `.then()` / `.catch()`
- Eksportowane funkcje i klasy muszą mieć typy TypeScript (bez `any` bez uzasadnienia)
- Każda funkcja publiczna musi mieć testy jednostkowe
- Obsługuj błędy explicite – nie łykaj wyjątków przez pusty `catch`
- Nie używaj `console.log` w kodzie produkcyjnym – używaj loggera (`src/lib/logger.ts`)

## Konwencje nazewnictwa

- **Pliki:** `kebab-case.ts` (np. `user-service.ts`)
- **Klasy i interfejsy:** `PascalCase` (np. `UserService`, `IUserRepository`)
- **Zmienne i funkcje:** `camelCase` (np. `getUserById`)
- **Stałe:** `UPPER_SNAKE_CASE` (np. `MAX_RETRY_COUNT`)
- **Tabele w bazie:** `snake_case` (np. `user_profiles`)

## Struktura katalogów

```
src/
├── controllers/   # Obsługa żądań HTTP
├── services/      # Logika biznesowa
├── repositories/  # Dostęp do danych (Prisma)
├── middleware/    # Express middleware
├── lib/           # Narzędzia pomocnicze (logger, walidacja)
└── types/         # Typy TypeScript
tests/
├── unit/          # Testy jednostkowe (Vitest)
└── integration/   # Testy integracyjne (Supertest)
```

## Bezpieczeństwo

- Nigdy nie umieszczaj sekretów, kluczy API ani haseł w kodzie
- Używaj zmiennych środowiskowych (`.env`) i biblioteki `zod` do ich walidacji
- Waliduj wszystkie dane wejściowe od użytkownika przed przetwarzaniem
- Stosuj parametryzowane zapytania – nie buduj SQL przez konkatenację stringów

## Code Review

- Sprawdzaj czy każda nowa funkcja ma odpowiednie testy
- Weryfikuj obsługę błędów i przypadki brzegowe
- Upewnij się że nowe endpointy API mają dokumentację OpenAPI
- Flagi TODO bez przypisanego Issue traktuj jako ostrzeżenie
