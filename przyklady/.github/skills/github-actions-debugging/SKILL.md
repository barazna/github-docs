---
name: github-actions-debugging
description: Przewodnik debugowania nieudanych workflow GitHub Actions. Używaj gdy prosisz o naprawienie błędów CI/CD lub nieudanych pipeline'ów.
allowed-tools: shell
---

Aby debugować nieudane workflow GitHub Actions, postępuj według poniższego procesu używając narzędzi z GitHub MCP Server:

## Proces debugowania

1. **Sprawdź ostatnie uruchomienia** – użyj narzędzia `list_workflow_runs` aby znaleźć nieudane uruchomienia powiązane z PR lub branchem

2. **Uzyskaj podsumowanie błędów** – użyj `summarize_job_log_failures` aby otrzymać podsumowanie AI bez wypełniania kontekstu tysiącami linii logów

3. **Pobierz szczegółowe logi** – jeśli podsumowanie nie wystarczy, użyj `get_job_logs` lub `get_workflow_run_logs` dla pełnych logów konkretnego joba

4. **Odtwórz błąd lokalnie** – spróbuj odtworzyć problem w lokalnym środowisku przed wprowadzeniem poprawek:
   ```bash
   # Uruchom te same kroki co w CI
   npm ci
   npm test
   npm run build
   ```

5. **Napraw błąd** – wprowadź poprawki i upewnij się że problem jest rozwiązany lokalnie

6. **Zweryfikuj poprawkę** – uruchom testy lokalnie i upewnij się że wszystkie przechodzą przed commitem

## Typowe błędy i rozwiązania

- **Brakujące zależności**: sprawdź `package.json` vs `package-lock.json`, uruchom `npm ci`
- **Błędy typowania TypeScript**: uruchom `npx tsc --noEmit`
- **Failing testy**: uruchom `npm test -- --reporter=verbose` dla szczegółów
- **Problemy z uprawnieniami**: sprawdź `permissions:` w pliku workflow
