# Komponent: _after_script

Dostarcza ukryty job `.after_script`, który wyświetla wynik i dokumentację
po zakończeniu sekcji `script` joba konsumenta.

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_after_script@1.0.0
    inputs:
      image: registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0
      after_script:
        - echo "Dodatkowe zakończenie"

verify:
  extends:
    - .after_script
  variables:
    DOCS_MD_FILE_PATH: docs/verify/README.md
  script:
    - echo "Weryfikacja"
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## Co robi

Komponent wykonuje kolejne kroki po skrypcie joba:

1. Ładuje funkcje logowania z helpera `logger`.
2. Ponownie inicjalizuje środowisko przez `job-prepare`, ponieważ GitLab
   uruchamia `after_script` w osobnym procesie.
3. Wyświetla status joba i link do dokumentacji przez `job-docs`.
4. Wykonuje polecenia przekazane przez input `after_script`.
5. Wykonuje zawartość zmiennej `JOB_AFTER_SCRIPT`, jeżeli jest ustawiona.

Sekcja `after_script` jest wykonywana również wtedy, gdy główny skrypt joba
zakończy się błędem.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz kontenera przypisywany do joba |
| `after_script` | array | `[]` | Dodatkowe polecenia wykonywane po helperze `job-docs` |

## Zmienne

| Zmienna | Wymagana | Opis |
|---|---|---|
| `CI_CONFIG_PATH` | tak | Źródło danych repozytorium przetwarzane przez `job-prepare` |
| `DOCS_MD_FILE_PATH` | tak | Ścieżka do pliku dokumentacji joba |
| `GITLAB_CI_COMPONENTS_PATH` | nie | Ścieżka repozytorium komponentów używana w URL-u dokumentacji |
| `PROJECT_TYPE` | nie | Dla `gitlab-pipelines` włącza sprawdzenie istnienia pliku dokumentacji |
| `JOB_AFTER_SCRIPT` | nie | Dodatkowy skrypt Bash wykonywany na końcu `after_script` |

Gdy `GITLAB_CI_COMPONENTS_PATH` nie jest ustawiona, URL dokumentacji używa
ścieżki `GITLAB_CI_REPOSITORY_PATH` wyznaczonej przez `job-prepare`.

Przykład przekazania skryptu przez zmienną:

```yaml
verify:
  extends:
    - .after_script
  variables:
    DOCS_MD_FILE_PATH: docs/verify/README.md
    JOB_AFTER_SCRIPT: |
      echo "Zakończono job $CI_JOB_NAME"
  script:
    - echo "Weryfikacja"
```

## Uwagi

- Input `after_script` przyjmuje tablicę poleceń GitLab CI/CD.
- `JOB_AFTER_SCRIPT` jest wykonywany przez `eval` i powinien pochodzić
  wyłącznie z zaufanej konfiguracji.
- `after_script` działa w osobnym procesie, dlatego zmiany środowiska wykonane
  wcześniej w `before_script` nie są automatycznie dostępne.
