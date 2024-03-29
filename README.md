# werf-debug

werf-debug.yaml — это конфигурационный файл для настройки окружения и развертывания веб-приложения deckhouse-web, которое создает готовые кластеры Kubernetes и управляет ими. В файле используются шаблонные конструкции Go Templates.

**Назначение файла:**

- Определение базовых Docker-образов.
- Конфигурация Docker-образов для статической веб-страницы, бэкенда и фронтенда.
- Установка зависимостей.
- Настройка окружения и импорт артефактов.

## Этапы конфигурации

### Определение базовых Docker-образов и создание путей к каждому из них

Необходимые для конфигурации веб-приложения базовые Docker-образы хранятся в файле `image_versions.yml`. Для каждого образа создается базовый путь исходя из условий среды выполнения `production` или `development` и наличия пути к реестру Docker.

### Конфигурация артефакта статической веб-страницы

Создается артефакт `web-static`, который выполняет генерацию статической веб-страницы с помощью Ansible и Jekyll. Артефакт — это особый вид Docker-образа, который используется в deckhouse. Он предназначен преимущественно для отделения ресурсов инструментов сборки от процесса сборки образа приложения.

С помощью Ansible происходит установка зависимостей и сборка статических файлов. Затем определяются файлы и каталоги, которые должны быть добавлены в проект из Git репозитория.

### Конфигурация бэкенда

Для настройки сборки и запуска бэкенда создается образ `web-backend`. Рабочим каталогом контейнера является директория `/app`. Базовым образом контейнера выступает значение переменной `$BASE_GOLANG_16_BUSTER`, определенной в начале файла.

```YAML
{{ $BASE_GOLANG_16_BUSTER := "golang:1.16.3-buster@sha256:9d64369fd3c633df71d7465d67d43f63bb31192193e671742fa1c26ebc3a6210" }}
```

Ansible выполняет обновление и установку пакетов и инструментов git, curl и jq, а также утилиты для работы с Go модулями. Выполняется сборка приложения внутри Docker-контейнера и клонирование кода из удаленного Git репозитория в директорию `/go/src/app`. Артефакт `web-static` импортируется в директорию `/app/root` до настройки окружения.

### Конфигурация фронтенда

Для настройки сборки и запуска фронтенда создается образ `web-frontend`. Рабочим каталогом контейнера является директория `/app`. Базовым образом контейнера выступает значение переменной `.Images.BASE_NGINX_ALPINE`.

Ansible выполняет форматирование файла `nginx.conf` и копирует его в директорию `/etc/nginx`. Данные артефакта `web-static` копируются в директорию `/app` с указанием прав доступа для владельца и группы nginx до настройки окружения.
