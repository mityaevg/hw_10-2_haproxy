# hw_10-2_haproxy
HW_10-2_Кластаризация и балансировка нагрузки

# Домашнее задание к занятию 2 "Кластеризация и балансировка нагрузки"

### Задание 1

1. Запустить 2 simple python-сервера на ВМ на портах **:8888** и **:9999**:

```
mkdir http1 http2
cd http1
nano index.html
cd http2
nano index.html
```
<kbd>![Содержимое файла index.html -сервер 1](img/index_html_server1.png)</kbd>
<kbd>![Содержимое файла index.html -сервер 2](img/index_html_server2.png)</kbd>

Запускаем **Server 1** и **Server 2** в разных вкладках консоли:
```
mityaevg@haproxy-server:~/http1$ python3 -m http.server 8888 --bind 0.0.0.0
mityaevg@haproxy-server:~/http2$ python3 -m http.server 9999 --bind 0.0.0.0
```
<kbd>![Запуск -сервер 1](img/simple_python_server1_running.png)</kbd>
<kbd>![Запуск -сервер 2](img/simple_python_server2_running.png)</kbd>

2. Установим **HAProxy**:
```
sudo apt update && sudo apt install haproxy
```
3. Настройка балансировки **round-robin** на Layer 4 модели **OSI**:
```
root@haproxy-server:~#nano /etc/haproxy/haproxy.cfg nano /etc/haproxy/haproxy.cfg
```
В конфиг-файл добавим следующий разделы:
```
listen stats  # веб-страница со статистикой по адресу http://10.0.2.15:888/stats
       bind                    :888
       mode                    http
       stats                   enable
       stats uri               /stats
       stats refresh           5s
       stats realm             Haproxy\ Statistics

listen web_tcp # балансировка запросов на L4 OSI

        bind :1400

        server s1 127.0.0.1:8888 check inter 3s
        server s2 127.0.0.1:9999 check inter 3s
```
Перезапустим **haproxy.service**:
```
systemctl reload haproxy.service
```
Скриншот, отражающий перенаправление запросов на разные серверы:

<kbd>![Перенаправление запросов на s1 и s2](img/layer4_tcp_queries.png)</kbd>

Скриншот, отображающий статистику запросов через веб-интерфейс, доступный по адресу
**http://10.0.2.15:888/stats**:

<kbd>![Статистика в веб-интерфейсе](img/stats_web-interface.png)</kbd>
