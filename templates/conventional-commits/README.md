# Komponent: conventional-commits

Waliduje tytuły commitów na bieżącej gałęzi zgodnie ze standardem
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_before_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_after_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/conventional-commits@1.0.0
    inputs:
      job-stage: validate
      job-before-script: |
        echo "Przygotowanie walidacji"
      job-after-script: |
        echo "Zakończenie walidacji"
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## Co robi

1. Sprawdza, czy zmienna `CONVENTIONAL_COMMITS_REGEXP` jest ustawiona.
2. Wykonuje skrypt przekazany przez input `job-before-script`.
3. Pobiera domyślną gałąź z repozytorium `origin`.
4. Pobiera tytuły commitów między `origin/$CI_DEFAULT_BRANCH` a `HEAD`.
5. Pomija commity zaczynające się od `Merge`, `Revert` lub `Initial commit`.
6. Waliduje pozostałe tytuły wyrażeniem `CONVENTIONAL_COMMITS_REGEXP`.
7. Kończy job błędem, jeżeli co najmniej jeden tytuł jest niezgodny.
8. Wykonuje skrypt przekazany przez input `job-after-script`.

Analizowana lista commitów odpowiada wynikowi polecenia:

```bash
git --no-pager log origin/"$CI_DEFAULT_BRANCH"..HEAD --pretty=format:"%s"
```

Walidacja jest pomijana na gałęzi domyślnej oraz gdy nie znaleziono commitów
do analizy.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `job-name` | string | `conventional-commits` | Nazwa tworzonego joba |
| `job-stage` | string | `validate` | Etap pipeline, na którym zostanie uruchomiony job |
| `job-image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz kontenera joba |
| `job-rules` | array | wyłączenie dla `ENABLED_CONVENTIONAL_COMMITS=false`, w pozostałych przypadkach `on_success` | Reguły dodawania joba do pipeline |
| `job-needs` | array | `[]` | Lista zależności przekazywana do `needs` |
| `job-resource-group` | string | `${CI_PIPELINE_ID}-${CI_JOB_NAME}` | Grupa zasobów ograniczająca równoległe wykonanie joba |
| `job-before-script` | string | `:` | Dodatkowy skrypt wykonywany przed walidacją |
| `job-after-script` | string | `:` | Dodatkowy skrypt wykonywany po walidacji |

Dostępne wartości `job-stage`: `.pre`, `prepare`, `validate`, `dependency`,
`build`, `deployment`, `tests`, `publish` i `.post`.

## Reguły

Domyślnie job nie jest dodawany do pipeline, gdy zmienna
`ENABLED_CONVENTIONAL_COMMITS` ma wartość `false`. W pozostałych przypadkach
jest uruchamiany z `when: on_success`.

Domyślne reguły można całkowicie zastąpić przez input `job-rules`:

```yaml
inputs:
  job-rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - when: never
```

Jeżeli komponent jest jedynym jobem w konfiguracji, reguła wykluczająca ten
job może spowodować błąd GitLab: `The resulting pipeline would have been empty`.

## Zmienne

| Zmienna | Pochodzenie | Opis |
|---|---|---|
| `CONVENTIONAL_COMMITS_REGEXP` | `variables/main.yml` | Wyrażenie regularne używane do walidacji tytułów commitów |
| `DOCS_MD_FILE_PATH` | `variables/main.yml` | Ścieżka do dokumentacji komponentu |
| `GITLAB_CI_COMPONENTS_PATH` | `variables/main.yml` | Ścieżka repozytorium komponentów używana do budowania URL-a dokumentacji |
| `ENABLED_CONVENTIONAL_COMMITS` | CI/CD Variables projektu | Ustawienie `false` wyłącza job przy domyślnych regułach |

## Przykłady commitów

Poprawne:

```text
feat(api): Dodanie endpointu
feat: Dodanie funkcjonalności
fix!: Zmiana niekompatybilna wstecznie
```

Niepoprawne:

```text
Dodanie funkcjonalności
feat : Dodanie funkcjonalności
Feat: Dodanie funkcjonalności
feature: Dodanie funkcjonalności
```

## Naprawa błędu

Dla ostatniego commita popraw tytuł poleceniem `git commit --amend`. Dla wielu
commitów użyj `git rebase -i`, a następnie wypchnij zmiany przez
`git push --force-with-lease`.
