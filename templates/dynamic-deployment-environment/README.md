# Komponent: dynamic-deployment-environment

Pobiera aktywne środowiska projektu z API GitLab, generuje plik
`deployment.yml` zawierający osobny job wdrożeniowy dla każdego środowiska,
a następnie uruchamia go jako child pipeline.

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_before_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_after_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/dynamic-deployment-environment@1.0.0
    inputs:
      job-stage: prepare
      job-definitons-script: |
        "Deploy:$ENV_NAME":
          stage: deployment
          extends: [.deployment]
          environment:
            name: $ENV_NAME
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## Co robi

1. Pobiera z API GitLab listę środowisk projektu.
2. Pomija środowiska w stanie `stopped`.
3. Gdy nie istnieją aktywne środowiska, generuje informacyjny job zastępczy.
4. Tworzy plik `deployment.yml` z sekcją `include` wskazującą główną
   konfigurację repozytorium komponentów.
5. Dla każdego środowiska dopisuje definicję przekazaną w parametrze
   `job-definitons-script`.
6. Publikuje `deployment.yml` jako artefakt i uruchamia go w child pipeline.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `job-name` | string | `dynamic-deployment-environment` | Nazwa tworzonego joba |
| `trigger-job-name` | string | `trigger-deployment` | Nazwa joba uruchamiającego child pipeline |
| `job-stage` | string | `.pre` | Etap pipeline, na którym zostanie uruchomiony job |
| `trigger-job-stage` | string | `deployment` | Etap pipeline joba uruchamiającego child pipeline |
| `job-image` | string | `registry.gitlab.com/pl.rachuna-net/artifacts/containers/python:v1.2.0` | Obraz kontenera joba |
| `job-rules` | array | uruchomienie dla pipeline CD, z wyłączeniem child pipeline | Reguły dodawania joba do pipeline |
| `trigger-job-rules` | array | uruchomienie dla pipeline CD, z wyłączeniem child pipeline | Reguły dodawania joba wyzwalającego do pipeline |
| `job-needs` | array | `[]` | Lista zależności przekazywana do `needs` |
| `job-resource-group` | string | `${CI_PIPELINE_ID}-${CI_JOB_NAME}` | Grupa zasobów ograniczająca równoległe wykonanie joba |
| `trigger-job-resource-group` | string | `${CI_PIPELINE_ID}-${CI_JOB_NAME}` | Grupa zasobów joba uruchamiającego child pipeline |
| `job-before-script` | string | `:` | Dodatkowy skrypt wykonywany przed pobraniem środowisk |
| `job-after-script` | string | `:` | Dodatkowy skrypt wykonywany po wygenerowaniu konfiguracji |
| `job-definitons-script` | string | job `💥  Deployment:$ENV_NAME` rozszerzający `.deployment` | Definicja joba dopisywana dla każdego środowiska |

Dostępne wartości `job-stage` i `trigger-job-stage`: `.pre`, `prepare`,
`dependency`, `validate`, `build`, `deployment`, `tests`, `publish` i `.post`.

Nazwa inputu `job-definitons-script` jest zachowana ze względu na zgodność
interfejsu komponentu.

## Definicja joba

W parametrze `job-definitons-script` dostępna jest zmienna `ENV_NAME`. Dla każdej
iteracji zawiera nazwę aktualnie przetwarzanego środowiska i jest rozwijana
podczas generowania pliku `deployment.yml`.

Domyślna definicja:

```yaml
💥  Deployment:<ENV_NAME>:
  stage: deployment
  extends: [.deployment]
  environment:
    name: <ENV_NAME>
```

Projekt używający komponentu musi udostępnić konfigurację `.deployment` albo
przekazać własną definicję przez `job-definitons-script`.

## Reguły

Domyślnie oba joby komponentu są uruchamiane tylko wtedy, gdy `PIPELINE_TYPE`
ma wartość `CD`, a źródłem pipeline nie jest `parent_pipeline`. Dla pipeline
typu `CI` oraz wszystkich pozostałych przypadków joby nie są dodawane.

Domyślne reguły można całkowicie zastąpić przez inputy `job-rules` oraz
`trigger-job-rules`.

## Zmienne

| Zmienna | Pochodzenie | Opis |
|---|---|---|
| `CI_PROJECT_ID` | GitLab CI | Identyfikator projektu używany w zapytaniu o środowiska |
| `PIPELINE_TYPE` | CI/CD Variables projektu | Określa typ pipeline używany przez domyślne reguły |
| `GITLAB_CI_REPOSITORY_PATH` | zmienne wspólne pipeline | Projekt wskazywany w sekcji `include` generowanego pliku |
| `GITLAB_CI_REPOSITORY_BRANCH` | zmienne wspólne pipeline | Referencja wskazywana w sekcji `include` generowanego pliku |
| `CONVENTIONAL_COMMITS_REGEXP` | komponent | Zmienna sprawdzana przed uruchomieniem generowania |
| `DOCS_MD_FILE_PATH` | komponent | Ścieżka do dokumentacji komponentu |

Obraz joba musi udostępniać polecenia `glab` i `jq`. Dostęp do API GitLab jest
wykonywany przez poświadczenia dostępne dla `glab` w jobie CI.

## Wynik

Komponent zapisuje wygenerowaną konfigurację w pliku `deployment.yml`,
publikuje ją jako artefakt na tydzień i przekazuje do joba
`trigger-deployment`. Job wyzwalający uruchamia konfigurację jako child
pipeline ze strategią `mirror`, dlatego jego wynik odzwierciedla wynik child
pipeline.
