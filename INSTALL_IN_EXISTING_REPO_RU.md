# Установка ТЗ в существующий репозиторий MOSAIC

Содержимое этого архива нужно поместить **в корень уже клонированного репозитория** `mosaic`, рядом с `LICENSE` и папкой `.git`.

После копирования ожидаемая структура:

```text
mosaic/
├── .git/
├── LICENSE
├── AGENTS.md
├── README.md
├── INSTALL_IN_EXISTING_REPO_RU.md
├── docs/
└── tasks/
```

Не создавайте дополнительную папку `project` или `mosaic-repo-overlay` внутри репозитория.

Проверьте изменения:

```sh
git status
git add .
git commit -m "docs: add MOSAIC specification and Codex tasks"
git push origin main
```

Затем откройте в Codex корень репозитория и первым заданием используйте содержимое `tasks/MASTER_START_PROMPT.md`.
