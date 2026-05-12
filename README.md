# MXUserBot Documentation

Documentation site: https://mxuserbot.github.io/documentation/

## Adding translations

Want to add a translation? Great!

The docs use MkDocs i18n plugin with folder-based structure. Each language has its own folder:
- `docs/ru/` — Russian (default)
- `docs/en/` — English

### How to add a new language

1. Copy `docs/ru/` folder to `docs/<your_locale>/`
2. Translate all `.md` files
3. Add the locale to `mkdocs.yml` in `plugins.i18n.languages` section

Example for German:
```yaml
plugins:
  - i18n:
      docs_structure: folder
      languages:
        - locale: ru
          default: true
          name: Русский
          build: true
        - locale: en
          name: English
          build: true
          nav_translations:
            ...
        - locale: de
          name: Deutsch
          build: true
          nav_translations:
            Главная: Home
            Преимущества: Advantages
            ...
```

4. Send a Pull Request!
