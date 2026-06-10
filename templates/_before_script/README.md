# Komponent: _before_script

Dostarcza ukryty job `.before_script`, który przygotowuje środowisko przed
wykonaniem sekcji `script` joba konsumenta.

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_before_script@1.0.0
    inputs:
      image: registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0
      before_script:
        - echo "Dodatkowe przygotowanie"

verify:
  extends:
    - .before_script
  script:
    - echo "Weryfikacja"
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## Co robi

Komponent wykonuje kolejne kroki przed skryptem joba:

1. Ładuje funkcje logowania z helpera `logger`.
2. Inicjalizuje środowisko i dane repozytorium przez `job-prepare`.
3. Wyświetla baner pipeline za pomocą helpera `logo`.
4. Ładuje funkcje GitLab i SSH z helpera `gitlab-tools`.
5. Wykonuje polecenia przekazane przez input `before_script`.
6. Wykonuje zawartość zmiennej `JOB_BEFORE_SCRIPT`, jeżeli jest ustawiona.

## Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz kontenera przypisywany do joba |
| `before_script` | array | `[]` | Dodatkowe polecenia wykonywane po załadowaniu helperów |

## Zmienne

| Zmienna | Wymagana | Opis |
|---|---|---|
| `CI_CONFIG_PATH` | tak | Źródło danych repozytorium przetwarzane przez `job-prepare` |
| `PIPELINE_TYPE` | tak | Typ pipeline wyświetlany przez helper `logo`, na przykład `CI` lub `CD` |
| `JOB_BEFORE_SCRIPT` | nie | Dodatkowy skrypt Bash wykonywany na końcu `before_script` |

Przykład przekazania skryptu przez zmienną:

```yaml
verify:
  extends:
    - .before_script
  variables:
    JOB_BEFORE_SCRIPT: |
      check_var "PACKAGE_NAME"
      echo "Przygotowanie $PACKAGE_NAME"
  script:
    - echo "Weryfikacja"
```

## Uwagi

- Input `before_script` przyjmuje tablicę poleceń GitLab CI/CD.
- `JOB_BEFORE_SCRIPT` jest wykonywany przez `eval` i powinien pochodzić
  wyłącznie z zaufanej konfiguracji.
- Job konsumenta może nadpisać pole `image`; w przeciwnym razie używany jest
  obraz przekazany do komponentu.
