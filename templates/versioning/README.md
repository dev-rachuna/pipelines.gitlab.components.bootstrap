# Komponent: versioning

Wyznacza wersję projektu na podstawie
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) i
uruchamia [Semantic Release](https://semantic-release.gitbook.io/semantic-release/).

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/versioning@1.0.0
    inputs:
      job-stage: .pre
      job-dry-run: true
      job-before-script: |
        echo "Przygotowanie wersjonowania"
      job-after-script: |
        echo "Zakończenie wersjonowania"
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## Co robi

1. Sprawdza wymagane zmienne konfiguracyjne i token GitLab.
2. Wykonuje skrypt przekazany przez input `job-before-script`.
3. Używa pliku `.releaserc.cjs` dostarczonego przez projekt albo pobiera
   domyślną konfigurację z repozytorium komponentów.
4. Odczytuje najnowszy tag Git i zapisuje wersję kandydującą do
   `versioning.env`. Gdy repozytorium nie ma tagów, używa `0.0.1-dev`.
5. Uruchamia `semantic-release`, opcjonalnie z parametrem `--dry-run`.
6. Dla wydania typu `release` publikuje zmieniony `CHANGELOG.md` w bieżącej
   gałęzi, o ile nie jest aktywny tryb dry-run.
7. Wykonuje skrypt przekazany przez input `job-after-script`.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `job-name` | string | `versioning` | Nazwa tworzonego joba |
| `job-stage` | string | `.pre` | Etap pipeline, na którym zostanie uruchomiony job |
| `job-image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz kontenera joba |
| `job-rules` | array | wyłączenie dla `ENABLED_YAMLLINT=false`, w pozostałych przypadkach `on_success` | Reguły dodawania joba do pipeline |
| `job-needs` | array | `[]` | Lista zależności przekazywana do `needs` |
| `job-resource-group` | string | `${CI_PIPELINE_ID}-${CI_JOB_NAME}` | Grupa zasobów ograniczająca równoległe wykonanie joba |
| `job-before-script` | string | `:` | Dodatkowy skrypt wykonywany przed wersjonowaniem |
| `job-after-script` | string | `:` | Dodatkowy skrypt wykonywany po wersjonowaniu |
| `job-dry-run` | boolean | `false` | Włącza symulację Semantic Release bez publikowania wydania i tagów |

Dostępne wartości `job-stage`: `.pre`, `prepare`, `validate`, `dependency`,
`build`, `deployment`, `tests`, `publish` i `.post`.

## Wymagane zmienne

| Zmienna | Pochodzenie | Opis |
|---|---|---|
| `GITLAB_TOKEN` | CI/CD Variables projektu | Token używany do pobrania konfiguracji i publikacji wydania |
| `CI_API_V4_URL` | GitLab CI | Bazowy URL API GitLab |
| `CI_COMMIT_BRANCH` | GitLab CI | Gałąź docelowa dla publikacji `CHANGELOG.md` |
| `GITLAB_USER_EMAIL` | CI/CD Variables projektu | Adres e-mail autora commita aktualizującego changelog |
| `GITLAB_USER_NAME` | CI/CD Variables projektu | Nazwa autora commita aktualizującego changelog |

Komponent ustawia także:

| Zmienna | Wartość | Opis |
|---|---|---|
| `RELEASERC_PATH` | `templates/versioning/.releaserc.cjs` | Ścieżka domyślnej konfiguracji Semantic Release |
| `VERSIONING_DRY_RUN` | wartość `job-dry-run` | Steruje uruchomieniem z `--dry-run` |
| `NODE_EXTRA_CA_CERTS` | `/etc/ssl/certs/ca-certificates.crt` | Certyfikaty CA używane przez Node.js |
| `DOCS_MD_FILE_PATH` | ustawiona przez komponent | Ścieżka dokumentacji wyświetlanej przez `job-docs` |

## Plik versioning.env

Przed uruchomieniem Semantic Release komponent tworzy plik:

```dotenv
RELEASE_CANDIDATE_VERSION=1.2.3
```

Domyślna konfiguracja `.releaserc.cjs` może następnie nadpisać ten plik i
dodać:

| Zmienna | Przykład | Opis |
|---|---|---|
| `RELEASE_CANDIDATE_VERSION` | `1.2.3` | Wersja wyznaczona przez Semantic Release |
| `RELEASE_CANDIDATE_TAG` | `v1.2.3` | Tag wersji z prefiksem `v` |
| `ARTIFACTS_TYPE` | `release` lub `snapshot` | Typ artefaktów zależny od gałęzi |
| `JIRA_ISSUES_IDS` | `"PROJ-1, PROJ-2"` | Unikalne identyfikatory JIRA znalezione w commitach |

Aktualny komponent nie deklaruje `versioning.env` jako raportu `dotenv` ani
artefaktu. Plik pozostaje w katalogu roboczym joba.

## Reguły wersjonowania

Domyślna konfiguracja obsługuje następujące gałęzie:

| Gałąź | Kanał | Typ wydania |
|---|---|---|
| `main` | domyślny | stabilne wydanie |
| `develop` | `dev` | prerelease |
| `alpha` | `alpha` | prerelease |
| `release` | `rc` | release candidate |
| `hotfix` | `ht` | prerelease hotfix |

Wpływ typu commita na wersję:

| Commit | Zmiana wersji |
|---|---|
| `BREAKING CHANGE` lub `!` | major |
| `feat` | minor |
| `fix` | patch |

## CHANGELOG.md

Semantic Release generuje lub aktualizuje `CHANGELOG.md`. Komponent tworzy
commit:

```text
docs: update changelog [skip ci]
```

i wysyła go do `CI_COMMIT_BRANCH` tylko wtedy, gdy:

- plik `CHANGELOG.md` istnieje,
- `ARTIFACTS_TYPE` ma wartość `release`,
- `VERSIONING_DRY_RUN` nie ma wartości `true`.

## Własna konfiguracja

Projekt może dostarczyć w katalogu głównym własny plik `.releaserc.cjs`.
W takim przypadku komponent nie pobiera konfiguracji domyślnej.

## Lokalne uruchomienie

```bash
VERSIONING_DRY_RUN=true semantic-release --dry-run
```
