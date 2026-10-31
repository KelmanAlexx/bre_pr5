# Команды Git по этапам Git Flow

Команды ниже показывают типовой порядок действий. Для трёх feature-команд блок создания ветки выполняется каждым разработчиком отдельно в своей рабочей копии репозитория.

## 1. Создание интеграционной ветки `develop`

```bash
git clone https://github.com/KelmanAlexx/pre_pr6.git
cd ais-integration-gateway
git switch main
git pull --ff-only origin main
git switch -c develop
git push -u origin develop
```

## 2. Три параллельные feature-ветки

```bash
# Платежная команда
git switch develop
git pull --ff-only origin develop
git switch -c feature/payments
# работа над платежным модулем
git add .
git commit -m "Add payment status handling"
git push -u origin feature/payments
```

```bash
# Команда складского учета
git switch develop
git pull --ff-only origin develop
git switch -c feature/inventory
# работа над остатками и резервированием
git add .
git commit -m "Add inventory synchronization"
git push -u origin feature/inventory
```

```bash
# Команда каталога
git switch develop
git pull --ff-only origin develop
git switch -c feature/catalog
# работа над импортом карточек товаров
git add .
git commit -m "Add catalog import validation"
git push -u origin feature/catalog
```

После отправки ветки каждая команда создаёт Pull Request: `feature/...` → `develop`. В GitHub проверяются изменения, тесты и результаты SAST. После одобрения выполняется Merge Pull Request. Вариант с командной строкой для одной ветки:

```bash
git switch develop
git pull --ff-only origin develop
git merge --no-ff feature/payments -m "Merge feature/payments"
git push origin develop
git branch -d feature/payments
git push origin --delete feature/payments
```

Для `feature/inventory` и `feature/catalog` действия повторяются после проверки их PR. Если в `develop` появились новые коммиты, до завершения PR разработчик может обновить свою feature-ветку:

```bash
git fetch origin
git switch feature/catalog
git merge origin/develop
# разрешить конфликты, если они появились
git add .
git commit -m "Resolve develop merge conflicts"
git push
```

## 3. Недельный релиз `release/v1.12.0`

```bash
git switch develop
git pull --ff-only origin develop
git switch -c release/v1.12.0
git push -u origin release/v1.12.0
```

Во время стабилизации изменения вносятся только для исправления ошибок и подготовки выпуска. После проверки версия переносится в `main`, получает тег и возвращается в `develop`:

```bash
# Публикация проверенной версии
git switch main
git pull --ff-only origin main
git merge --no-ff release/v1.12.0 -m "Release v1.12.0"
git tag -a v1.12.0 -m "Release version 1.12.0"
git push origin main --follow-tags

# Возврат стабилизационных исправлений в develop
git switch develop
git pull --ff-only origin develop
git merge --no-ff release/v1.12.0 -m "Merge release/v1.12.0 back to develop"
git push origin develop

# Удаление завершённой ветки
git branch -d release/v1.12.0
git push origin --delete release/v1.12.0
```

## 4. Экстренное исправление `hotfix/v1.12.1`

Пример: в production найдена критическая ошибка обработки повторного уведомления платежного провайдера.

```bash
git switch main
git pull --ff-only origin main
git switch -c hotfix/v1.12.1
# исправить ошибку и добавить регрессионный тест
git add .
git commit -m "Fix duplicate payment notification"
git push -u origin hotfix/v1.12.1
```

После code review, SAST и срочных регрессионных тестов:

```bash
# Исправление в стабильной ветке
git switch main
git merge --no-ff hotfix/v1.12.1 -m "Hotfix v1.12.1"
git tag -a v1.12.1 -m "Hotfix version 1.12.1"
git push origin main --follow-tags

# Обязательный перенос исправления в develop
git switch develop
git merge --no-ff hotfix/v1.12.1 -m "Merge hotfix into develop"
git push origin develop

# Если уже существует активный релиз, исправление переносится и туда:
# git switch release/v1.13.0
# git merge --no-ff hotfix/v1.12.1
# git push origin release/v1.13.0
```

В конце hotfix-ветку удаляют после слияния во все необходимые ветки.
