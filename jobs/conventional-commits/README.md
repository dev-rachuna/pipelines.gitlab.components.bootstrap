# Job: conventional-commits

Sprawdza, czy commity na feature branchu są zgodne ze standardem [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

## Co robi

Analizuje commity między `origin/main` a `HEAD` i waliduje tytuł każdego commita wyrażeniem regularnym. Job kończy się błędem jeśli którykolwiek commit nie jest zgodny z wzorcem.

```bash
git --no-pager log origin/$CI_DEFAULT_BRANCH..HEAD --pretty=format:"%s"
```

## Jak naprawić błąd walidacji

Popraw tytuł commita: `git commit --amend` (dla ostatniego) lub `git rebase -i` (dla wielu), zmień message na zgodny ze wzorcem, a następnie `git push --force-with-lease`.

## Przykłady

> [!important] Opis polecenia
> To polecenie wyświetla listę commitów z lokalnej gałęzi (HEAD) względem domyślnego brancha - `main`, czyli badany jest tylko przyrost.

> [!tip] Przykłady poprawnych commitów
> - ✔️ feat(api): Zmiana biznesowa
> - ✔️ feat: Zmiana biznesowa
> - ✔️ feat!: Zmiana biznesowa

> [!caution] Przykłady niepoprawnych commitów
> - ❌ Zmiana biznesowa
> - ❌ feat : Zmiana biznesowa
> - ❌ feat :Zmiana biznesowa
> - ❌ !feat :Zmiana biznesowa
> - ❌ Feat :Zmiana biznesowa
> - ❌ feature :Zmiana biznesowa

## Zmienne

| Zmienna | Wartość | Opis |
|---------|---------|------|
| `DOCS_MD_FILE_PATH` | `common/jobs/conventional-commits/README.md` | Ścieżka do tej dokumentacji |
