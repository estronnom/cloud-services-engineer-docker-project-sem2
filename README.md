# Momo Store: Docker-проект
## Подготовка конфигурации

```bash
cp .env.be.example .env.be
cp .env.fe.example .env.fe
cp .env.common.example .env.common
cp secrets/api-keys.json.example secrets/api-keys.json
```

### Переменные окружения

| Переменная | Сервис и назначение | Значение по умолчанию/пример |
|---|---|---|
| `TAG` | Compose, тег для собираемых образов | `latest` |
| `FRONTEND_PORT` | Compose, порт запуска фронтенд сервиса | `80` |
| `BACKEND_PORT` | Compose, порт запуска load-balancer для бекенда | `8081` |
| `VUE_APP_API_URL` | base backend url для frontend, передается в качестве билд-аргумента контейнеру фронтенда | `http://localhost:8081` |
| `API_KEYS_FILE` | backend, пример секрета | `/run/secrets/api-keys` |
| `BACKEND_CONFIGURATION_KEY_1` | backend, пример переменной окружения | `foo` |
| `BACKEND_CONFIGURATION_KEY_2` | backend, пример переменной окружения | `bar` |
| `FRONTEND_CONFIGURATION_KEY_1` | frontend, пример переменной окружения | `foo` |
| `FRONTEND_CONFIGURATION_KEY_2` | frontend, пример переменной окружения | `bar` |
| `COMMON_KEY_1` | все, пример переменной окружения | `foo` |

## Сборка и запуск

Запуск:

```bash
docker compose up -d --build
```

Запуск и сборка с горизонатальным масштабированием бекенда в 3 инстанса (в качестве балансировщика нагрузки используется nginx):

```bash
docker compose up -d --build --scale backend=3
```

Остановка:

```bash
docker compose down
```

Доступ к приложению после запуска: `http://localhost/momo-store/`.

## Оптимизация образов

Использованы multi-stage сборки:

- backend: Go builder → `alpine` runtime;
- frontend: Node builder → `nginx-unprivileged` runtime.

В runtime-образах отсутствуют исходники и зависимости разработки. Оба образа multi-stage. Для снижения размера образов использованы легковесные alpine-based образы, очищен apk cache.
Оба образа запускаются не от имени root юзера.

Для большей уверенности в безопасности рантайм-образов и снижении возможной плоскости атаки для бекенда следовало бы использовать distroless образ, однако в таком случае не сработал бы healthcheck через curl. Решение: использовать собственный binary для хелсчека.

Размеры итоговых образов после сборки:

```text
backend:
"Size": 38032812 (38Mb)

frontend:
"Size": 62287828 (62Mb)
```

## Безопасность

- Контейнеры запускаются от непривилегированных пользователей.
- В runtime-образах удалены/не устанавливаются dev-зависимости.
- Чувствительные данные не включаются в образы и передаются через Compose Secret
  `api-keys`.
- Открытые порты: 80 для фронтенда, 8081 для бекенда
- Настроены `cap_drop`, `no-new-privileges`, `read_only`/`tmpfs`, лимиты CPU и памяти.

## Сканирование образов

Для сканирования используется Trivy в GitHub Actions. Результат последнего
запуска: https://github.com/estronnom/cloud-services-engineer-docker-project-sem2/actions/runs/34886459131.

Локальная команда:

```bash
trivy image --severity HIGH,CRITICAL --ignore-unfixed [image:tag]
```

## CI/CD

1. Запуск и сборка приложений с помощью docker/build-push-action@v4, публикация в Docker Hub
2. Запуск и сборка приложений с помощью compose файла
3. Matrix проверка образов бекенда и фронтенда на уязвимости

## Dev compose.yml

Для примера использования dev-конфигурации compose создан файл compose.dev.yml.

Запуск в дев конфигурации

```bash
docker compose -f compose.yml -f compose.dev.yml up -d --build
```
