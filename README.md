
# Мониторинговый стек на базе Grafana, Prometheus, Loki и Promtail

Проект предоставляет полный стек для мониторинга и сбора логов с помощью Docker Compose. В состав входят Grafana для визуализации, Prometheus для сбора метрик, Loki и Promtail для агрегации логов.

## Состав проекта

- **Grafana**: Визуализация метрик и логов, HTTPS, LDAP-аутентификация через AD.
- **Prometheus**: Сбор метрик по HTTP.
- **Loki**: Агрегация логов.
- **Promtail**: Сбор логов с Docker-контейнеров и отправка в Loki.

## Структура проекта

```text
docker-compose.yml
grafana/
  grafana.ini          # Настройки сервера (HTTPS, LDAP)
  ldap.toml            # Конфигурация LDAP/AD
  certs/
    cert.crt
    key.key
prometheus/
  prometheus.yml       # Конфигурация сбора метрик
promtail/
  promtail-config.yaml
```

## Быстрый старт

1. Клонируйте репозиторий и перейдите в папку проекта.
2. Убедитесь, что установлены Docker и Docker Compose.
3. Положите TLS-сертификаты для Grafana в папку `grafana/certs/` (`cert.crt` и `key.key`).
4. Заполните `grafana/ldap.toml` параметрами вашего AD (хост, bind_dn, пароль, search_base_dns).
5. Запустите:
   ```bash
   docker-compose up -d
   ```
6. Откройте [https://localhost](https://localhost) для доступа к Grafana.

## Конфигурация

- **Grafana**: `grafana/grafana.ini` — настройки сервера, HTTPS, LDAP.
- **Prometheus**: `prometheus/prometheus.yml` — задачи сбора метрик (работает по HTTP).
- **Promtail**: `promtail/promtail-config.yaml` — сбор логов с Docker.
- **LDAP**: `grafana/ldap.toml` — подключение к Active Directory, маппинг групп на роли Grafana.

## Порты

| Сервис     | Порт | Протокол |
|------------|------|----------|
| Grafana    | 443  | HTTPS    |
| Prometheus | 9090 | HTTP     |
| Loki       | 3100 | HTTP     |

## LDAP / Active Directory

Grafana настроена на аутентификацию через AD. Файл конфигурации: `grafana/ldap.toml`.

Ключевые параметры для заполнения:

| Параметр          | Описание                                          |
|-------------------|---------------------------------------------------|
| `host`            | FQDN или IP контроллера домена                    |
| `bind_dn`         | DN сервисного аккаунта для поиска в AD            |
| `bind_password`   | Пароль сервисного аккаунта                        |
| `search_base_dns` | OU, в которой находятся пользователи              |
| `group_dn`        | DN группы AD → роль Grafana (Admin/Editor/Viewer) |

Проверка LDAP после запуска:

```bash
docker logs grafana | grep -i ldap
```

## Заметки

- Для работы HTTPS у Grafana необходимы сертификаты в `grafana/certs/` (можно self-signed).
- Prometheus работает без TLS — доступен только внутри Docker-сети (не проброшен наружу через firewall в продакшене).

## Лицензия

MIT
