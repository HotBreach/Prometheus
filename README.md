
# Мониторинговый стек на базе Grafana, Prometheus, Loki, Promtail и Nginx

Данный проект предоставляет полный стек для мониторинга и сбора логов с помощью Docker Compose. В его состав входят Grafana для визуализации, Prometheus для сбора метрик, Loki и Promtail для агрегации логов и Nginx в роли обратного прокси.

## Состав проекта
- **Grafana**: Визуализация метрик и логов, HTTPS, кастомная конфигурация.
- **Prometheus**: Сбор метрик, HTTPS, кастомная конфигурация.
- **Loki**: Агрегация логов.
- **Promtail**: Сбор логов с Docker-контейнеров и отправка в Loki.
- **Nginx**: Редирект HTTP на HTTPS.

## Структура проекта
```text
docker-compose.yml
grafana/
  grafana.ini
  certs/
    cert.crt
    key.key
prometheus/
  prometheus.yml
  web-config.yml
  certs/
    cert.crt
    key.key
promtail/
  promtail-config.yaml
nginx/
  nginx.conf
```

## Быстрый старт
1. Клонируйте репозиторий и перейдите в папку проекта.
2. Убедитесь, что у вас есть Docker и Docker Compose.
3. Запустите команду:
   ```pwsh
   docker-compose up -d
   ```
4. Откройте [https://localhost](https://localhost) для доступа к Grafana (логин/пароль: admin/admin).

## Конфигурация
- **Grafana**: `grafana/grafana.ini` — настройки сервера, HTTPS, логин/пароль.
- **Prometheus**: `prometheus/prometheus.yml` — сбор метрик, `web-config.yml` — TLS.
- **Promtail**: `promtail/promtail-config.yaml` — сбор логов с Docker.
- **Nginx**: `nginx/nginx.conf` — редирект HTTP на HTTPS.
- **Сертификаты**: Хранятся в папках `grafana/certs` и `prometheus/certs`.

## Порты
- Grafana: 443 (HTTPS)
- Prometheus: 8443 (HTTPS)
- Loki: 3100
- Nginx: 80 (HTTP)

## Заметки
- Все сервисы запускаются в отдельных контейнерах.
- Для работы HTTPS необходимы валидные сертификаты (можно использовать self-signed для теста).
- Для изменения настроек редактируйте соответствующие конфиги в папках сервисов.

## Лицензия
MIT
