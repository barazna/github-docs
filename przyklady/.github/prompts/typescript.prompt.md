---
mode: edit
description: Generuj kod TypeScript zgodny ze standardami projektu
---

Napisz kod TypeScript zgodny z poniższymi zasadami:

1. **Typy:** Używaj ścisłego typowania – bez `any`. Eksportowane funkcje muszą mieć jawne typy parametrów i zwracanej wartości.

2. **Obsługa błędów:** Każda operacja async musi mieć obsługę błędów przez `try/catch` lub `Result<T, E>`.

3. **Nazewnictwo:**
   - Funkcje i zmienne: `camelCase`
   - Klasy i interfejsy: `PascalCase`
   - Stałe: `UPPER_SNAKE_CASE`

4. **Importy:** Sortuj importy: najpierw moduły Node/zewnętrzne, potem wewnętrzne (`@/`).

5. **Testy:** Do każdej nowej funkcji dodaj szkielet testu jednostkowego Vitest:
   ```typescript
   import { describe, it, expect } from 'vitest';
   ```

6. **Dokumentacja:** Dodaj JSDoc dla eksportowanych funkcji:
   ```typescript
   /**
    * @param id - Identyfikator zasobu
    * @returns Znaleziony zasób lub null jeśli nie istnieje
    */
   ```

Przykład dobrze napisanej funkcji:

```typescript
import { db } from '@/lib/db';
import { logger } from '@/lib/logger';

export interface User {
  id: string;
  email: string;
  createdAt: Date;
}

/**
 * Pobiera użytkownika po ID.
 * @param id - UUID użytkownika
 * @returns Użytkownik lub null jeśli nie znaleziono
 */
export async function getUserById(id: string): Promise<User | null> {
  try {
    return await db.user.findUnique({ where: { id } });
  } catch (error) {
    logger.error('Błąd pobierania użytkownika', { id, error });
    throw error;
  }
}
```
