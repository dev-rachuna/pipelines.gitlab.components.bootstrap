# Helper: logo

Wyświetla baner ASCII z nazwą organizacji, typem pipeline oraz aktywną galezią repozytorium CI.

## Co robi

Definiuje i natychmiast wywoluje funkcje `logo()`, ktora drukuje ramke ASCII z:
- logotypem `rachuna-dev pl` w stylu blokowym,
- typem pipeline (`PIPELINE_TYPE`) wyrownany do prawej,
- galezią repozytorium CI (`GITLAB_CI_REPOSITORY_BRANCH`) wyrownana do prawej.

Przyklad wyjscia:

```
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
░░                                                                                                                              ░░
░░   █▀▄ █▀█ █▀▀ █ █ █ █ █▀█ █▀█     █▀█ █▀▀ ▀█▀    █▀█ █                                                                    ░░
░░   █▀▄ █▀█ █   █▀█ █ █ █ █ █▀█ ▄▄▄ █ █ █▀▀  █     █▀▀ █                                                                    ░░
░░   ▀ ▀ ▀ ▀ ▀▀▀ ▀ ▀ ▀▀▀ ▀ ▀ ▀ ▀     ▀ ▀ ▀▀▀  ▀  ▀  ▀   ▀▀▀                               ⚙️ PIPELINE CI ░░
░░                                                                                                  ❤️ main ░░
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
```

## Wymagane zmienne

| Zmienna | Skad pochodzi | Opis |
|---|---|---|
| `PIPELINE_TYPE` | `workflow/main.yml` | Typ pipeline: `CI` (branch/MR) lub `CD` (tag) |
| `GITLAB_CI_REPOSITORY_BRANCH` | `job-prepare.sh` | Galaz repozytorium CI |

## Uzycie w jobach

Helper jest wstrzykiwany po `logger` w `before_script` — wywoluje `logo()` automatycznie w momencie wstrzykniecia:

```yaml
before_script:
  - !reference [.helpers.logger.script.sh]
  - !reference [.helpers.job-prepare.script.sh]
  - !reference [.helpers.logo.script.sh]   # ← baner drukowany tu
  - !reference [.helpers.gitlab-tools.script.sh]
```

## Uwagi

- Helper nie jest wstrzykiwany w `after_script` — baner pojawia sie tylko na poczatku joba.
- `logo()` jest wywolywana natychmiast po definicji (ostatnia linia skryptu to `logo`).
