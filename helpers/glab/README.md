# Helper: glab

Konfiguruje uwierzytelnienie narzędzia `glab` do GitLab.com przed wykonaniem
skryptu joba.

## Co robi

Helper wykonuje kolejno:

1. Wyświetla w logu nagłówek `glab/script.sh.yml` za pomocą
   funkcji `h3`.
2. Loguje `glab` do hosta `gitlab.com`, używając tokenu ze zmiennej
   `GITLAB_TOKEN`.
3. Ustawia protokół Git dla hosta `gitlab.com` na HTTPS.

Odpowiadają temu polecenia:

```bash
glab auth login --hostname gitlab.com --token "${GITLAB_TOKEN}"
glab config set git_protocol https --host gitlab.com
```

Po wykonaniu helpera kolejne polecenia `glab` w jobie mogą korzystać z
uwierzytelnionej sesji bez ponownego przekazywania tokenu.

## Wymagania

| Wymaganie | Opis |
|---|---|
| `glab` | GitLab CLI musi być dostępne w obrazie joba i znajdować się w `PATH` |
| `GITLAB_TOKEN` | Token dostępu używany do logowania w GitLab.com |
| `h3` | Funkcja logująca dostarczana przez helper `logger` |

Zakres uprawnień tokenu powinien odpowiadać operacjom `glab` wykonywanym przez
joba. Dla operacji korzystających z GitLab API zazwyczaj wymagany jest zakres
`api`. Zmienną `GITLAB_TOKEN` należy przechowywać jako maskowaną zmienną CI/CD,
a dla chronionych gałęzi i tagów również jako zmienną chronioną.

## Użycie w jobach

Helper jest ładowany automatycznie przez komponent `_before_script`, po
helperach `logger`, `job-prepare`, `logo` i `gitlab-tools`:

```yaml
before_script:
  - !reference [.helpers.logger.script.sh]
  - !reference [.helpers.job-prepare.script.sh]
  - !reference [.helpers.logo.script.sh]
  - !reference [.helpers.gitlab-tools.script.sh]
  - !reference [.helpers.glab.script.sh]
```

Po jego załadowaniu skrypt joba może bezpośrednio wywoływać GitLab CLI, na
przykład:

```yaml
script:
  - glab auth status --hostname gitlab.com
  - glab release list
```

## Uwagi

- Host `gitlab.com` jest wpisany w helperze na stałe. Helper nie konfiguruje
  uwierzytelnienia do prywatnych instancji GitLab.
- Protokół operacji Git wykonywanych przez `glab` jest ustawiany na HTTPS.
- Brak `GITLAB_TOKEN` albo programu `glab` powoduje błąd podczas
  `before_script` i zatrzymuje job działający w trybie strict mode.
