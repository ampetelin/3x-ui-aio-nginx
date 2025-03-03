
# 3x-ui-aio-nginx

## Запуск

### 1. Создайте два поддомена:
Например:
- `panel.example.com` - панель 3x-ui 
- `cloud.example.com` - xray/сайт-заглушка

A-записи этих поддоменов должны содержать IP вашего VPS

### 2. Установите certbot

Инструкции по установке: [https://certbot.eff.org/](https://certbot.eff.org/)

### 3. Получите сертификаты для поддоменов:

Для поддомена `panel.example.com`:

```bash
sudo certbot certonly --standalone -m <your_email> -d panel.example.com --agree-tos --no-eff-email
```

Для поддомена `cloud.example.com`:

```bash
sudo certbot certonly --standalone -m <your_email> -d cloud.example.com --agree-tos --no-eff-email
```

### 4. Установите Docker

Инструкции по установке Docker: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

### 5. Клонируйте репозиторий

```bash
git clone https://github.com/ampetelin/3x-ui-aio-nginx.git && cd 3x-ui-aio-nginx
```

### 6. Отредактируйте файл `nginx.conf`

Замените:
- `<your_panel_domain>` на поддомен для панели (например, `panel.example.com`)
- `<your_xray_domain>` на поддомен для xray (например, `cloud.example.com`)

### 7. Запустите Docker Compose

```bash
docker compose up -d
```

### 8. Перейдите в панель

Откройте браузер и перейдите по адресу: [https://panel.example.com](https://panel.example.com)

**Дефолтные данные для авторизации:**  
- Логин: `admin`  
- Пароль: `admin`

> [!CAUTION]
> Обязательно измените дефолтный логин и пароль в разделе **Panel Settings -> Authentication**.

> [!WARNING]
> Для повышения безопасности рекомендуется изменить путь до панели в разделе **Panel Settings -> URI Path**.
> 
> Например, при изменении URI Path на `/my-secret-panel-path/` панель будет доступна только по адресу [https://panel.example.com/my-secret-panel-path/](https://panel.example.com/my-secret-panel-path/)

### 9. Добавьте inbound в панели 3x-ui

При добавлении inbound настройте следующие обязательные параметры:

- **Protocol:** vless
- **Port:** 8443
- **Proxy Protocol:** Enabled
- **External Proxy:** cloud.example.com:443
- **Security:** Reality
- **Xver:** 1
- **Dest (Target):** nginx:2443
- **SNI:** cloud.example.com

![inbound](https://github.com/user-attachments/assets/46583c72-dbcb-4f9f-a024-f83f16480421)



### 10. Отредактируйте конфигурационные файлы обновления сертификатов

- /etc/letsencrypt/renewal/panel.example.com
- /etc/letsencrypt/renewal/cloud.example.com

В конфигурационных файлах требуется изменить `authenticator = standalone` на `authenticator = webroot` и добавить `webroot_path = /var/www/certbot/`

### 11. Добавьте хук перезапуска nginx после обновления сертификатов
Создайте скрипт `restart-nginx.sh`
```shell
sudo nano /etc/letsencrypt/renewal-hooks/post/restart-nginx.sh
```
Вставьте в файл и сохраните
```bash
#!/bin/bash
docker exec nginx nginx -s reload 2>&1
```
Сделайте его исполняемым
```shell
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/restart-nginx.sh
```

### 12. Протестируйте обновление сертификатов
```shell
sudo certbot renew --dry-run
```
