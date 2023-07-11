# Установка teamcity на сервер centos

создание директорий для volume

```bash
cd /srv
mkdir -p teamcity
mkdir teamcity/data
mkdir teamcity/logs
mkdir teamcity/agent
```

скачиваем образ
```bash
docker pull jetbrains/teamcity-server
```

запускаем сервер

```bash
docker run -d -u 0 --name teamcity-server-instance  \
    -v /srv/teamcity/data:/data/teamcity_server/datadir \
    -v /srv/teamcity/logs:/opt/teamcity/logs  \
    -v /srv/teamcity/plugins:/opt/teamcity/plugins  \
    -p 8111:8111 \
    jetbrains/teamcity-server
```

После запуска переходим по адресу: 
```
http://ip:8111/
```

# Docker 

## Teamcity

Установить плагины
Administration->Plugins

```yaml
Docker
Docker Compose
```

## Права пользователя Docker

Пользователя агента нужно добавить в группу Docker, чтобы он могу запускать docker сборки
```
sudo usermod -aG docker $USER
```
*Агента нужно авторизовать заново, иначе настройки не вступят в силу*

## Настройка

Для работы предварительно нужно создать базу данных mysql с удаленным подключением так как docker изолирован
И во время установки указываем базу для подключения


## Права на директории

без них не будет работать нормально

```bash
chmod 666 /var/run/docker.sock \
  sudo chown -R 1000:1000 /srv/teamcity/agent/conf \ 
  sudo chown -R 1000:1000 /srv/teamcity/buildagent/work \
  sudo chown -R 1000:1000 /srv/teamcity/buildagent/system
```



################ ПРОБЛЕМА С ЗАПУСКОМ DOCKER

# Установка Agent 

Для работы нам нужен рабочий агент(runners)
** информация о проблемах https://habr.com/ru/company/visiology/blog/594515/ **

```bash
docker pull jetbrains/teamcity-agent:2021.2-linux-sudo
```


## Запуск агента

```bash
docker run -dt \
  -u 0 \
  -e SERVER_URL="http://ip:8111/" \
  --name teamcity-agent-instance \
  -v /srv/teamcity/agent/conf:/data/teamcity_agent/conf \
  --privileged -e DOCKER_IN_DOCKER=start \
  --restart=always \
  --log-driver=none \
  --network host \
  custom-agent-image:2021.2-linux-sudo
```


# Восстановления бэкапа

Если с файла на серерве то файл нужно положить в 
data/
путь в контейнере получиться 
/data/teamcity_server/datadir/dump.zip
