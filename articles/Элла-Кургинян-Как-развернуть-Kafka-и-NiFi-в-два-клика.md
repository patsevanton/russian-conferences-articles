# Элла Кургинян – Как развернуть Kafka и NiFi в два клика

Я работаю в компании Arenadata. Занимаюсь разработкой одного из наших продуктовых подразделений. В частности, занимаюсь Arenadata streaming. 

![](https://habrastorage.org/webt/v9/z0/r-/v9z0r-0lv2bbfo8w1kvw3crasd0.jpeg)

И сегодня я хочу немножко рассказать про то, что это такое и чем оно вам может быть полезным. 

![](https://habrastorage.org/webt/pp/y4/ac/ppy4ach3l6oaosbqvp4yqlf1y0u.jpeg)

Сначала немного расскажу о компании и о том, кто мы такие:

- Мы появились в 2015-ом году. 
- Мы – молодая компания, которая быстро растет. За последние полтора года мы выросли примерно в 3 раза. 
- Мы разрабатываем свои продукты на базе таких open source проектов, как Apache Kafka, Greenplum, NiFi, Hadoop и ClickHouse. 
- Мы являемся контрибьюторами Greenplum, ClickHouse и Hadoop.
- Также мы являемся членами EDP и Linux Foundation.

![](https://habrastorage.org/webt/3u/xa/-h/3uxa-hbmwgf2ko1twmimahl16tc.jpeg)

И хочу рассказать немного о тех проблемах, с которыми мы и наши заказчики сталкиваемся при построении дата-платформ и стриминговых платформ в частности. 

- Когда мы начинаем строить свою дата-платформу, мы понимаем, что она должна закрывать в себе определенный круг задач. Но мы растем, компания растет, соответственно, дата-платформа должна успевать удовлетворять потребностям бизнеса. 

- Про то, как появляются новые потоки, появляются новые компоненты и как это все деплоить и о том, как ландшафт компании усложняется рассказали коллеги из Тинькоффа. И я не буду акцентировать внимание на этом. 

  Но скажу, что со временем платформу становится сложнее и сложнее поддерживать. И приходится поддерживать несколько сред, в частности dev, test, prod. Для того чтобы поддерживать подобного рода платформу, нужны ресурсы. 

- Нужно такую платформу еще и обновлять. Обновили Kafka, обновили Flink, забыли или не захотели обновить NiFi и все flow разъехались. Почему? Потому что текущая версия NiFi не умеет считывать из Kafka, старый процессор перестал работать с новой Kafka. Это типичная ситуация, поэтому совместимость компонентов для нас очень важна, особенно при обновлении. 
- Стоимость такой платформы. Приходится держать, как минимум, 3 среды. И, естественно, это дорого. Но было бы неплохо, если часть каких-то ресурсов можно было бы расходовать временно. Например, вы понимаете, что dev можно держать на cloud. Почему нет? И тогда стоимость платформы хоть как-то, но снизится. 
- И последняя проблема – это мониторинг, точнее разрозненный мониторинг. Инфраструктурные системные метрики в одном месте у вас хранятся, сервисные и метрики приложения во втором месте, бизнес-критичные метрики в третьем месте. И порой очень сложно понять, почему моргнула та или иная бизнес-важная метрика. 

**![](https://habrastorage.org/webt/lh/0a/p4/lh0ap4343kvhid6d-fubtva0ypi.jpeg)**

И решение – это Cluster Manager.

Cluster Manager – это наша собственная разработка. Она у нас лицензируется под лицензией ASF 2, поэтому здесь нет никакого vendor lock. 

Как мы достигли решения этих проблем?

**![](https://habrastorage.org/webt/mx/zc/lb/mxzclbn62my8i2zhe7-50ojs_lm.jpeg)**

Мы разделили инфраструктуру и уровень приложений на два независимых слоя. И за счет этого снизились <font color="red">касты</font> на DevOps, снизились касты на саму платформу. 

Как вы видите, Cluster Manager – это для нас единая точка для всех продуктов. Никто не мешает вам сделать свой продукт на базе Cluster Manager, т. к. у нас SDK открыт, API открыт. И позже я расскажу более подробно, как это сделать. 

![](https://habrastorage.org/webt/jb/nm/q9/jbnmq9snaigskq13dwo0hisu-ji.jpeg)

Немного о стриминге. Что собой представляет стриминг? Это как раз интерфейс Cluster Manager. Это Kafka-Manager, REST Proxy, KSQL, NiFi, NiFi registry и ZooKeeper. 

![](https://habrastorage.org/webt/-q/5m/ec/-q5mec80mo3ig3k2ulo-wym3u_8.jpeg)

Какие возможности мы предоставляем?

- Только open source.
- У нас своя сборка пакетов. Мы собираем, тестируем, выкладываем в открытый репозиторий. У нас репозиторий открыт. Пакетами можно пользоваться, даже если вы не планируете пользоваться ADCM. Я знаю, что сейчас многие vendors закрыли свои репозитории, а у нас все открыто. 
- Более того, в своих продуктах, в частности в стриминге, мы стараемся делать так, чтобы версии продуктов, версии компонентов были последними, но в то же время совместимыми. 

С точки зрения функционала мы предоставляем:

- Деплоймент и менеджмент кластеров; 
- Расширение/ сжатие кластера на лету. В случае Kafka это возможно сделать. Допустим, вы поставили 5 брокеров, потом вы захотели расширить кластер и добавляете 6-ой. И для вашего приклада расширение происходит на лету. И приклад не чувствует того, что Kafka под ним, условно, обновилась;
- Обновление кластера без простоя. Да, если это возможно. Обновляете кластер, обновляете сразу все компоненты, получаете совместимые между собой компоненты и никаких проблем с точки зрения интеграции; 
- Интеграция сервисов из различных кластеров между собой. Например, вы развернули с помощью Cluster Manager Hadoop, потом вы решили с помощью Cluster Manager развернуть еще Kafka. И все мы знаем, что как в Hadoop, так и в Kafka нужен ZooKeeper. KIP-500 говорит, что Kafka сейчас уже ZooKeeper не нужен, но мы по-прежнему деплоим Kafka вместе с ZooKeeper. Соответственно, вы можете для своего dev-окружения сделать так, чтобы у вас ZooKeeper как бы в разных кластерах мог импортироваться с одного кластера в другой. И тогда вам не нужно делать на каждый кластер свой отдельный ZooKeeper. В случае dev это очень удобно. 

![](https://habrastorage.org/webt/ia/z6/ul/iaz6ulbhynt1xsrwum2nephkgk4.jpeg)

Как это устроено? 

Arenadata Cluster Manager представляет собой докерский контейнер, поэтому вы можете поставить его везде, где у вас есть и может быть докер. С точки зрения самого продукта бэкенд написан на Go, Python и Django. 

- Сами по себе продукты: Hadoop, ClickHouse, Greenplum, Streaming – это набор сервисов и действий для управления ими. 
- Как таковые действия (Action) – это Ansible-Playbook, т. е. мы развертываем и управляем своими сервисами с помощью Ansible. Поэтому для вас здесь все прозрачно. Вы можете открыть эти Playbook, вы можете внести свои изменения, если вам нужно. И дальше работать с этим. Все свои приложения, которые вы хотите раскатывать Cluster Manager, вы также можете раскатывать, написав для него простой Ansible-Playbook.
- В качестве агента, отсылающего метрики, у нас выбран Diamond. Он отсылает метрики в Graphite. 
- И Grafana обеспечивает визуализацию самих метрик и в ней можно настроить оповещение. 

![](https://habrastorage.org/webt/vu/vj/uu/vuvjuuhgt8_0nwqe_avawnvvlog.jpeg)

Как с этим работать?

ADCM выложен в Docker Hub. Можно просто скачать его. 3 команды: 

- ```
  docker pull arenadata/adcm:latest
  ```

- ```
  docker create --name adcm -p 8000:8000 -v /opt/adcm:/adcm/data arenadata/adcm:latest
  ```

- ```
  docker run adcm
  ```

Документация у нас есть. Как вы здесь видите, Software Kit тоже у нас открыт. Все очень просто делается.

![](https://habrastorage.org/webt/1y/eo/uj/1yeoujwsja842-bn7q28fe8xlem.jpeg)

Мы запустили наш контейнер. Приветственная страница ADCM. 

**![](https://habrastorage.org/webt/xg/md/l6/xgmdl6attgxttpxj6xyv7uvpq1o.jpeg)**

Далее здесь представлены те bundle, которые я уже скачала и загрузила. Все bundle у нас тоже поставляются в открытом виде. Они доступны на сайте, просто берете и скачиваете. 

Вот здесь у нас Yandex.Cloud Bundle. Т. е. этот bundle позволяет вам создавать хосты на Яндекс.Облаке. 

Также у нас есть другие инфраструктурные bundle, но чаще всего используют iSH Bundle. 

Мониторинг здесь у нас есть. 

И несколько версий самого стриминга. 

![](https://habrastorage.org/webt/d7/cg/pi/d7cgpi2hbaxebvmr0zzcifwiclu.jpeg)

Дальше переходим на вкладку с кластером. Создаем кластер нужной нам версии.

![](https://habrastorage.org/webt/-b/-c/_w/-b-c_wlrs8rvxn1vk-w2vcrkybs.jpeg)

Здесь настройки самого репозитория, где лежат все наши пакеты. 

![](https://habrastorage.org/webt/7x/az/hp/7xazhpk2d9ihrlyhnvlwzm8hieg.jpeg)

Дальше мы добавляем хосты. 

![](https://habrastorage.org/webt/hn/8c/z9/hn8cz9p_95otpmqwnusy99ub6-e.jpeg)

У хостов тоже есть свои настройки: ядра, память, диски, образы, которые мы хотим использовать. 

![](https://habrastorage.org/webt/6e/pr/tq/6eprtqe2rjhwbpzka9ccoe3npiy.jpeg)

И дальше накидываем сервисы, которые нам нужны. 

![](https://habrastorage.org/webt/zi/h0/5j/zih05jnkycztabcbttpcunmrzvw.jpeg)

Выбрали сервисы. Их можно настраивать, вносить изменения или добавлять свои кастомные параметры. 

![](https://habrastorage.org/webt/qk/wp/sb/qkwpsbpvhomyjuaaulajpj_l1to.jpeg)

Раскидываем сервис по хостам и нажимаем на кнопку «Install». 

![](https://habrastorage.org/webt/bz/s9/oz/bzs9oz-zzs8jmk20kxu-bf-ainu.jpeg)

Здесь нам предлагается воспользоваться настройками хостов. Например, можно SeLinux выключить или Firewalld.

![](https://habrastorage.org/webt/gj/i2/xi/gji2xitkajoqnoqhelvem4j6cys.jpeg)

Все, кластер инсталлирован. 

![](https://habrastorage.org/webt/eu/dp/22/eudp22mvxjp3ehyufkdyblfjoyy.jpeg)

Что дальше можно делать с этим кластером? User cases.

Например, NiFi, как нам кажется, может выступать продюсером для Kafka. NiFi’йном собираем данных с различных источников, кладем в Kafka.

![](https://habrastorage.org/webt/pg/3l/lf/pg3llflfxqhcf3giip4g8olzqwe.jpeg)

Второй кейс. Расскажу немножко про наш опыт с одним проектом, который мы внедряли заказчикам. 

Данные лежали в Kafka. В первом случае мы из Kafka забирали в NiFi. С помощью NiFi формировали витрину ADS-слоя, а потом из ADS-слоя с помощью оркестрации через NiFi уже строили DDS-слой. 

![](https://habrastorage.org/webt/i8/o2/fq/i8o2fqyginbecvd2rnyuuxraquw.jpeg)

Здесь еще может быть много других сложных трансформаций. Можно еще подключить MiNiFi. В общем, много интересного можно построить на базе NiFi, Kafka и MiNiFi.

Roadmap про то, как мы видим развитие нашего продукта в дальнейшем. 

То, о чем я рассказала, это на самом деле уже сейчас доступно. Это 1.4.11 версия стриминга.

В следующей версии мы уже ведем работы по добавлению MiNiFi в наш стек. Мы хотим сделать репликацию между кластерами Kafka. Нам кажется, что это очень интересная идея. Более того, мы выполняем много различных миграций. И этот инструмент нам, как vendor’у, очень полезен. 

А дальше мы начнем автоматизировать вещи, связанные с безопасностью. Хотим сделать поддержку SSL, SASL, киберизиновые кластера, интеграцию с Ranger, Knox и т. д.

Если у вас есть какие-то идеи по тому, как вы видите развитие подобного рода продуктов, а для нас очень важно делать то, что будет нравится не только нам, то пишите. 

И в конце мне бы хотелось немножко продемонстрировать то, как это работает. Мне кажется, видео лучше дополняет мои слова. 

Демо начинается с 14.30 секунд.

Например, сейчас у нас есть кластер, в котором есть Kafka, NiFi. Мы видим, что здесь есть flow. И мы хотим обновить этот кластер до более поздней версии. Скачиваем новый bundle. Загружаем его. И далее мы видим, что нам предлагают обновить этот bundle.

Обновление состоит из двух этапов. Сначала обновляются конфиги, потом обновляются сами пакеты. Но мы видим, что обновление у нас не прошло. Почему? Нам предлагается установить NiFi, потому что NiFi на лету не обновишь. 

Устанавливаем NiFi. И делаем вторую попытку обновления. И видим, что у нас уже проходят какие-то чеки. Обновились конфиги ZooKeeper, обновились конфиги Kafka, NiFi.

И теперь мы запускаем обновление самих пакетов. Проверяем, что все нужные нам пакеты доступны в репозитории. Видим, что ZooKeeper обновился, а также Kafka, NiFi.

И теперь идем проверять то, что мы обновили. Видим, что с NiFi все хорошо, ничего плохого не происходит. 

И теперь мы хотим расширить этот кластер. 

Вы обратили внимание, что кластер в текущий момент работает в режиме standalone. 

Мы понимаем, что это не production, мы хотим кластеризованный NiFi. Соответственно, мы создаем хосты на Яндекс.Облаке. Вносим нужные нам настройки, делаем нужную конфигурацию хостов. Создали один хост для NiFi, второй хост для Kafka. 

И теперь идем расширять NiFi. Мы видим, что у нас появился новый компонент – NiFi-Registry. Мы его тоже устанавливаем. После того, как мы его установим, он у нас автоматически добавится в NiFi. 

И идем устанавливать Kafka-manager. В Kafka-manager мы будем смотреть на брокеров. Мы видим, что некоторые этапы expand уже прошли. NiFi установился, NiFi-Registry тоже. 

Заходим в NiFi-Registry. Здесь мы видим, что он поднялся. Заходим в NiFi и видим, что он кластеризовался. 

Теперь мы идем смотреть, что же происходит у нас с Kafka. Заходим в Kafka-manager. Здесь мы видим, что у нас добавлен наш кластер. И у нас есть один топик. И у нас есть два брокера. И так расширяем кластер NiFi.

Добавляем брокер на новый хост. 

Более того, если кому-то не нравится так играться с интерфейсом ADCM, то у нас есть SDK и можно все это делать через код. Условно, можно разворачивать и плодить среды с помощью клиентского кода.

Мы видим, что у нас новый брокер присоединился к нашему кластеру. И жизнь хороша и прекрасно. Всего 6-8 минут и у вас новый кластер. Управлять этим легко и удобно. 

![](https://habrastorage.org/webt/th/s-/o2/ths-o2d5fijhbvxzhjmaa76dcgk.jpeg)

У меня все!

Мы доступны на всех информационных ресурсах. Заходите на наш сайт Arenadata.io. Скачивайте bundle, пользуйтесь ими. Мы за open source, мы ничего не закрываем. 

Все, спасибо!

**Вопросы**

*Хороший доклад! Мне понравился! Почему* *Diamond, Graphite,* *а* *не* *Prometheus?*  

Хороший вопрос. Так исторически сложилось. Но скажу честно, что у нас много было в компании холиваров по этому поводу. Но это решение было принято. И мы живем теперь с этим.

*Т. е.* *Prometheus* *нет в планах?*

Мы думаем о том, чтобы Graphite заменить на Prometheus. Мы каждый раз об этом думаем. Но чтобы нам какую-то боль решить, нам нужно понимать, что кто-то с другой стороны готов на это. Т. е. мы на данный момент сделали стек, основываясь на тех компонентах и решениях, которые мы хорошо знаем. С Prometheus нам нужно хорошо повозиться. И у нас есть в планах изменить свой мониторинг. 

Более того, мы думаем, чтобы сделать мониторинг таким образом, чтобы его можно было интегрировать с вашим мониторингом. Если вы большая компания, то, скорее всего, у вас уже есть свой мониторинг и вы все мониторите. И у нас, безусловно, есть такие планы по поводу именно интеграции с мониторингом заказчика. 

Можно вам вопрос? Почему Prometheus? Может быть, вы нам поможете определиться.

*У нас сейчас очень большие проблемы с* *Diamond*. *Это отдельная тема. И у нас* *Graphite* *не справляется в нашем случае. Мы сейчас смотрим в сторону* *Prometheus*. *У нас есть* *Graphite*, *у нас он используется, но с ним проблемы.*

Мы вас понимаем. 

*Спасибо за доклад! Меня впечатлило демо, где вы создавали новые кластера. Вопрос у меня про обновление. Дело в том, что обновление – это не совсем тривиальная операция. Нужно знать то, что сейчас установлено. Нужно уметь протестировать* *Ansible**-**Playbook* *относительно той версии, что уже установлена, относительно того кода, что мы уже написали в* *playbook*, *относительно той версии, что мы хотим поставить. Как вы тестируете связку трех этих сущностей?*

*И второй вопрос про* *Ansible*. *Дело в том, что в* *Ansible* *есть очень неприятные спецэффекты. Если у нас есть группа, которую мы хотим обновить, и есть* *playbook*, *по которому мы эту группу обновляем, то у* *Ansible* *есть такая особенность, что каждый стек внутри* *Playbook* *для каждого хоста внутри группы он выполняет последовательно. И если мы хотим обновить* *Kafka*,*то мы можем стопнуть все брокеры и всё станет колом*. *Ansible* *здесь не очень подходящий инструмент. Как вы эти проблемы решаете?*

Я поняла. Давайте с конца начну по поводу того, что нельзя стопать одновременно Kafka. Безусловно, это так. Но у того же Ansible вы можете playbook запускать с опцией serial 1. И тогда у вас будет происходить не одновременный рестарт всех брокеров, а поочередный. Т. е. мы, когда обновляем и рестартим наши сервисы, в частности я говорю про Kafka и ZooKeeper, делаем это по одному. Никаких спецэффектов не возникает. Более того то, что живет поверх Kafka с этим справляется. Это для приклада происходит незаметно. Это первое.

Второй вопрос был про обновление. Естественно, разрабатывая Ansible-Playbook мы их тестируем. У нас есть тесты, которые мы ручным образом запускаем. У нас есть тесты, которые интеграционные. У нас есть тесты и на обновление тоже. На все операции, которые мы производим, написаны тесты. И на обновление в том числе. 

В случае Arenadata streaming существует пока только два обновления. В каком смысле? Переход с одного стека на другой. Видели, у нас есть стеки, которые базируются 1.3 и стеки, которые 1.4? Мы прекрасно понимаем то, как работал сервис в определенной предыдущей версии и то как работает в следующей версии. И, естественно, разрабатывая playbook мы учитываем это все. Таких ситуаций в данный момент не возникало. 

Возможно, потому что версии компонентов, которые у нас существовали и существуют, не сильно изменяются. Например, если бы обновляли Kafka с версии 0.10 на 2.0 или 2.1, тогда действительно была бы проблема. Когда мы начинали разработку, первый Streaming был вообще на базе Ambar. Это были версии 1.0 и 1.1. И Kafka была на версии 1.0. И переход на версию 2.0 был легким. Т. е. сложной процедуры обновления не было. 

С другой стороны, мы понимаем, что, обновляя те же самые компоненты, мы гарантируем только работу самого сервиса. Мы, к сожалению, не можем гарантировать, что ваш приклад поверх этого сервиса будет так же работать. Потому что методы в API меняются, какие-то классы уходят. Под это дело написан определенный гайд. Но то, что в этом гайде написано с точки зрения пошагового обновления самого сервиса, мы это все гарантируем в рамках своих проектов. 

*Спасибо за доклад! Я немножко не в теме, поэтому вопрос может странным показаться. По функционалу вопрос. Я так понял, что это разворачивание сервиса довольно типовых. Чем радикально, кроме интерфейса, это лучше связки* *Kubernetes* *и* *Helm*? *На мой взгляд эта связка покрывает большую часть, о чем вы говорили. Вы знакомы с этими вещами?*

Я поняла, о чем вы говорите. К сожалению, на данном этапе я подобными инфраструктурными вещами не занимаюсь. Но в обозримом будущем мы планируем работать с контейнерами. Более того, у нас уже есть сервисы, которые запакованы в контейнер. Понятно, что это сам ADCM. И у нас есть еще Kafka, Greenplum connector. Но он не open source. И он пакуется в докер. Т. е. историю с контейнеризацией мы уже начали, но конкретно на ваш вопрос, я к сожалению, ответить не могу, потому что не обладаю этими знаниями, чтобы сравнить два этих решения. 

