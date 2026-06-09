# bootstrap

Komponent GitLab CI/CD udostępniający bazową konfigurację joba przez ukryty
szablon `.bootstrap`. Projekt jest przygotowany do publikacji w GitLab CI/CD
Catalog.

## Components

### `bootstrap`

Komponent dodaje ukryty job, który można dziedziczyć za pomocą `extends`.
Domyślnie ustawia obraz wykonawczy i `before_script`, ale nie uruchamia żadnego
dodatkowego joba w pipeline konsumenta.

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/bootstrap@1.0.0
    inputs:
      image: alpine:3.22

verify:
  extends: .bootstrap
  stage: test
  script:
    - bootstrap_info
    - echo "Tutaj uruchom własne polecenia"
```

Pełny przykład znajduje się w [`examples/consumer.gitlab-ci.yml`](examples/consumer.gitlab-ci.yml).
Opis wejść komponentu jest przechowywany bezpośrednio w
[`templates/bootstrap/template.yml`](templates/bootstrap/template.yml) i jest
wyświetlany automatycznie w GitLab CI/CD Catalog.

## Development

Pipeline projektu:

1. Dołącza komponent z aktualnego commita.
2. Uruchamia job testowy dziedziczący po `.bootstrap`.
3. Dla tagu zgodnego z SemVer tworzy GitLab Release.

Do testowania zmian przed wydaniem używaj gałęzi lub SHA commita. Konsumenci
produkcyjni powinni wskazywać tag wydania, na przykład `1.0.0`.

## Release

1. Połącz zmiany z gałęzią domyślną.
2. Utwórz tag SemVer, na przykład `1.0.0`.
3. Pipeline zweryfikuje komponent i utworzy release.

Projekt musi mieć włączoną opcję **CI/CD Catalog project** w ustawieniach
GitLab, aby wydania były publikowane w katalogu.

## Contributing

Zmiany powinny zachowywać kompatybilność wejść komponentu. Zmiana niezgodna
wstecznie wymaga nowej wersji major zgodnie z Semantic Versioning.
