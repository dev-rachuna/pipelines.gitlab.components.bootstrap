# Helper: job-prepare

Inicjalizuje środowisko bash oraz parsuje `CI_CONFIG_PATH` w celu wyekstrahowania danych o repozytorium CI.

## Co robi

1. Ustawia opcje bash: `set -eaou pipefail` — każdy job działa w trybie strict mode z automatycznym exportem zmiennych.
2. Parsuje zmienną `CI_CONFIG_PATH` i eksportuje trzy zmienne opisujące repozytorium CI.

## Parsowanie `CI_CONFIG_PATH`

GitLab wypełnia `CI_CONFIG_PATH` w formacie:

```
<plik>@<projekt>:<gałąź>
```

Przykład:

```
pipelines/image-builder/.gitlab-ci.yml@dev.rachuna/flows/gitlab:main
```

| Zmienna eksportowana | Wyrażenie bash | Wartość dla przykładu |
|---|---|---|
| `GITLAB_CI_REPOSITORY_FILEPATH` | `${CI_CONFIG_PATH%%@*}` | `pipelines/image-builder/.gitlab-ci.yml` |
| `GITLAB_CI_REPOSITORY_PATH` | `${CI_CONFIG_PATH#*@}` → część przed `:` | `dev.rachuna/flows/gitlab` |
| `GITLAB_CI_REPOSITORY_BRANCH` | `${CI_CONFIG_PATH#*@}` → część po `:` | `main` |

## Zmienne wyjściowe

| Zmienna | Opis |
|---|---|
| `GITLAB_CI_REPOSITORY_FILEPATH` | Ścieżka do pliku `.gitlab-ci.yml` w repozytorium CI |
| `GITLAB_CI_REPOSITORY_PATH` | Ścieżka projektu repozytorium CI (używana do budowania URL-i) |
| `GITLAB_CI_REPOSITORY_BRANCH` | Gałąź repozytorium CI (używana przez `job-docs` do URL-a dokumentacji) |

## Użycie w jobach

Helper jest wstrzykiwany jako pierwszy element `before_script` i `after_script` każdego joba:

```yaml
before_script:
  - !reference [.common.job-prepare.sh]   # ← musi być pierwszy
  - !reference [.common.logger.script.sh]  - !reference [.common.logo.script.sh]
  - !reference [.common.gitlab-tools.sh]
  - !reference [.common.mój-job.before_script.sh]

after_script:
  - !reference [.common.job-prepare.sh]   # ← musi być pierwszy (after_script to nowy proces)
  - !reference [.common.logger.script.sh]  - !reference [.common.job-docs.script.sh]
```

## Uwagi

- `after_script` w GitLab CI uruchamia się w osobnym procesie — zmienne wyeksportowane w `before_script` nie są dostępne. Dlatego `job-prepare` jest wstrzykiwany zarówno w `before_script`, jak i `after_script`.
- `set -a` (część `set -eaou pipefail`) powoduje, że wszystkie zmienne zdefiniowane po wywołaniu helpera są automatycznie eksportowane do środowiska.
