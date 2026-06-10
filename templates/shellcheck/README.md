# Komponent: shellcheck

Waliduje skrypty Bash w plikach `*.sh` oraz regularnych plikach z ustawionym
bitem wykonywalności za pomocą narzędzia ShellCheck.

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_before_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_after_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/shellcheck@1.0.0
    inputs:
      job-stage: validate
      job-before-script: |
        echo "Przygotowanie plików do analizy"
```

## Co robi

1. Wykonuje skrypt przekazany przez input `job-before-script`.
2. Wyszukuje rekurencyjnie pliki `*.sh` oraz pliki wykonywalne.
3. Pomija zawartość katalogu `.git`.
4. Uruchamia `shellcheck --shell=bash` dla każdego znalezionego pliku.
5. Kończy job po pierwszym błędzie ShellCheck.
6. Wykonuje skrypt przekazany przez input `job-after-script`.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `job-name` | string | `shellcheck` | Nazwa tworzonego joba |
| `job-stage` | string | `validate` | Etap pipeline, na którym zostanie uruchomiony job |
| `job-image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz kontenera joba |
| `job-rules` | array | wyłączenie dla `ENABLED_SHELLCHECK=false`, w pozostałych przypadkach `on_success` | Reguły dodawania joba do pipeline |
| `job-needs` | array | `[]` | Lista zależności przekazywana do `needs` |
| `job-resource-group` | string | `${CI_PIPELINE_ID}-${CI_JOB_NAME}` | Grupa zasobów ograniczająca równoległe wykonanie joba |
| `job-before-script` | string | `:` | Dodatkowy skrypt wykonywany przed walidacją |
| `job-after-script` | string | `:` | Dodatkowy skrypt wykonywany po walidacji |

## Reguły

Domyślnie job nie jest dodawany do pipeline, gdy zmienna
`ENABLED_SHELLCHECK` ma wartość `false`. W pozostałych przypadkach jest
uruchamiany z `when: on_success`. Domyślne reguły można zastąpić przez input
`job-rules`.

## Logika wyszukiwania

```bash
find ./ -path './.git' -prune -o -type f \
  \( -name "*.sh" -o -perm /111 \) -print0
```

Plik pasujący jednocześnie do obu warunków jest analizowany tylko raz.

## Zmienne

| Zmienna | Pochodzenie | Opis |
|---|---|---|
| `DOCS_MD_FILE_PATH` | komponent | Ścieżka do dokumentacji komponentu |
| `GITLAB_CI_COMPONENTS_PATH` | komponent | Ścieżka repozytorium komponentów używana do budowania URL-a dokumentacji |
| `ENABLED_SHELLCHECK` | CI/CD Variables projektu | Ustawienie `false` wyłącza job przy domyślnych regułach |

## Lokalne uruchomienie

```bash
while IFS= read -r -d '' file_sh; do
  shellcheck --shell=bash "$file_sh"
done < <(find ./ -path './.git' -prune -o -type f \
  \( -name "*.sh" -o -perm /111 \) -print0)
```
