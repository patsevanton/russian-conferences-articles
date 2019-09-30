Предлагаю вашему вниманию перевод поста: [Миграция с Nginx на Envoy Proxy](https://www.katacoda.com/envoyproxy/scenarios/migrating-from-nginx-to-envoy).

Envoy - это высокопроизводительный распределенный прокси-сервер (написанный на C++), предназначенный для отдельных сервисов и приложений, также это коммуникационная шина и «universal data plane», разработанный для больших микросервисных архитектур «service mesh. При его создании были учтены решения проблем, которые возникали при разработке таких серверов, как NGINX, HAProxy, аппаратных балансировщиков нагрузки и облачных балансировщиков нагрузки. Envoy работает вместе с каждым приложением и абстрагирует сеть, предоставляя общие функции независимо от платформы. Когда весь служебный трафик в инфраструктуре проходит через сетку Envoy, становится легко визуализировать проблемные области с помощью согласованной наблюдаемости, настройки общей производительности и добавления основных функций в определённом месте.


## Возможности

- Архитектура вне процесса: envoy - это автономный, высокопроизводительный сервер занимающий небольшой объем оперативной памяти. Он работает совместно с любым языком приложения или фреймворком.
- Поддержка http/2 и grpc: envoy имеет первоклассную поддержку http/2 и grpc для входящих и исходящих соединений. Это прозрачный прокси от http/1.1 до http/2.
- Расширенная балансировка нагрузки: envoy поддерживает расширенные функции балансировки нагрузки, включая автоматические повторные попытки, разрыв цепи, глобальное ограничение скорости, затенение запросов, локальную балансировку нагрузки зоны и т. д.
- API для управления конфигурацией: envoy предоставляет надежный API для динамического управления своей конфигурацией.
- Наблюдаемость: глубокая наблюдаемость трафика L7, встроенная поддержка распределенной трассировки и наблюдаемость на уровне проводов mongodb, dynamodb и многое другое.

<cut/>


## Шаг 1 - Пример конфига NGINX

В этом сценарии используется специально созданный файл *nginx.conf*, основанный на полном примере из [NGINX Wiki](https://www.nginx.com/resources/wiki/start/topics/examples/fullexample2/). Вы можете просмотреть конфигурацию в редакторе, открыв *nginx.conf*



Исходный конфиг nginx

```nginx
user  www www;
pid /var/run/nginx.pid;
worker_processes  2;

events {
  worker_connections   2000;
}

http {
  gzip on;
  gzip_min_length  1100;
  gzip_buffers     4 8k;
  gzip_types       text/plain;

  log_format main      '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$gzip_ratio"';

  log_format download  '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$http_range" "$sent_http_content_range"';


  upstream targetCluster {
    172.18.0.3:80;
    172.18.0.4:80;
  }

  server {
    listen        8080;
    server_name   one.example.com  www.one.example.com;

    access_log   /var/log/nginx.access_log  main;
    error_log  /var/log/nginx.error_log  info;

    location / {
      proxy_pass         http://targetCluster/;
      proxy_redirect     off;

      proxy_set_header   Host             $host;
      proxy_set_header   X-Real-IP        $remote_addr;
    }
  }
}
```

Конфигурации NGINX обычно имеют три ключевых элемента:

1) Настройка сервера NGINX, структуры журналов и функциональности Gzip. Это определяется глобально во всех случаях.

2) Настройка NGINX для приема запросов на хост *one.example.com* на порту 8080.

3) Настройка целевого местоположения, как обрабатывать трафик для разных частей URL.

Не вся конфигурация будет применяться к Envoy Proxy, и вам не нужно настраивать некоторые параметры. Envoy Proxy имеет **четыре ключевых типа**, которые поддерживают базовую инфраструктуру, предлагаемую NGINX. Ядро это:

- **Слушатели:** Они определяют, как Envoy Proxy принимает входящие запросы. В настоящее время Envoy Proxy поддерживает только слушателей на основе TCP. Как только соединение установлено, оно передается на набор фильтров для обработки.
- **Фильтры:** Они являются частью конвейерной архитектуры, которая может обрабатывать входящие и исходящие данные. Данная функциональность включает фильтры, такие как Gzip, который сжимает данные перед отправкой их клиенту.
- **Маршрутизаторы:** Они перенаправляют трафик в требуемый пункт назначения, определенный как кластер.
- **Кластеры:** Они определяют конечную точку для трафика и параметры конфигурации.

Мы будем использовать эти четыре компонента для создания конфигурации Envoy Proxy чтобы соответстветствовать определенной конфигурации NGINX. Цель Envoy работа с API и динамическая конфигурация. В этом случае базовая конфигурация будет использовать статические, жестко заданные параметры из NGINX.



## Шаг 2 - Конфигурация NGINX

Первая часть *nginx.conf* определяет некоторые внутренние компоненты NGINX, которые должны быть настроены.

#### Worker Connections (Рабочие соединения)

Приведенная ниже конфигурация определяет количество рабочих процессов и соединений. Это указывает на то, как NGINX будет масштабироваться для удовлетворения спроса.

```nginx
worker_processes  2;

events {
  worker_connections   2000;
}
```

Envoy Proxy по-разному управляет рабочими процессами и соединениями.

Envoy создает рабочий поток для каждого аппаратного потока в системе. Каждый рабочий поток выполняет неблокирующий цикл событий, который отвечает за

2. Прослушивание каждого слушателя (listener)
4. Принятие новых подключений
6. Создание набора фильтров для соединения
8. Обработка всех операций ввода-вывода за время существования соединения.

Вся дальнейшая обработка соединения полностью обрабатывается в рабочем потоке, включая любое поведение переадресации.

Для каждого рабочего потока в Envoy существует соединение в пуле. Таким образом, пулы соединений HTTP/2 устанавливают только одно соединение для каждого внешнего хоста за раз, при наличии четырех рабочих потоков будет четыре соединения HTTP/2 для каждого внешнего хоста в стабильном состоянии. Сохраняя все в одном рабочем потоке, почти весь код может быть написан без блокировок, как если бы он был однопоточным. Если рабочих потоков выделено больше чем необходимо, это может привести к не рациональному использованию памяти, созданию большого количества простаивающих соединений и уменьшения числа возвратов соединения обратно в пул.

Для получения дополнительной информации посетите [Envoy Proxy blog](https://blog.envoyproxy.io/envoy-threading-model-a8d44b922310).

#### Конфигурация HTTP

Следующий блок конфигурации NGINX определяет настройки HTTP, такие как:

- Какие mime типы поддерживаются
- Тайм-ауты по умолчанию
- Конфигурация Gzip

Вы можете настроить эти аспекты с помощью фильтров в Envoy Proxy, что мы обсудим позже.

## Шаг 3 - Конфигурация Server

В блоке конфигурирования HTTP, конфигурация NGINX указывает прослушивать порт 8080 и отвечать на входящие запросы для доменов *one.example.com* и *www.one.example.com*.

```nginx
 server {
    listen        8080;
    server_name   one.example.com  www.one.example.com;
```

Внутри Envoy этим управляют Слушатели.

#### Слушатели Envoy

Самый важный аспект начала работы с Envoy Proxy - это определение слушателей. Вам нужно создать файл конфигурации, который описывает, как вы хотите запустить экземпляр Envoy.

Приведенный ниже фрагмент создаст нового слушателя и свяжет его с портом 8080. Конфигурация указывает Envoy Proxy, к каким портам он должен быть привязан для входящих запросов.

Envoy Proxy использует нотацию YAML для своей конфигурации. Для знакомства с этой нотацией, посмотрите здесь [ссылку](https://yaml.org/spec/1.2/spec.html).

```yaml
Copy to Editorstatic_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
```

Нет необходимости определять *server_name*, так как фильтры Envoy Proxy справятся с этим.

## Шаг 4 - Конфигурация местоположения

Когда запрос поступает в NGINX, блок местоположения определяет, как обрабатывать и куда направлять трафик. В следующем фрагменте весь трафик на сайт передается в upstream (примечание переводчика: upstream - обычно это сервера приложений) кластер с именем *targetCluster*. Upstream кластер определяет узлы, которые должны обрабатывать запрос. Мы обсудим это на следующем шаге.

```nginx
location / {
    proxy_pass         http://targetCluster/;
    proxy_redirect     off;

    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
}
```

В Envoy этим занимается Filters.

#### Envoy Filters

Для статической конфигурации фильтры определяют, как обрабатывать входящие запросы. В этом случае мы устанавливаем фильтры, которые соответствуют *server_names* на предыдущем шаге. Когда поступают входящие запросы, которые соответствуют определенным доменам и маршрутам, трафик направляется в кластер. Это эквивалент восходящей конфигурации NGINX.

```yaml
Copy to Editor    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: backend
              domains:
                - "one.example.com"
                - "www.one.example.com"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: targetCluster
          http_filters:
          - name: envoy.router
```

Имя *envoy.http_connection_manager* является встроенным фильтром в Envoy Proxy. Другие фильтры включают *Redis*, *Mongo*, *TCP*. Вы можете найти полный список в [документации](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-file-envoy-api-v2-listener-listener-proto).

Для получения дополнительной информации о других политиках балансировки нагрузки посетите [Документацию Envoy](https://www.envoyproxy.io/docs/envoy/v1.8.0/intro/arch_overview/load_balancing).



## Step 5 - Proxy and Upstream Configuration

В NGINX конфигурация upstream определяет набор целевых серверов, которые будут обрабатывать трафик. В этом случае два кластера были назначены.

```nginx
  upstream targetCluster {
    172.18.0.3:80;
    172.18.0.4:80;
  }
```

В Envoy это управляется кластерами.

#### Envoy Clusters

Эквивалент upstream определяется как кластеры. В этом случае были определены хосты, которые будут обслуживать трафик. Способ доступа к хостам, например время ожидания, определяется как конфигурация кластера. Это позволяет более точно контролировать зернистость таких аспектов, как время ожидания и балансировка нагрузки.

```yaml
Copy to Editor  clusters:
  - name: targetCluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    hosts: [
      { socket_address: { address: 172.18.0.3, port_value: 80 }},
      { socket_address: { address: 172.18.0.4, port_value: 80 }}
    ]
```

При использовании обнаружения службы *STRICT_DNS* Envoy будет непрерывно и асинхронно разрешать указанные цели DNS. Каждый возвращенный IP-адрес в результате DNS будет считаться явным хостом в восходящем кластере. Это означает, что если запрос возвращает два IP-адреса, Envoy предположит, что в кластере есть два хоста, и оба должны быть сбалансированы по нагрузке. Если хост удален из результата, Envoy предполагает, что он больше не существует, и будет отбирать трафик из любых существующих пулов соединений.

Для получения дополнительной информации см. [Документацию прокси-сервера Envoy](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/service_discovery#strict-dns).

## Шаг 6 - Доступ к журналу и ошибки

Окончательная конфигурация - регистрация. Вместо того, чтобы передавать журналы ошибок на диск, Envoy Proxy использует облачный подход. Все журналы приложений выводятся в *stdout* и *stderr*.

Когда пользователи делают запрос, журналы доступа являются необязательными и по умолчанию отключены. Чтобы включить журналы доступа для запросов HTTP, включите конфигурацию *access_log* для диспетчера подключений HTTP. Путь может быть либо устройством, таким как *stdout*, либо файлом на диске, в зависимости от ваших требований.

Следующая конфигурация перенаправит все журналы доступа в *stdout* (примечание переводчика - stdout необходимо для использования envoy внутри docker. Если использовать без docker, то замените /dev/stdout на путь до обычного лог-файла). Скопируйте фрагмент в раздел конфигурации для менеджера соединений:

```yaml
Copy to Clipboardaccess_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
```

Результаты должны выглядеть так:

```yaml
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          access_log:
          - name: envoy.file_access_log
            config:
              path: "/dev/stdout"
          route_config:
```

По умолчанию Envoy имеет строку формата, которая включает подробности HTTP-запроса:

```
[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%" "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n
```

Результат этой строки формата:

```bash
[2018-11-23T04:51:00.281Z] "GET / HTTP/1.1" 200 - 0 58 4 1 "-" "curl/7.47.0" "f21ebd42-6770-4aa5-88d4-e56118165a7d" "one.example.com" "172.18.0.4:80"
```

Содержимое вывода можно настроить, установив поле формата. Например:

```yaml
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    format: "[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n"
```

Строка журнала также может быть выведена в формате JSON путем установки поля *json_format*. Например:

```yaml
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    json_format: {"protocol": "%PROTOCOL%", "duration": "%DURATION%", "request_method": "%REQ(:METHOD)%"}
```

Для получения дополнительной информации о методике регистрации посланника, посетите

<https://www.envoyproxy.io/docs/envoy/latest/configuration/access_log#config-access-log-format-dictionaries>

Ведение журнала - не единственный способ получить представление о работе с Envoy Proxy. В него встроены расширенные возможности трассировки и метрик. Вы можете узнать больше в [документации по трассировке](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/tracing) или через [Сценарий интерактивной трассировки](https://www.envoyproxy.io/try/implementing-metrics-tracing).



## Шаг 7 - Запуск

Теперь вы перевели конфигурацию с NGINX на Envoy Proxy. Последний шаг - запустить экземпляр Envoy Proxy для его тестирования.

#### Запуск от пользователя

В верхней части конфигурации NGINX строка *user www www;* указывает на запуск NGINX в качестве пользователя с низким уровнем привилегий для повышения безопасности.

Envoy Proxy использует облачный подход к управлению тем, кто является владельцем процесса. Когда мы запускаем Envoy Proxy через контейнер, мы можем указать пользователя с низким уровнем привилегий.

#### Запуск Envoy Proxy

Команда ниже запустит Envoy Proxy через Docker-контейнер на хосте. Эта команда предоставляет Envoy возможность прослушивать входящие запросы через порт 80. Однако, как указано в конфигурации прослушивателя, Envoy Proxy прослушивает входящий трафик через порт 8080. Это позволяет процессу запускаться как пользователь с низкими привилегиями.

```bash
docker run --name proxy1 -p 80:8080 --user 1000:1000 -v /root/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy
```

#### Тестирование

С запущенным прокси, тесты теперь могут быть сделаны и обработаны. Следующая команда cURL выдает запрос с заголовком узла, определенным в конфигурации прокси.

```bash
curl -H "Host: one.example.com" localhost -i
```

Запрос HTTP приведет к ошибке *503*. Это связано с тем, что восходящие соединения не работают и они недоступны. Таким образом, у Envoy Proxy нет доступных целевых адресатов для запроса. Следующая команда запустит серию HTTP-сервисов, которые соответствуют конфигурации, определенной для Envoy.

```bash
docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server;
```

С доступными услугами Envoy может успешно проксировать трафик к месту назначения.

```bash
curl -H "Host: one.example.com" localhost -i
```

Вы должны увидеть ответ, указывающий, какой контейнер Docker обработал запрос. В журналах Envoy Proxy вы также должны увидеть выведенную строку доступа.

#### Дополнительные заголовки ответа HTTP (HTTP Response)

В заголовках ответа действительного запроса вы увидите дополнительные заголовки HTTP. В заголовке отображается время, которое вышестоящий хост потратил на обработку запроса. Выражается в миллисекундах. Это полезно, если клиент хочет определить время обслуживания по сравнению с задержкой в сети.

```
x-envoy-upstream-service-time: 0
server: envoy
```



## Итоговый конфиг

```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: backend
              domains:
                - "one.example.com"
                - "www.one.example.com"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: targetCluster
          http_filters:
          - name: envoy.router
          clusters:
  - name: targetCluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    hosts: [
      { socket_address: { address: 172.18.0.3, port_value: 80 }},
      { socket_address: { address: 172.18.0.4, port_value: 80 }}
    ]

admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9090 }
```



## Допонительная информация от переводчика

Инструкции по установке Envoy Proxy вы можете найти на сайте https://www.getenvoy.io/

По умолчанию в rpm отсутствует systemd service конфиг.

Добавьте systemd service конфиг /etc/systemd/system/envoy.service:

```bash
[Unit]
Description=Envoy Proxy
Documentation=https://www.envoyproxy.io/
After=network-online.target
Requires=envoy-auth-server.service
Wants=nginx.service

[Service]
User=root
Restart=on-failure
ExecStart=/usr/bin/envoy --config-path /etc/envoy/config.yaml
[Install]
WantedBy=multi-user.target
```

Вам нужно создать директорию /etc/envoy/ и положить туда конфиг config.yaml.

По envoy proxy есть телеграм чат: https://t.me/envoyproxy_ru

