# Шаблон обратного проксирования на traefik
## ВНИМАНИЕ ИНСТРУКЦИЯ ДЕМОНСТРИРУЕТ ПРОКСИРОВАНИЕ С ДОМЕНОВ 3го УРОВНЯ
## Описание
Данный шаблон демонстрирует пример обратного проксирования traefik для контейнеров докера.  
В этом примере Traefik:
* Перенаправляет все запросы с http на https.
* Автоматически генерирует и перегенерирует сертификаты let's encrypt.
* Проксирует запросы к контейнерам докера, без необходимости на них пробрасывать порты наружу.

## Используемые технологии:
* docker
* docker-compose v3
* traefik v2.5
* htpasswd из пакета apache2-utils

## Инструкция по созданию обратного прокси для контейнеров докер.
1. В настройках DNS вашего регистратора, добавьте записи доменов 3го уровня. Примеры:  

| NAME | TYPE | VALUE |
| --- | --- | --- |
| example.example.com | A | 192.168.0.1 |
| example | A | 192.168.0.1 |
| * | A | 192.168.0.1 |

2. Дайте права 600 на фаил [acme.json](./acme.json).
```bash
sudo chmod 600 acme.json
```

3. Добавьте labels в docker-compose.yml проекта, который должен будет хоститься на домене 3го уровня. Пример находится в файле [docker-compose.another-project.yml](./docker-compose.another-project.yml).

4. Создайте общую сеть для проксируемых контейнеров docker. В моём примере она носит название traefik. В неё нужно будет пробрасывать все интересующие контейнеры docker.
```bash
docker network create traefik
```
5. Если вы оставили включённой панель управления traefik, измените traefik.example.com на ваш домен в файле: [docker-compose.yml](docker-compose.yml).

6. При наличии авторизации для панели управления traefik. Создайте пароль для базовой авторизации панели traefic командой:
```bash
echo $(htpasswd -nb USER PASSWORD) | sed -e s/\\$/\\$\\$/g
```
Пример:
```bash
echo $(htpasswd -nb john 2021john093124a) | sed -e s/\\$/\\$\\$/g
```
^ вывод этой команды нужно будет полностью в ставить в [docker-compose.yml](./docker-compose.yml) в "traefik.http.middlewares.basic-auth.basicauth.users".  
Пример:
```yml
labels:
 - "traefik.http.middlewares.basic-auth.basicauth.users=john:$$apr1$$OTx4Ge0E$$gSBhoiWdi90Z1ggNcLzn2/"
```

7. Запустите контейнер с traefik коммандой:
```bash
docker-compose up -d
```
## Хитрости и советы
* Если у вас для домена был сгенерирован сертификат, но браузер пишет что он недействителен то его можно перегенерировать. Для этого вам нужно убрать сертификат из "acme.json".
Пример готового сертификата:
```json
{
  "domain": {
    "main": "example.example.com"
  },
  "certificate": "certificate",
  "key": "key",
  "Store": "default"
},
```
Подобную структуру нужно будет удалить.
