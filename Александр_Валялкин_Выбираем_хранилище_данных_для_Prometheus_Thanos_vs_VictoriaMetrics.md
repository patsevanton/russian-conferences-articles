**[Prometheus](https://github.com/prometheus)** – система мониторинга различных систем и сервисов, с помощью которой системные администраторы могут собирать информацию о текущих параметрах, настраивать хуки для получения иных уведомлений (например, предупреждение, когда возникают какие-либо проблемы).

В докладе будет сравнение [Thanos](https://github.com/thanos-io/thanos) и [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics) - проектов для долгосрочного хранения метрик Prometheus.

![](https://habrastorage.org/webt/vm/ha/hd/vmhahdu2edj4dsc6o3lvzaf-noo.png)

https://www.youtube.com/watch?v=HyOXAdQE0Pk


![](https://habrastorage.org/webt/pg/3x/hc/pg3xhcm_wiipwhdxavyd6e1q8sm.png)

![](https://habrastorage.org/webt/ly/t4/ly/lyt4lyh_ig976fd0yrj_kxgto50.png)

![](https://habrastorage.org/webt/b4/jo/gz/b4jogziezacw5myfzttwx7-c26g.png)

![](https://habrastorage.org/webt/tj/hb/2x/tjhb2xocarftzjxirb0mmdoyy7u.png)

![](https://habrastorage.org/webt/op/ni/ao/opniaodpqcis34bw3f4-twdnsic.png)

Сначала расскажу про prometheus. Prometheus это система мониторинга, которая собирает метрики каких-то target и сохраняет их в local storage. Prometheus умеет записывать метрики в remote storage, умеет собирать алерты и recording rules. 

![](https://habrastorage.org/webt/kg/y1/wf/kgy1wf1db2occcivabh3lqppvw4.png)

![](https://habrastorage.org/webt/ci/iy/l6/ciiyl6uct2oq2r4ngiwcxjo5ksg.png)

![](https://habrastorage.org/webt/ge/8y/xe/ge8yxeqvzh2ov-m5mjgbajdb9zm.png)

![](https://habrastorage.org/webt/dl/ji/yn/dljiynnywlw3fcjl7oc3m7lgstw.png)

### Ограничения Prometheus:

- У него нет global query view. Это когда у вас есть несколько независимых экземпляров prometheus. Они собирают метрики. И вы хотите сделать запрос поверх всех этих метрик, собранных с разных экземпляров prometheus. Prometheus это не позволяет.
- У prometheus производительность ограничено только одним сервером. Prometheus автоматически не может масштабироваться на несколько серверов. Вы только можете вручную разделить ваши target, которые делают scrape между этими prometheus.
- Объем метрик в prometheus ограничен только одним сервером по той же причине что он автоматически не может масштабироваться на несколько серверов.
- В prometheus не так-то просто организовать сохранность данных.

![](https://habrastorage.org/webt/t3/wi/lf/t3wilf2sbn7ndupjqbe1wgxqrzw.png)

### Решения этих проблем/задач?

Решения такие:

- [Thanos](https://github.com/thanos-io/thanos)
- [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics)
- [Uber M3](https://github.com/m3db)
- [Cortex](https://github.com/cortexproject/cortex)

Все эти решения для remote storage prometheus. Они решают проблему remote storage с предыдущего слайда по разному. В данной презентации я расскажу только про первые два решения: [Thanos](https://github.com/thanos-io/thanos) и [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics).

![](https://habrastorage.org/webt/k2/dp/tb/k2dptbckze9plps6m71ycuktv3y.png)

![](https://habrastorage.org/webt/dc/kr/9x/dckr9xddxfufhzctkcfqp6tr3d4.png)

Впервые информация про [Thanos](https://github.com/thanos-io/thanos) появилась по этой ссылке. Там описана архитектура [Thanos](https://github.com/thanos-io/thanos) и как он работает. 

![](https://habrastorage.org/webt/wg/bq/-b/wgbq-b3pwr1emeewan-q5hszprs.png)

Thanos берет данные, которая сохранил prometheus на локальный диск и копируют их в S3 либо в [GCS](https://cloud.google.com/storage/) либов другой object storage.

![](https://habrastorage.org/webt/p7/4g/rj/p74grj159fi9xb58vq-hphcrqyi.png)

Таким образом Thanos обеспечивает global query view. Вы можете запрашивать данные сохраненное в object storage или в S3 c нескольких prometheus.

![](https://habrastorage.org/webt/aj/3u/wi/aj3uwi8dd4wgy-fyogditn--kek.png)

Thanos поддерживает PromQL и prometheus API.

![](https://habrastorage.org/webt/sr/t7/ql/srt7qlyoruhbxr5wglfuyhi9jp4.png)

Thanos использует код prometheus, он использовует storage prometheus и многие части кода переиспользуются Thanos.

![](https://habrastorage.org/webt/3y/ws/ik/3ywsiklzziqimz1aig7uzieblqw.png)

Thanos на самом деле разрабатывает те же разработчики, что и prometheus.

![](https://habrastorage.org/webt/bm/gr/bh/bmgrbh1y5arap-nilpvleilsaei.png)

Про [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics). Вот ссылка где мы впервые рассказали что [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics). VictoriaMetrics - best remote storate for prometheus. 

![](https://habrastorage.org/webt/z9/q2/wv/z9q2wvhshsoqb1ron0jx47djiwc.png)

VictoriaMetrics получает данные с нескольких prometheus по remote write API протоколу, поддерживаемым prometheus.

![](https://habrastorage.org/webt/pl/xl/ad/plxladg1rdvyudc0wzs5oc2ljsa.png)

Также обеспечивает global query view, так как несколько prometheus пишут в VictoriaMetrics. И соответственно вы можете сделать запросы по этомим всем данным. 

![](https://habrastorage.org/webt/9l/cp/aw/9lcpawp68_pq2mrhzya9okprroy.png)

VictoriaMetrics также поддерживает как и Thanos - PromQL и Prometheus querying API.

![](https://habrastorage.org/webt/d1/t1/ek/d1t1ek9rbq_ftlasl48scoriyq0.png)

В отличие от Thanos, исходный код VictoriaMetrics написан с нуля и оптимизирован по скорости и потребляемым ресурсам. 

![](https://habrastorage.org/webt/ps/t6/gu/pst6guuoseu2dicx9tra1bvjhry.png)

VictoriaMetrics в отличие от Thanos масштабируется как вертикально так и горизонтально. То есть Single-node верстия, которая масштабируется вертикально. Вы можете начать с одного процессора и 1 ГБ памяти и постепенно расти до 100 процессоров и 1ТБ памяти. VictoriaMetrics умеет использовать все эти ресурсы. Ее производительность вырастет примерно в 100 раз по сравнению с 1-ядерной системой. 

![](https://habrastorage.org/webt/tb/s7/t4/tbs7t4unnruaytt7duwambgofne.png)

История Thanos началась в ноябре 2017 года, когда появился первый публичный коммит. До этого Thanos разрабатывался внутри компании [improbable.io](https://improbable.io). 

![](https://habrastorage.org/webt/ik/g7/dg/ikg7dgnmbyiph40zo7v7kyhxrdy.png)

В июне 2019 года был знаковый релиз 0.5.0, в котором убрали [gossip](https://en.wikipedia.org/wiki/Gossip_protocol) протокол. Eго убрали из Thanos, потому что он себя показал не с лучшей стороны. Часто кластер Thanos неправильно работал, неправильно подключались к нему ноды из-за [gossip](https://en.wikipedia.org/wiki/Gossip_protocol) протокола. Поэтому решили его оттуда убрать. Я считаю что правильно сделали. 

![](https://habrastorage.org/webt/5y/bu/cu/5ybucuovkh85tqwsrv3cjre-zmu.png)

В том же июне 2019 года они отправили проползал в [Cloud Native Computing Foundation](https://www.cncf.io/).

![](https://habrastorage.org/webt/16/rz/x_/16rzx_bswddquoqw4cmw9txobxa.png)

И через пару месяцев Thanos приняли в [Cloud Native Computing Foundation](https://www.cncf.io/), в которую входит Kubernetes и другие популярное проекты.

![](https://habrastorage.org/webt/9c/i2/ic/9ci2ictg_n0xtgdcxtklghw4n2w.png)

В январе 2018 года началась разработка VictoriaMetrics.

![](https://habrastorage.org/webt/wi/li/co/wiliconuw5mwhyzizzthqk8a2og.png)

В сентябре 2018 я впервые публично упомянул про VictoriaMetrics.

![](https://habrastorage.org/webt/gd/th/ka/gdthkaxubi8jzg8gedilcotjdje.png)

В декабре 2018 года опубликовали Single-node версию.

![](https://habrastorage.org/webt/ui/r7/23/uir723wmd-l1dwxds6katr8cfco.png)

В мае 2019 были опубликованы исходники как Single-node, так и кластерной версии. 

![](https://habrastorage.org/webt/a7/tk/ug/a7tkughz1ut9dmlfb6ued1irbh8.png)

В июне 2019 также как Thanos, мы подали заявку CNCF foundation под номером 255. Мы подали заявку на один день раньше чем подал заявку Thanos. 

![](https://habrastorage.org/webt/pk/dm/zr/pkdmzr2xn880osldinx6sb10-ai.png)

Но, к сожалению, нас до сих пор не приняли туда. Нужна помощь коммьюнити.

![](https://habrastorage.org/webt/mw/2w/we/mw2wwenfu8tecrrkyrkx-8dhf-8.png)

Мы рассмотрим самые главные слайды про архитектуру Thanos и VictoriaMetrics.

![](https://habrastorage.org/webt/1w/by/96/1wby96ev7cqhx7-psbem1b-bf-i.png)

Начнем с Thanos. Желтые компоненты - это компоненты prometheus. Все остальное это компоненты Thanos. Начнем с самого главного компонента. Thanos side-car - это компонент, который устанавливается рядом с каждым prometheus. Он занимается тем, что загружает данные prometheus локального storage в S3. 

Есть еще такой компонент как Thanos Store Gateway, который умеет считывать эти данные c Cloud storage динамически на входящие запросы от Thanos query. Thanos query - компонент реализует promQL и prometheus API. То есть снаружи он выглядит как prometheus. Принимает запрос promql, отправляет его в Thanos Store Gateway, Thanos Store Gateway достает нужные данные из Object Storage S3, отправляет их назад.

Но у нас в Object Storage хранятся данные без последний двух часов из-за особенности реализации Thanos side-car, который не может закачать последние два часа в Object Storage S3, так как это два часа еще prometheus не создал из них файловы в локальном Object Storage.

Как же это решили обойти? Thanos query кроме запростов в Thanos Store Gateway отправляет параллельно запросы еще в каждые Thanos side-car, которые находятся рядом с prometheus. 

А Thanos side-car свою очередь проксирует этот запрос дальше в Thanos side-car и достает данные за последние два часа.

Кроме этих компонент, есть еще опциональный компонент, без которого Thanos будет плохо себя чувствовать. Это Thanos Compact, который занимается слиянием мелких файлов на Object Storage в более крупные файлы, которые были загружены отсюда с Thanos side-car. Thanos side-car выгружает туда файлы размером стандартного два часа. 

Эти файлы, если их не сливать более крупные файлы, то их количество может вырастить очень существенно. Чем больше таких файлов, тем больше нужно памяти для Thanos Store Gateway, тем больше нужно ресурсов для передачи данных по сети, метаданных. Работа Thanos Store Gateway становится неэффективной. Поэтому нужно обязательно запускать Thanos Compact, который сливает мелкие файлы в более крупные, чтобы таких файлов был меньше и чтобы уменьшить overhead на Thanos Store Gateway.

Есть еще такой компонент как Thanos Rule. Он выполняет prometheus-е rule для alert и может вычислять prometheus-е rule для того чтобы записывать данные снова в Object Storage. И этот компонент Thanos Rule можно использовать не только с Thanos, так как он универсальной, то можно использовать и с VictoriaMetrics.

Вот такая схема простая. 

![](https://habrastorage.org/webt/os/em/y4/osemy4ys_bkpy-ngxj7jg-gdmom.png)



Теперь сравним со схемой VictoriaMetrics.

У VictoriaMetrics есть 2 версии: Single-node и кластерная версия. Single-node работает на одном компьютере. В Single-node нет этих компонентов, просто один бинарник. Этот бинарник на слайде выглядит вот этим квадратом. Все что внутри находится это содержимое бинарного файла для Single-node версии. Вам необязательно про него знать. Просто запускаете бинарник и все у нас работает. 

Кластерная версия сложнее. Внутри нее находится три разных компонента: select, insert и storage. Из названия должно быть понятно чем каждый занимается. Insert компонент принимает данные в разных форматах: из prometheus remote write API, inluxe протокола, graphite протокола и из opentsdb протокола. Insert компонент принимает их, парсит и распределяет между имеющимися storage компонентами, где данные уже сохраняется. Select компонент в свою очередь принимает promql запросы. Он реализует promql и также prometheus API, может быть использован как замена prometheus в Grafana или других prometheus API клиентах. Select принимает promql запрос, парсит его, считывает необходимые данные для выполнение этого запроса из storage нод, процессит эти данные и возвращает ответ.

![](https://habrastorage.org/webt/sb/co/mx/sbcomx2usdrwzbyfgqjubvoichg.png)

Основная основные слайды прошли. 

Сравним сложность установки Thanos и VictoriaMetrics.

![](https://habrastorage.org/webt/hv/gw/r6/hvgwr6nk7yh4nog7rpjac4fn0se.png)

Начнем с Thanos. Перед тем, как начать работать с Thanos нужно создать storage bucket в Object Storage так как S3 или GCS. Чтобы Thanos могу туда данные записывать. 

![](https://habrastorage.org/webt/st/hk/5s/sthk5swss1mkaalb4p3oiebnesq.png)

Потом для каждого Thanos что нужно установить Thanos Sidecar. Но перед этим не забыть отключить data compaction в prometheus. Data compaction похожий компонент Thanos Compact в prometheus, который сжимает данные в Object Storage для того чтобы меньше потреблять ресурсов. Похожий компонент есть в prometheus. Он тоже самое делает с данными, которые пишутся в локальный storage, который периодически сливаются более крупные данные, чтобы уменьшить накладные расходы.

Когда вы устанавливаете Thanos Sidecar к вашим prometheus, вы должны отключить этот data compaction, потому что Thanos Sidecar не умеет нормально работать, когда включен у prometheus data compaction. Это означает что ваш prometheus начинает сохранять данные блоками по два часа и перестает сливать в более крупные блоки. Соответственно, если вы делаете запросы, которые превышают длительность последние два часа, то они будут не настолько эффективно работать, по сравнению с тем как могли бы работать. этот был включен data compaction.

![](https://habrastorage.org/webt/gd/as/pw/gdaspwys_r63cegoxqovl9bcmie.png)

Поэтому Thanos рекомендует уменьшать data retention в локальном storage до 6-8 часов, чтобы снизить этот overhead большого количества мелких блоков.

После того как вы установили Thanos Sidecar, вы должны для каждого Object Storage Bucket установить два компонента. Это Thanos Compactor и Thanos Store Gateway. 

![](https://habrastorage.org/webt/cw/4f/yk/cw4fyksvnkmp1tcn-clf-2gzx3q.png)

После этого установите Thanos query и настроить его чтобы он умел подключаться ко всем Thanos Store Gateway, который у вас есть, и также умел подключатся ко всем Thanos Sidecar.

Тут может быть небольшая проблемка. 

![](https://habrastorage.org/webt/vf/sc/30/vfsc30z00pvcnupbdym2jztx09k.png)

Вам нужно настроить надежно и защищенное подключение от этих компонентов. И если у вас prometheus находится в разных дата-центрах, либо в разных VPC, то к ним запрещены подключения из вне. Но для работы Thanos query вам нужно как-то настроить подключение туда и вы должны придумать способ.

Если у вас таких дата-центров много, то соответственно снижается надежность всей системы. Так как Thanos query должен постоянно держать подключения ко всем Thanos Sidecar, расположеных в разных дата-центрах. При каждом входящем запросе он будет направлять запросы на Thanos Sidecar. Если соединение прервется, то вы получите либо не полный набор данных, либо получите ответ кластер не работает. 

![](https://habrastorage.org/webt/rb/ka/aw/rbkaawevhardp1d85mhgdejylxg.png)

В VictoriaMetrics немного все проще. В Single-node версии достаточно просто запустить один бинарник и все работает. 

![](https://habrastorage.org/webt/co/ui/ll/couillafhtmobnbutejgs6qvszu.png)

В кластерной на версии достаточно запустить все вышеупомянутые три типа компонентов в любом необходимом вам количестве, либо использовать helm chart для автоматизации запуска компонентов в Kubernetes. Мы еще планируем сделать Kubernetes оператор. Helm chart не покрывает некоторые кейсы и позволяет вам выстрелить ногу. Например, он позволяет уменьшить количество storage node, что приведет к потере данных.

![](https://habrastorage.org/webt/jb/ta/0t/jbta0t7qcko6jov_x1ximqkfni0.png)

После того как вы запустили один бинарник либо кластерную на версию, вам достаточно добавить в конфиг prometheus remote write url чтобы он начал записывать даннык параллельно в локальный storage и в remote storage. Как вы заметили эта настройка должна работать на много надежнее по сравнению с настройкой Thanos. Нам не нужно держать подключение от VictoriaMetrics к prometheus, например от большого количества prometheus в разных дата-центрах, потому что prometheus сами подключаются в VictoriaMetrics. 

![](https://habrastorage.org/webt/u3/3a/if/u33aifuu7myx3vvs0mkbeysfnei.png)

Рассмотрим сопровождение Thanos и VictoriaMetrics.

![](https://habrastorage.org/webt/fq/dt/d2/fqdtd27cjx-yktnyozgxoon2pte.png)

У Thanos нужно следить за Sidecar чтобы они не прекращали загрузку данных в Object Storage. Они могут прекратить эту загрузку данных в вследствие ошибок загрузки, например у вас временно прервалась сетевое соединение с Object Storage, либо Object Storage временно стал недоступен. Thanos Sidecar в этот момент заметит это, сообщит об ошибке, может свалиться и после этого перестать работать. Если вы не будете мониторить, то у вас перестанут передаваться данные в Object Storage. Если пройдет время retention (6-8 часов рекомендованное), то вы будете терять данные, которой не попали в Object Storage.

![](https://habrastorage.org/webt/z7/bm/cx/z7bmcxgwyscnrnnehhtjvuqipho.png)

Thanos сompactor-ы могут перестать работать из-за race-ов (ошибок) с Sidecar. Сompactor-ы берут данные с Object Storage и сливают их в более крупные куски данных. Так как сompactor-ы не синхронизированы с Sidecar-и, то может произойти что Sidecar не успел еще дописать блок, сompactor решает что этот блок полностью записан. Compactor начинает его считывать. Он считывает блок не в полном виде и перестает работать.

![](https://habrastorage.org/webt/_u/c6/hb/_uc6hbgr_kl9l2an5yl0cs9hfba.png)

Store Gateway могут отдавать неконсистентная данные из-за рейсов между сompactor-и sidecar-и. Тут такая же штука, потому что Store Gateway никак не синхронизирован с сompactor-и sidecar-и. Соответственно могут возникать рейсы, когда Store Gateway не видит часть данных, либо видит слишком много данных лишних. 

![](https://habrastorage.org/webt/-y/20/ur/-y20urze7qgwxkrklj8ave_ct3i.png)

Query компонент в Thanos по умолчанию отдает частичный результат, если некоторые sidecar-ы либо Store Gateway не доступны в данный момент. Вы получите часть данных или даже не будет знать что вы получили не все данные. Это он работает так по умолчанию. В похожей ситуации VictoriaMetrics возвращает помеченные данные как частичные. 

![](https://habrastorage.org/webt/2g/op/wp/2gopwpbh41vviugdebarsjuql4w.png)

В отличие от Thanos VictoriaMetrics редко теряет данные, даже если прервалось подключение от prometheus к VictoriaMetrics, то не проблема, так как prometheus продолжается записывать поступающие новые данные в Write Ahead Log, размер которого равен 2 часам. Если вы в течении двух часов восстановите подключение к VictoriaMetrics, то данные не потеряются. Prometheus умеет данные, которые были не записаны, записать после восстановления подключения к VictoriaMetrics.

![](https://habrastorage.org/webt/kg/nl/mv/kgnlmvkqxfagsjhodoybpwzl7am.png)

В отличие от Thanos, который записывают данные только спустя два часа. Последние два часа Thanos не записывать в S3, prometheus реплицирует автоматически данные по remote write протоколу в remote storage соответственно VictoriaMetrics. И вам не страшна потеря local storage в prometheus. Если он вдруг потерял local storage, то вы худшем случае потеряете последние секунды данных, которые не успели записаться в remote storage.

![](https://habrastorage.org/webt/ba/xj/90/baxj90beqmojti9xkubodbkbvdw.png)

Kubernetes автоматически управляет кластером в отличие от Thanos. В Thanos сложно поместить его в один Kubernetes кластер.

Thanos намного сложнее поместить в Kubernetes, чем VictoriaMetrics, можно поместить весь кластер VictoriaMetrics поместить в Kubernetes. 

![](https://habrastorage.org/webt/gz/lb/6l/gzlb6lg2dkrabcovqcohvovd-mk.png)

VictoriaMetrics очень простое обновление на новую версию. Просто останавливаете VictoriaMetrics, обновляете бинарники и запускаете. При остановке через SIGINT сигнал все бинарники VictoriaMetrics делают gracefull shutdown. Они правильно сохраняет нужны данные, правильно закрывают входящие соединения чтобы ничего не потерять. Поэтому вы ничего не потеряете при обновлении. 

![](https://habrastorage.org/webt/qz/yf/8c/qzyf8cvpqlimwabno0zx7v5_rxw.png)

VictoriaMetrics очень просто расширять кластер. Просто добавляете необходимые компоненты и продолжаете работать. 

![](https://habrastorage.org/webt/hl/le/sj/hllesjwiv-f1wosgssd7wp6aclo.png)

Подводные камни и там и там.

![](https://habrastorage.org/webt/xs/ec/b0/xsecb0yy15vimtsl9k9quqplyiw.png)

У Thanos подводные камни. Prometheus должен хранить данные за последние два числа. Если они потеряются, вы их полностью потеряете, так как они еще не успели записаться в Object Storage S3. 

![](https://habrastorage.org/webt/fg/xo/yo/fgxoyo1q8mihcdy7m_yyio_pyum.png)

Store Gateway компонент и сompactor компонент может требовать много памяти для работы с большим Object Storage, если там хранится много мелких файлов. Чем больше количество файлов, тем больше требуется оперативной памяти Store Gateway и сompactor для хранения метаинформации. У Thanos много issue по поводу того, что Store Gateway и сompactor падают при средних объемов записанных данных.

![](https://habrastorage.org/webt/-n/k3/rg/-nk3rgph8ghszyegz1j8jx89bio.png)

Thanos рекламирует что он может скейлиться бесконечно на количество ваших prometheus, производительность скейлиться. На самом деле это неправда. Так как все запросы идут через Query компонент, который должен параллельно опросить все Store Gateway компоненты и все Sidecar компоненты, вытянуть оттуда данные и потом их препроцессить. Очевидно что скорость запросов ограничена самым медленным слабым звеном, самым медленным Store Gateway либо самым медленным Sidecar. 

Эти компоненты могут быть неравномерно нагружены. Например, у вас есть prometheus, который собирает миллионы метрик в секунду. И есть prometheus, в котором собирается тысячи метрик в секунду. И prometheus, в котором собираются миллионы метрик в секунды, он намного сильнее загружает сервер, на котором он работает. Соответственно Sidecar там работает медленнее. И вообще все там медленно работает. И Query компонент будет очень медленно оттуда данные вытягивать. Соответственно производительность вашего всего кластера будет ограничена этим медленным Sidecar.

![](https://habrastorage.org/webt/mg/an/ra/mganracsmhkyz1_1aq014v_6cfc.png)

Я уже про это упоминал. По умолчанию Thanos отдает частичное данные, если некоторые Sidecar и либо Store Gateway недоступны. Например, если у вас Sidecar раскиданы по всему миру в разных дата-центрах, то вероятность разрыва соединения и недоступности компонентов сильно возрастает. Соответственно, в большинстве случаев вы будете получать частичные данные даже не зная об этом.

![](https://habrastorage.org/webt/1t/m2/j7/1tm2j75o5cpscbevahbz4m0wusk.png)

У VictoriaMetrics тоже есть подводные камни. Первый подводный камень это опция, которая ограничивает объём оперативной памяти используемой под кеш VictoriaMetrics. По умолчанию она равна 60% оперативной памяти на машине, где VictoriaMetrics запущена либо 60% ОЗУ пода VictoriaMetrics в kubernetes.

Если неправильно поменять это значение, то можно угробить производительность VictoriaMetrics. Например, установить слишком низкое значение, то данные могут перестать помещаться в кеш VictoriaMetrics. Из-за это ей придется делать лишнюю работу и соответственно же нагружать процессор и диск. Если вы сделаете эту опцию слишком большой, то это повышает, во-первых, вероятность того что VictoriaMetrics будет вылетать с ошибкой out of memory, и, во-вторых, это будет приводить к тому, что в операционной системе будет оставаться очень мало оперативной памяти для файлового кэша. А VictoriaMetrics очень сильно полагается на файлового кэша для производительности. Если его недостаточно, то соответственно сильно увеличиться нагрузка на диск. Поэтому совет: не менять параметр без крайней необходимости.

![](https://habrastorage.org/webt/3q/eo/25/3qeo25x5juqpyquhllrhpt9eh34.png)

Вторая опция. Это retentionPeriod - период, который по умолчанию выставлен в 1 месяц. Это время, в течение которого VictoriaMetrics хранит данные. По истечении этого срока VictoriaMetrics данные удаляет.

Многие запускают VictoriaMetrics без этого параметра, записывают данные в течение месяца. А потом спрашивают: почему данные пропали за предыдущий месяц? Потому что retentionPeriod по умолчанию равен 1 месяц. Про этому нужно знать и устанавливать правильный retentionPeriod. 

![](https://habrastorage.org/webt/n1/5a/jm/n15ajmeqozq7zfzc0uv3imzfebw.png)

Пройдемся по уникальных возможностям.

![](https://habrastorage.org/webt/k8/b7/br/k8b7brhqxwy1fa1vdurx0yolgra.png)

У Thanos есть такая фича как downsampling: 5-минутные и часовые интервалы, которые зачастую не правильно работают. Если погуглить и посмотреть их issue на github, там очень много issue, связанных с этим downsampling, что он иногда неправильно работает, либо работать не так как ожидают пользователи.

![](https://habrastorage.org/webt/iv/nq/4v/ivnq4vtxkkwf_1hs40ttkyhd1ie.png)

У Thanos есть дупликация данных для prometheus HA pairs. Когда 2 prometheus собирают одни и те же метрики с одних и тех же target и Thanos их складывает в Object Storage. Thanos умеет правильно дублировать эти данные в отличие от VictoriaMetrics.

![](https://habrastorage.org/webt/wn/rv/jh/wnrvjhzqrcx6_o9hoojijeivb8o.png)

У Thanos есть alert компонент, который был на схеме Thanos. Но этот же alert компонент подходит и для VictoriaMetrics. 

![](https://habrastorage.org/webt/ti/qp/ci/tiqpcidg_t_ahj9lornrey8_rda.png)

У Thanos преимущество что код у Thanos и у prometheus общий. Thanos и prometheus разработан 9 разработчиками. При улучшениях либо в Thanos либо prometheus выигрывает другая сторона.

![](https://habrastorage.org/webt/en/vo/o-/envoo-2amya3w4xo2c5lg-pa1bo.png)

У VictoriaMetrics главная фича это Extended PromQL. Это расширенный VictoriaMetrics, про который рассказывал на предыдущем big monitoring metup.

![](https://habrastorage.org/webt/yk/xq/hi/ykxqhipgapygdv7ra2c6rmaitnk.png)

VictoriaMetrics поддерживает заливку данных по множеству разных протоколов. VictoriaMetrics не только может принимать данные от prometheus, но и по протоколам Influx, OpenTSDB и Graphite.

![](https://habrastorage.org/webt/0k/zy/jp/0kzyjptb5sbscny1ffsdh2irqau.png)

Данные VictoriaMetrics занимают обычные намного меньше места по сравнению с Thanos и prometheus. 

Если записывать реальные данные, то пользователи говорят о 2-3-4-5 кратном уменьшение размера данных на диске по сравнению с  prometheus и Thanos.

![](https://habrastorage.org/webt/2y/xz/yg/2yxzygpbfnqt3ldibpggeo35rpw.png)

Еще одно преимущество VictoriaMetrics что она оптимизирована под скорость. 

![](https://habrastorage.org/webt/kk/jd/hy/kkjdhy-zae6bc_3uqu6yp3q1gwy.png)

Пройдемся по стоимости инфраструктуры.  

![](https://habrastorage.org/webt/79/yz/yw/79yzywujo1uxm6zc5rti6ebjzpq.png)

Одно из преимуществ Thanos то, что он сохраняет данные в object storage, который сравнительно дешевый.

При сохранении данных в object storage, вы должны оплачивать операции запись и чтение данных (~10 долларов за миллион операций). Когда вы записываете данные в object storage вы оплачиваете расходы вашего хостинга на загрузку данных в интернет, если ваш кластер находится не в AWS - там бесплатно. Когда вы считываете данные, вы оплачиваете от 10 до 230 долларов за 1ТБ. Это может быть существенно, если вы часто запрашиваете исторические данные Thanos кластера.

![](https://habrastorage.org/webt/eg/--/mv/eg--mv_wpjcv4m-tojp8l9r60ea.png)

Для Thanos кластера нужно оплачивать сервера для Compact, Store Gateway, Query компонентов, которые требует много памяти, ЦПУ для больших объемов данных.



![](https://habrastorage.org/webt/pz/0v/np/pz0vnpflfebkxmaqi3sbbucktz0.png)

У VictoriaMetrics расходы такие. Если хранить данные в GCE  HDD дисках за 40 $ за 1ТБ. У VictoriaMetrics достаточно обычная HDD дисков, не нужны никакие SSD, который раз пять дороже. VictoriaMetrics оптимизирована под HDD.

![](https://habrastorage.org/webt/dr/6q/lz/dr6qlzowqew8et2zpev1wkrq4cs.png)

Для VictoriaMetrics нужны сервера для компонентов: либо Single-nod либо для кластерных компонентов, которые в отличие от Thanos компонентов требует намного меньше ЦПУ, ОЗУ - соответственно будет дешевле. 

![](https://habrastorage.org/webt/ja/72/cr/ja72crhogqpdixrt8jzj-komdcq.png)

Примеры внедрения. 

![](https://habrastorage.org/webt/od/ji/fw/odjifwunkpjdwuqq-hbpwswu_cy.png)

У Thanos пример внедрения - это Gitlab. Gitlab полностью работает на Thanos. Но там не все так гладко. Если посмотреть по issue, то можно увидеть что у них постоянно возникают какие-то операционные проблемы с Thano: не хватает памяти для Store Gateway либо Query компонентов. Им постоянно приходится увеличивать объем памяти. 

Из-за этого увеличиваются расходы на это. Постоянно какие-то проблемы. 

Второе внедрение, которое может быть более успешное - это компания Improbable, которые начали разработку Thanos. Они опубликовали исходники Thanos. Improbable - компания, которая занимается разработкой игровых движков.

![](https://habrastorage.org/webt/t4/yh/y4/t4yhy4gyfrimyzyfopgdwk1ixyc.png)

У VictoriaMetrics публичные примеры внедрения это:

- wix.com конструктор сайтов.
- Adidas внедряет VictoriaMetrics и даже сделал доклад на последнем PromCon 2019. 
- TrafficStars - компания это ad network
- Seznam.cz - какой-то чешский поисковик.

А дальше пошли ноунейм компании, которые я не могу назвать сейчас. Они не дали согласие. 

- Один крупный разработчик игр крупнее чем им Improbable.
- Крупный разработчик графического ПО
- Крупный российский банк.
- Европейский производитель ветряных турбин, который успешно протестировал VictoriaMetrics. Этот производитель внедряет VictoriaMetrics для мониторинга данных полученных с ветряных турбин со скоростью 50 сэмплов в секунду на каждый датчик. В каждой ветряной турбине по несколько сотен датчиков. У них несколько сотен ветряных турбин.
- Российские авиалинии, которые хотят внедрить VictoriaMetrics, но все никак не могут. Мы с ними на стадии договора.

![](https://habrastorage.org/webt/lw/0f/r3/lw0fr3hdyce3yw6n0i3ywjnpxhs.png)Выводы.

VictoriaMetrics и Thanos решают похожие задачи, но по разными способами:

- Global query view
- горизонтальное масштабирование  
- произвольный retention

![](https://habrastorage.org/webt/pv/-b/ob/pv-bobndxjkjrcukph5xm3sa-98.png)

 Cпасибо.







![](https://habrastorage.org/webt/gu/0s/w5/gu0sw56ppge1icdy-ensv-wl3f4.png)
