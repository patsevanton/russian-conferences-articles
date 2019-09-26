Envoy is a high performance C++ distributed proxy designed for single services and applications, as well as a communication bus and “universal data plane” designed for large microservice “service mesh” architectures. Built on the learnings of solutions such as NGINX, HAProxy, hardware load balancers, and cloud load balancers, Envoy runs alongside every application and abstracts the network by providing common features in a platform-agnostic manner. When all service traffic in an infrastructure flows via an Envoy mesh, it becomes easy to visualize problem areas via consistent observability, tune overall performance, and add substrate features in a single place.

Envoy - это высокопроизводительный распределенный прокси-сервер (написанный на C++), предназначенный для отдельных сервисов и приложений, а также коммуникационная шина и «универсальный data plane», разработанные для больших микросервисных архитектур «service mesh». Основанный на изучении таких решений, как NGINX, HAProxy, аппаратных балансировщиков нагрузки и облачных балансировщиков нагрузки, Envoy работает параллельно с каждым приложением и абстрагирует сеть, предоставляя общие функции независимо от платформы. Когда весь служебный трафик в инфраструктуре проходит через сетку Envoy, становится легко визуализировать проблемные области с помощью постоянной наблюдаемости, настраивать общую производительность и добавлять функции подложки в одном месте.


## FEATURES Возможности

- out of process architecture: envoy is a self contained, high performance server with a small memory footprint. it runs alongside any application language or framework.
- Архитектура вне процесса: envoy - это автономный, высокопроизводительный сервер с небольшим объемом памяти. Он работает вместе с любым языком приложений или фреймворком.
- http/2 and grpc support: envoy has first class support for http/2 and grpc for both incoming and outgoing connections. it is a transparent http/1.1 to http/2 proxy.
- Поддержка http/2 и grpc: envoy имеет первоклассную поддержку http/2 и grpc для входящих и исходящих соединений. это прозрачный прокси от http/1.1 до http/2.
- advanced load balancing: envoy supports advanced load balancing features including automatic retries, circuit breaking, global rate limiting, request shadowing, zone local load balancing, etc.
- расширенная балансировка нагрузки: Envoy поддерживает расширенные функции балансировки нагрузки, включая автоматические повторные попытки, разрыв цепи, глобальное ограничение скорости, затенение запросов, локальную балансировку нагрузки зоны и т. д.
- apis for configuration management: envoy provides robust apis for dynamically managing its configuration.
- API для управления конфигурацией: envoy предоставляет надежный API для динамического управления своей конфигурацией.
- observability: deep observability of l7 traffic, native support for distributed tracing, and wire-level observability of mongodb, dynamodb, and more.
- Наблюдаемость (Мониторинг): глубокая наблюдаемость трафика L7, встроенная поддержка распределенной трассировки и наблюдаемость на уровне проводов mongodb, dynamodb и многое другое.

<cut/>


## Шаг 1 - Пример конфига NGINX

This scenario uses a crafted *nginx.conf* based on the full example from the [NGINX Wiki](https://www.nginx.com/resources/wiki/start/topics/examples/fullexample2/). You can view the configuration in the editor by opening *nginx.conf*

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

NGINX configurations generally have three key elements:

Конфигурации NGINX обычно имеют три ключевых элемента:

1) Configuring the NGINX server, logging structure and Gzip functionality. This is defined globally across all instances.

1) Настройка сервера NGINX, структуры журналов и функциональности Gzip. Это определяется глобально во всех случаях.

2) Configuring NGINX to accept requests for *one.example.com* host on port 8080.

2) Настройка NGINX для приема запросов на хост *one.example.com* на порту 8080.

3) Configuring the target location of how to handle traffic to different parts of the URL.

3) Настройка целевого местоположения, как обрабатывать трафик для разных частей URL.

Not all of the configuration will apply to Envoy Proxy, and you won’t need to configure certain aspects. Envoy Proxy has **four key types** that support the core infrastructure offered by NGINX. The core is:

Не вся конфигурация будет применяться к Envoy Proxy, и вам не нужно настраивать определенные аспекты. Envoy Proxy имеет **четыре ключевых типа**, которые поддерживают базовую инфраструктуру, предлагаемую NGINX. Ядро это:

- **Listeners:** They define how Envoy Proxy accepts incoming requests. At present, Envoy Proxy only supports TCP-based listeners. Once a connection is made, it’s passed on to a set of filters for processing.
- **Слушатели:** Они определяют, как Envoy Proxy принимает входящие запросы. В настоящее время Envoy Proxy поддерживает только прослушиватели на основе TCP. Как только соединение установлено, оно передается на набор фильтров для обработки.
- **Filters:** They are part of a pipeline architecture that can process inbound and outbound data. This functionality enables filters such as Gzip which compresses data before sending it to the client.
- **Фильтры:** Они являются частью конвейерной архитектуры, которая может обрабатывать входящие и исходящие данные. Эта функция включает фильтры, такие как Gzip, который сжимает данные перед отправкой их клиенту.
- **Routers:** These forward traffic to the required destination, defined as a cluster.
- **Маршрутизаторы:** Они перенаправляют трафик в требуемый пункт назначения, определенный как кластер.
- **Clusters:** They define the target endpoint for traffic and the configuration settings.
- **Кластеры:** Они определяют целевую конечную точку для трафика и параметры конфигурации.

We'll use these four components to create an Envoy Proxy configuration to match the NGINX configuration defined. Envoy's focus has been on API and dynamic configuration. In this case, the configuration will use static, hardcoded resources as defined by NGINX.

Мы будем использовать эти четыре компонента для создания конфигурации Envoy Proxy в соответствии с определенной конфигурацией NGINX. Envoy сосредоточился на API и динамической конфигурации. В этом случае конфигурация будет использовать статические, жестко закодированные ресурсы, как определено NGINX.



## Шаг 2 - Конфигурация NGINX

The first part of the *nginx.conf* defines some of the NGINX internals that should be configured.

Первая часть *nginx.conf* определяет некоторые внутренние компоненты NGINX, которые должны быть настроены.

#### Worker Connections (Рабочие соединения)

The configuration below focuses on defining the number of worker processes and connections. This indicates how NGINX will scale to handle demand.

Приведенная ниже конфигурация фокусируется на определении количества рабочих процессов и соединений. Это указывает на то, как NGINX будет масштабироваться для удовлетворения спроса.

```nginx
worker_processes  2;

events {
  worker_connections   2000;
}
```

Envoy Proxy manages the worker processes and connections differently.

Envoy Proxy по-разному управляет рабочими процессами и соединениями.

Envoy spawns a worker thread for every hardware thread in the system. Each worker thread runs a non-blocking event loop that is responsible for

Envoy создает рабочий поток для каждого аппаратного потока в системе. Каждый рабочий поток выполняет неблокирующий цикл событий, который отвечает за

1. Listening on every listener
2. Прослушивание каждого слушателя
3. Accepting new connections
4. Принятие новых подключений
5. Instantiating a filter stack for the connection
6. Создание набора фильтров для соединения
7. Processing all IO for the lifetime of the connection.
8. Обработка всех операций ввода-вывода за время существования соединения.

All further handling of the connection is entirely processed within the worker thread, including any forwarding behaviour.

Вся дальнейшая обработка соединения полностью обрабатывается в рабочем потоке, включая любое поведение переадресации.

All connection pools in Envoy are per worker thread. Though HTTP/2 connection pools only make a single connection to each upstream host at a time, if there are four workers, there will be four HTTP/2 connections per upstream host at steady state. By keeping everything within a single worker thread, almost all the code can be written without locks and as if it is single threaded. Having more workers than necessary will waste memory, create more idle connections, and lead to a lower connection pool hit rate.

Все пулы соединений в Envoy для каждого рабочего потока. Хотя пулы соединений HTTP/2 устанавливают только одно соединение с каждым вышестоящим хостом за раз, при наличии четырех рабочих будет четыре соединения HTTP/2 для каждого вышестоящего хоста в устойчивом состоянии. Сохраняя все в одном рабочем потоке, почти весь код может быть написан без блокировок и, как если бы он был однопоточным. Наличие большего количества работников, чем необходимо, приведет к потере памяти, созданию большего количества незанятых соединений и снижению частоты обращений в пул соединений.

For more information, visit the [Envoy Proxy blog](https://blog.envoyproxy.io/envoy-threading-model-a8d44b922310).

Для получения дополнительной информации посетите [Envoy Proxy blog](https://blog.envoyproxy.io/envoy-threading-model-a8d44b922310).

#### Конфигурация HTTP

The next block of NGINX configuration defines the HTTP settings, such as:

Следующий блок конфигурации NGINX определяет настройки HTTP, такие как:

- Which mime types are supported
- Какие mime типы поддерживаются
- Default timeouts
- Тайм-ауты по умолчанию
- Gzip configuration
- Конфигурация Gzip

You can configure these aspects via Filters within Envoy Proxy, which we will discuss in a later step.

Вы можете настроить эти аспекты с помощью фильтров в Envoy Proxy, что мы обсудим позже.

## Шаг 3 - Конфигурация Server

Within the HTTP configuration block, the NGINX configuration specifies to listen on port 8080 and respond to incoming requests for the domains *one.example.com* and *www.one.example.com*.

В блоке конфигурации HTTP конфигурация NGINX указывает прослушивать порт 8080 и отвечать на входящие запросы для доменов *one.example.com* и *www.one.example.com*.

```nginx
 server {
    listen        80;
    server_name   one.example.com  www.one.example.com;
```

Within Envoy, this is managed by Listeners.

Внутри Envoy этим управляют Слушатели.

#### Envoy Listeners

The most important aspect of starting with Envoy Proxy is defining the listers. You need to create a configuration file that describes how you want to run the Envoy instance.

Самый важный аспект начала работы с Envoy Proxy - это определение списков. Вам нужно создать файл конфигурации, который описывает, как вы хотите запустить экземпляр Envoy.

The snippet below will create a new listener and bind it to port 8080. The configuration indicates to Envoy Proxy which ports it should be bound to for incoming requests.

Приведенный ниже фрагмент создаст новый прослушиватель и свяжет его с портом 8080. Конфигурация указывает Envoy Proxy, к каким портам он должен быть привязан для входящих запросов.

Envoy Proxy uses YAML notation for its configuration. If you are not familiarized with this notation can see this [link](https://yaml.org/spec/1.2/spec.html).

Envoy Proxy использует нотацию YAML для своей конфигурации. Если вы не знакомы с этой нотацией, можете посмотреть эту  [ссылку](https://yaml.org/spec/1.2/spec.html).

```yaml
Copy to Editorstatic_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
```

There is no need to define the *server_name* as Envoy Proxy filters will handle this.

Нет необходимости определять *server_name*, так как фильтры Envoy Proxy справятся с этим.

## Step 4 - Location Configuration

When a request comes into NGINX, a location block defines how to process and where to forward the traffic. In the following snippet, all the traffic to the site is proxied to an upstream cluster called *targetCluster*. The upstream cluster defines the nodes that should process the request. We will discuss this in the next step.

Когда запрос поступает в NGINX, блок местоположения определяет, как обрабатывать и куда направлять трафик. В следующем фрагменте весь трафик на сайт передается в восходящий кластер с именем *targetCluster*. Восходящий кластер определяет узлы, которые должны обрабатывать запрос. Мы обсудим это на следующем шаге.

```nginx
location / {
    proxy_pass         http://targetCluster/;
    proxy_redirect     off;

    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
}
```

Within Envoy, this is managed by Filters.

В Envoy этим занимается Filters.

#### Envoy Filters

For the static configuration, the filters define how to handle incoming requests. In this case, we are setting the filters that match the *server_names* in the previous step. When incoming requests are received that match the defined domains and routes, the traffic is forwarded to the cluster. This is the equivalent of the upstream NGINX configuration.

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

The name *envoy.http_connection_manager* is a built-in filter within Envoy Proxy. Other filters include *Redis*, *Mongo*, *TCP*. You can find the complete list in the [documentation](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-file-envoy-api-v2-listener-listener-proto).

Имя *envoy.http_connection_manager* является встроенным фильтром в Envoy Proxy. Другие фильтры включают *Redis*, *Mongo*, *TCP*. Вы можете найти полный список в [документации](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-file-envoy-api-v2-listener-listener-proto).

For more information about other load balancing policies visit the [Envoy documentation](https://www.envoyproxy.io/docs/envoy/v1.8.0/intro/arch_overview/load_balancing).

Для получения дополнительной информации о других политиках балансировки нагрузки посетите [Документацию Envoy](https://www.envoyproxy.io/docs/envoy/v1.8.0/intro/arch_overview/load_balancing).



## Step 5 - Proxy and Upstream Configuration

Within NGINX, the upstream configuration defines the set of target servers that will handle the traffic. In this case, two clusters have been assigned.

В NGINX конфигурация upstream определяет набор целевых серверов, которые будут обрабатывать трафик. В этом случае два кластера были назначены.

```nginx
  upstream targetCluster {
    172.18.0.3:80;
    172.18.0.4:80;
  }
```

Within Envoy, this is managed by clusters.

В Envoy это управляется кластерами.

#### Envoy Clusters

The equivalent of upstream is defined as Clusters. In this case, the hosts that will serve the traffic have been defined. The way the hosts are accessed, such as the timeouts, are defined as the cluster configuration. This allows finer grain control over aspects such as timeouts and load balancing.

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

When using *STRICT_DNS* service discovery, Envoy will continuously and asynchronously resolve the specified DNS targets. Each returned IP address in the DNS result will be considered an explicit host in the upstream cluster. This means that if the query returns two IP addresses, Envoy will assume the cluster has two hosts, and both should be load balanced to. If a host is removed from the result, Envoy assumes it no longer exists and will drain traffic from any existing connection pools.

При использовании обнаружения службы *STRICT_DNS* Envoy будет непрерывно и асинхронно разрешать указанные цели DNS. Каждый возвращенный IP-адрес в результате DNS будет считаться явным хостом в восходящем кластере. Это означает, что если запрос возвращает два IP-адреса, Envoy предположит, что в кластере есть два хоста, и оба должны быть сбалансированы по нагрузке. Если хост удален из результата, Envoy предполагает, что он больше не существует, и будет отбирать трафик из любых существующих пулов соединений.

For further information, refer to [Envoy Proxy documentation](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/service_discovery#strict-dns).

Для получения дополнительной информации см. [Документацию прокси-сервера Envoy](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/service_discovery#strict-dns).



## Шаг 6 - Доступ к журналу и ошибки

The final configuration is the logging. Instead of piping the error logs to disk, Envoy Proxy follows a cloud-native approach. All the application logs are outputted to *stdout* and *stderr*.

Окончательная конфигурация - регистрация. Вместо того, чтобы передавать журналы ошибок на диск, Envoy Proxy использует облачный подход. Все журналы приложений выводятся в *stdout* и *stderr*.

When users make a request, the access logs are optional and disabled by default. To enable access logs for HTTP requests, include an *access_log*configuration for the HTTP Connection Manager. The path can be either a device, such as *stdout*, or a file on disk, depending on your requirements.

Когда пользователи делают запрос, журналы доступа являются необязательными и по умолчанию отключены. Чтобы включить журналы доступа для запросов HTTP, включите конфигурацию *access_log* для диспетчера подключений HTTP. Путь может быть либо устройством, таким как *stdout*, либо файлом на диске, в зависимости от ваших требований.

The following configuration will pipe all access logs to *stdout*. Copy the snippet into the config section for the connection manager:

Следующая конфигурация перенаправит все журналы доступа в *stdout*. Скопируйте фрагмент в раздел конфигурации для менеджера соединений:

```yaml
Copy to Clipboardaccess_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
```

The results should look like this:

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

By default, Envoy has a format string that includes details of the HTTP request:

По умолчанию Envoy имеет строку формата, которая включает подробности HTTP-запроса:

```
[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%" "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n
```

The result of this format string is:

Результат этой строки формата:

```bash
[2018-11-23T04:51:00.281Z] "GET / HTTP/1.1" 200 - 0 58 4 1 "-" "curl/7.47.0" "f21ebd42-6770-4aa5-88d4-e56118165a7d" "one.example.com" "172.18.0.4:80"
```

The contents of the output can be customised by setting the format field. For example:

Содержимое вывода можно настроить, установив поле формата. Например:

```yaml
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    format: "[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n"
```

The log line can also be outputted as JSON by setting the *json_format* field. For example:

Строка журнала также может быть выведена в формате JSON путем установки поля *json_format*. Например:

```yaml
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    json_format: {"protocol": "%PROTOCOL%", "duration": "%DURATION%", "request_method": "%REQ(:METHOD)%"}
```

For further information on Envoy's logging approach, visit 

Для получения дополнительной информации о методике регистрации посланника, посетите

<https://www.envoyproxy.io/docs/envoy/latest/configuration/access_log#config-access-log-format-dictionaries>

Logging isn't the only way to gain visibility into production with Envoy Proxy. Advanced tracing and metrics capabilities are built into it. You can find out more in the [tracing documentation](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/tracing) or via the [Interactive Tracing Scenario](https://www.envoyproxy.io/try/implementing-metrics-tracing).

Ведение журнала - не единственный способ получить представление о работе с Envoy Proxy. В него встроены расширенные возможности трассировки и метрик. Вы можете узнать больше в [документации по трассировке](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/tracing) или через [Сценарий интерактивной трассировки](https://www.envoyproxy.io/try/implementing-metrics-tracing).



## Шаг 7 - Запуск

You have now translated the configuration from NGINX to Envoy Proxy. The final step is to launch the Envoy Proxy instance to test it.

Теперь вы перевели конфигурацию с NGINX на Envoy Proxy. Последний шаг - запустить экземпляр Envoy Proxy для его тестирования.

#### Run As User

#### Запуск от пользователя

At the top of the NGINX configuration, the line *user  www www;* indicates to run NGINX as a low privileged user to increase security.

В верхней части конфигурации NGINX строка *user www www;* указывает на запуск NGINX в качестве пользователя с низким уровнем привилегий для повышения безопасности.

Envoy Proxy takes a cloud native approach to managing who the process owner is. When we launch Envoy Proxy via a container we can specify a low privileged user.

Envoy Proxy использует облачный подход к управлению тем, кто является владельцем процесса. Когда мы запускаем Envoy Proxy через контейнер, мы можем указать пользователя с низким уровнем привилегий.

#### Запуск Envoy Proxy

The command below will launch Envoy Proxy via a Docker Container on the host. The command exposes Envoy to listen to incoming requests on port 80. However, as specified in the listener configuration, Envoy Proxy is listening to incoming traffic on port 8080. This allows the process to run as a low privileged user.

Команда ниже запустит Envoy Proxy через Docker-контейнер на хосте. Эта команда предоставляет Envoy возможность прослушивать входящие запросы через порт 80. Однако, как указано в конфигурации прослушивателя, Envoy Proxy прослушивает входящий трафик через порт 8080. Это позволяет процессу запускаться как пользователь с низкими привилегиями.

```bash
docker run --name proxy1 -p 80:8080 --user 1000:1000 -v /root/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy
```

#### Тестирование

With the Proxy initiated, tests can now be made and processed. The following cURL command issues a request with the host header defined in the proxy configuration.

С запущенным прокси, тесты теперь могут быть сделаны и обработаны. Следующая команда cURL выдает запрос с заголовком узла, определенным в конфигурации прокси.

```bash
curl -H "Host: one.example.com" localhost -i
```

The HTTP request will result in a *503* error. This is because the upstream connections are not running and they are unavailable. As such, Envoy Proxy has no available target destinations for the request. The following command will launch a series of HTTP services that match the configuration defined for Envoy.

Запрос HTTP приведет к ошибке *503*. Это связано с тем, что восходящие соединения не работают и они недоступны. Таким образом, у Envoy Proxy нет доступных целевых адресатов для запроса. Следующая команда запустит серию HTTP-сервисов, которые соответствуют конфигурации, определенной для Envoy.

```bash
docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server;
```

With the services available, Envoy can successfully proxy traffic to the target destination.

С доступными услугами Envoy может успешно проксировать трафик к месту назначения.

```bash
curl -H "Host: one.example.com" localhost -i
```

You should see a response indicating which docker container processed the request. Within the Envoy Proxy logs, you should also see the access line outputted.

Вы должны увидеть ответ, указывающий, какой контейнер Docker обработал запрос. В журналах Envoy Proxy вы также должны увидеть выведенную строку доступа.

#### Дополнительные заголовки ответа HTTP (HTTP Response)

Within the response headers of the valid request you will see additional HTTP headers. The header displays the time that upstream host spent processing the request. It is expressed in milliseconds. This is useful if the client wants to determine service time compared to network latency.

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

### Установка Envoy на CentOS 7

1. **Установите утилиту yum-config-manager.**

   ```sh
   $ sudo yum install -y yum-utils
   ```

2. **Добавить конфигурацию репозитория yum.**

   ```sh
   $ sudo yum-config-manager --add-repo https://getenvoy.io/linux/centos/tetrate-getenvoy.repo
   ```

   Nightly packages can be enabled using the `--enable` flag. To disable again, use the `--disable` flag.

   Ночные пакеты могут быть включены с помощью флага `--enable`. Чтобы снова отключить, используйте флаг `--disable`.

   ```sh
   $ sudo yum-config-manager --enable tetrate-getenvoy-nightly
   ```

3. **Install Envoy binary.**

   **Установите бинарный пакет Envoy.**

   ```sh
   $ sudo yum install -y getenvoy-envoy
   ```

4. **Убедитесь, что посланник установлен.**

   ```sh
   $ envoy --version
   ```

   

По умолчанию в rpm отсутствует systemd service конфиг. Поэтому я создал [issue](https://github.com/envoyproxy/envoy/issues/8387)

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





