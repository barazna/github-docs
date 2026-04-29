---
mode: ask
description: Generuj dokumentację JSDoc / OpenAPI dla zaznaczonego kodu
---

Wygeneruj dokumentację dla zaznaczonego kodu lub pliku.

## Dla funkcji TypeScript – generuj JSDoc

Format:
```typescript
/**
 * [Krótki opis co robi funkcja – jedno zdanie]
 *
 * [Opcjonalnie: dłuższy opis jeśli logika jest złożona]
 *
 * @param {Typ} nazwaParametru - Opis parametru
 * @returns {Typ} Opis zwracanej wartości
 * @throws {NazwaError} Kiedy i dlaczego rzuca błąd
 *
 * @example
 * const wynik = await nazwaFunkcji(parametr);
 */
```

Zasady:
- Opis powinien wyjaśniać **co** robi funkcja, nie **jak**
- Dla parametrów opisz ograniczenia (np. "UUID w formacie v4", "wartość 1-100")
- Dodaj `@example` dla nieoczywistego użycia
- Dla metod `async` zaznacz to w opisie lub przez typ zwracany (`Promise<T>`)

## Dla endpointów Express/REST – generuj OpenAPI (YAML)

Format:
```yaml
/ścieżka/{parametr}:
  get:
    summary: Krótki opis endpointu
    description: Dłuższy opis (opcjonalnie)
    tags:
      - NazwaGrupy
    parameters:
      - name: parametr
        in: path
        required: true
        schema:
          type: string
          format: uuid
    responses:
      '200':
        description: Sukces
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/NazwaModelu'
      '404':
        description: Zasób nie znaleziony
      '500':
        description: Błąd serwera
```

## Dodatkowe wskazówki

- Używaj języka polskiego dla opisów skierowanych do deweloperów projektu
- Dla parametrów wyliczeniowych wylistuj dozwolone wartości
- Jeśli funkcja modyfikuje stan globalny lub ma efekty uboczne – zaznacz to w dokumentacji
