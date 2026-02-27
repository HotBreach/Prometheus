
# 📊 Мониторинговый стек на базе Grafana, Prometheus, Loki и Promtail

Проект предоставляет полный стек для мониторинга и сбора логов с помощью Docker Compose. В состав входят Grafana для визуализации, Prometheus для сбора метрик, Loki и Promtail для агрегации логов.

## 📦 Состав проекта

- 🟢 **Grafana** — визуализация метрик и логов, HTTPS, LDAP-аутентификация через AD.
- 🔵 **Prometheus** — сбор метрик по HTTP.
- 🟣 **Loki** — агрегация логов.
- 🟠 **Promtail** — сбор логов с Docker-контейнеров и отправка в Loki.

## 🗂️ Структура проекта

```text
docker-compose.yml
grafana/
  grafana.ini            # Настройки сервера (HTTPS, LDAP)
  ldap.toml              # Конфигурация LDAP/AD
  certs/
    openssl.conf         # Конфиг для генерации ключа и CSR
    cert.crt             # Сертификат (получается от CA)
    key.key              # Приватный ключ
prometheus/
  prometheus.yml         # Конфигурация сбора метрик
promtail/
  promtail-config.yaml
```

## ⚡ Быстрый старт

1. Клонируйте репозиторий и перейдите в папку проекта.
2. Убедитесь, что установлены Docker и Docker Compose.
3. Выпустите сертификат для Grafana (см. раздел [🔐 Сертификаты](#-сертификаты)).
4. Заполните `grafana/ldap.toml` параметрами вашего AD (хост, bind_dn, пароль, search_base_dns).
5. Запустите:

   ```bash
   docker-compose up -d
   ```

6. Откройте [https://localhost](https://localhost) для доступа к Grafana.

## ⚙️ Конфигурация

- **Grafana**: `grafana/grafana.ini` — настройки сервера, HTTPS, LDAP.
- **Prometheus**: `prometheus/prometheus.yml` — задачи сбора метрик (работает по HTTP).
- **Promtail**: `promtail/promtail-config.yaml` — сбор логов с Docker.
- **LDAP**: `grafana/ldap.toml` — подключение к Active Directory, маппинг групп на роли Grafana.

## 🔌 Порты

| Сервис     | Порт | Протокол |
|------------|------|----------|
| Grafana    | 443  | HTTPS    |
| Prometheus | 9090 | HTTP     |
| Loki       | 3100 | HTTP     |

## 🔐 Сертификаты

Grafana использует TLS. Сертификат выпускается через корпоративный CA с помощью CSR-запроса.

Конфиг для генерации находится в `grafana/certs/openssl.conf`. Перед использованием отредактируй в нём:

- `CN` — FQDN хоста Grafana
- `DNS.*` — все DNS-имена, по которым будет доступен сервис

### 📋 Шаги получения сертификата

**1. Сгенерируй приватный ключ и CSR-запрос:**

```bash
cd grafana/certs

openssl req -new -nodes \
  -config openssl.conf \
  -keyout key.key \
  -out grafana.csr
```

> ⚠️ Флаг `-nodes` обязателен — он отключает шифрование приватного ключа паролем. Без него OpenSSL запросит PEM passphrase, и если оставить поле пустым, команда завершится ошибкой `result too small`. Кроме того, зашифрованный ключ потребует ручного ввода пароля при каждом старте Grafana.

После выполнения появятся два файла:

- `key.key` — приватный ключ (не передавать никому)
- `grafana.csr` — запрос на подпись, который нужно отправить в CA

**2. Передай `grafana.csr` администратору CA** для подписания.

**3. Получи подписанный сертификат** и сохрани его как `grafana/certs/cert.crt`.

**4. Проверь, что сертификат и ключ совпадают:**

```bash
# Хэши должны быть одинаковыми
openssl x509 -noout -modulus -in cert.crt | openssl md5
openssl rsa  -noout -modulus -in key.key  | openssl md5
```

**5. Просмотреть содержимое сертификата:**

```bash
openssl x509 -in cert.crt -text -noout
```

> ⚠️ Файлы `key.key` и `grafana.csr` не должны попадать в git-репозиторий. Убедись, что они указаны в `.gitignore`.

## 🏢 LDAP / Active Directory

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

## 📝 Заметки

- Prometheus работает без TLS — в продакшене закройте порт 9090 на firewall'е.
- Приватный ключ `key.key` храните только на сервере, не коммитьте в git.
