# Дерево представления веток

```text
branches/
├── main/                    # main, стабильный релиз v1.12.0
├── develop/                 # develop, интеграционная ветка
├── feature/                 # группа feature-веток
│   ├── payments/            # feature/payments
│   ├── inventory/           # feature/inventory
│   └── catalog/             # feature/catalog
├── release/                 # группа release-веток
│   └── v1.12.0/             # release/v1.12.0
└── hotfix/                  # группа hotfix-веток
    └── v1.12.1/             # hotfix/v1.12.1
```

Каждая конечная папка содержит `README.md` с описанием ветки. Внутренние каталоги показывают, какая часть приложения меняется в соответствующей ветке.