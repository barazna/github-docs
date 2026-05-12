---
applyTo: "**/*.ts,**/*.tsx"
description: Zasady kodowania TypeScript stosowane dla plików TS/TSX.
---

# Instrukcje dla plików TypeScript

## Typowanie

- Zawsze używaj ścisłego typowania – bez `any` bez uzasadnienia w komentarzu
- Eksportowane funkcje muszą mieć jawne typy parametrów i wartości zwracanej
- Używaj `interface` zamiast `type` dla obiektów danych (wyjątek: union types, mapped types)
- Włącz `strict: true` w `tsconfig.json`

## Obsługa błędów

- Każda operacja `async` musi mieć obsługę błędów przez `try/catch`
- Nie łykaj wyjątków przez pusty `catch` – zawsze loguj lub rzucaj dalej
- Używaj loggera (`src/lib/logger.ts`), nie `console.log`

## Importy

- Sortuj importy: najpierw moduły Node.js/zewnętrzne, potem wewnętrzne (`@/`)
- Używaj ścieżek aliasów (`@/`) zamiast ścieżek względnych gdzie możliwe

## Testy

- Każda eksportowana funkcja musi mieć testy jednostkowe w Vitest
- Testy umieszczaj w `tests/unit/` z sufiksem `.test.ts`
