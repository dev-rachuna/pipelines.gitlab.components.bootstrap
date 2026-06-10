# Komponent: yamllint

Waliduje składnię i formatowanie plików `.yml` oraz `.yaml` za pomocą
narzędzia [yamllint](https://yamllint.readthedocs.io/).

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_before_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_after_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/yamllint@1.0.0
    inputs:
      job-stage: validate
      job-before-script: |
        echo "Przygotowanie walidacji YAML"
      job-after-script: |
        echo "Zakończenie walidacji YAML"
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## Co robi

1. Wykonuje skrypt przekazany przez input `job-before-script`.
2. Tworzy w katalogu roboczym plik `.yamllint.yml` z konfiguracją komponentu.
3. Uruchamia polecenie `yamllint .`, które rekurencyjnie sprawdza pliki YAML.
4. Kończy job błędem, jeżeli yamllint wykryje nieprawidłowości.
5. Wykonuje skrypt przekazany przez input `job-after-script`.

## Konfiguracja

Komponent tworzy następujący plik `.yamllint.yml`:

```yaml
---
extends: default

rules:
  line-length: disable
  comments-indentation: disable
```

Obowiązują domyślne reguły yamllint z wyłączoną kontrolą długości linii
oraz wcięć komentarzy. Istniejący plik `.yamllint.yml` w repozytorium jest
nadpisywany na czas wykonywania joba.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `job-name` | string | `yamllint` | Nazwa tworzonego joba |
| `job-stage` | string | `validate` | Etap pipeline, na którym zostanie uruchomiony job |
| `job-image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz kontenera joba |
| `job-rules` | array | wyłączenie dla `ENABLED_YAMLLINT=false`, w pozostałych przypadkach `on_success` | Reguły dodawania joba do pipeline |
| `job-needs` | array | `[]` | Lista zależności przekazywana do `needs` |
| `job-resource-group` | string | `${CI_PIPELINE_ID}-${CI_JOB_NAME}` | Grupa zasobów ograniczająca równoległe wykonanie joba |
| `job-before-script` | string | `:` | Dodatkowy skrypt wykonywany przed walidacją |
| `job-after-script` | string | `:` | Dodatkowy skrypt wykonywany po walidacji |

Dostępne wartości `job-stage`: `.pre`, `prepare`, `validate`, `dependency`,
`build`, `deployment`, `tests`, `publish` i `.post`.

## Reguły

Domyślnie job nie jest dodawany do pipeline, gdy zmienna `ENABLED_YAMLLINT`
ma wartość `false`. W pozostałych przypadkach jest uruchamiany z
`when: on_success`.

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
| `DOCS_MD_FILE_PATH` | komponent | Ścieżka do dokumentacji komponentu |
| `GITLAB_CI_COMPONENTS_PATH` | komponent | Ścieżka repozytorium komponentów używana do budowania URL-a dokumentacji |
| `ENABLED_YAMLLINT` | CI/CD Variables projektu | Ustawienie `false` wyłącza job przy domyślnych regułach |

## Przykładowe błędy

Niepoprawne wcięcie:

```yaml
services:
 api:
   image: node:20
```

Poprawna wersja:

```yaml
services:
  api:
    image: node:20
```

Niepoprawna wartość logiczna:

```yaml
enabled: yes
```

Poprawna wersja:

```yaml
enabled: true
```

## Lokalne uruchomienie

```bash
yamllint -c .yamllint.yml .
```
