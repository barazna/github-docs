---
mode: ask
description: Przeprowadź przegląd kodu zgodnie ze standardami projektu
---

Przeprowadź przegląd kodu dla zaznaczonego fragmentu lub aktualnego pliku.

Sprawdź następujące aspekty i zgłoś problemy według priorytetu:

## 🔴 Krytyczne (należy naprawić przed merge)

- Potencjalne luki bezpieczeństwa (injection, hardcoded secrets, brak walidacji wejścia)
- Brak obsługi błędów w operacjach, które mogą rzucić wyjątek
- Race conditions lub problemy z concurrency
- Wycieki pamięci

## 🟡 Ostrzeżenia (warto naprawić)

- Niespójna obsługa błędów w stosunku do reszty kodu
- Brak lub niedostateczne testy dla nowej logiki
- Złożoność cyklomatyczna > 10 (funkcja robi zbyt wiele rzeczy)
- Zduplikowany kod który powinien być wyekstrahowany
- TODO/FIXME bez przypisanego Issue

## 🔵 Sugestie (do rozważenia)

- Nazewnictwo niezgodne z konwencjami projektu
- Możliwość uproszczenia logiki
- Brakująca dokumentacja JSDoc dla eksportowanych elementów
- Możliwość użycia istniejącej abstrakcji zamiast nowej implementacji

## Format odpowiedzi

Dla każdego znalezionego problemu podaj:
- **Priorytet:** 🔴 / 🟡 / 🔵
- **Linia:** numer linii lub zakres
- **Problem:** krótki opis co jest nie tak
- **Sugestia:** jak to naprawić (opcjonalnie z przykładem kodu)

Jeśli kod jest poprawny – napisz krótkie potwierdzenie i wymień mocne strony implementacji.
