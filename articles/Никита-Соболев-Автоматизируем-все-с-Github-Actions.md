![](https://habrastorage.org/webt/-a/lb/hm/-albhmv9qeedemv37c_yhko-t9i.png)

[Никита Соболев](https://habr.com/ru/users/sobolevn/) покажет множество функций Github Actions, поделится собственными впечатлениями и болями от первых месяцев использования, покажет, как сделать собственные инструменты.

<cut />


<oembed>https://www.youtube.com/watch?v=QoCSvwkP_lQ</oembed>


**Доклад 2019 года. Доклад интересный, полезный даже в 2021. Обратите внимание, что с 2019 года некоторые Actions могли измениться.**

![](https://habrastorage.org/webt/j2/ec/ei/j2eceiirn8koerua_djwwvoz3gc.jpeg)

Ребята, привет! Меня зовут [Никита Соболев](https://habr.com/ru/users/sobolevn/) - [sobolevn (Nikita Sobolev) · GitHub](https://github.com/sobolevn). Я технический директор компании «[We make services](https://wemake.services/)». У меня есть несколько традиционных вопросов для вас. 

Как вы думаете, чем мы занимаемся?

*Сервисами.*

Совершенно верно, мы делаем сервисы. А второй традиционный вопрос – кто-нибудь слышал про такую компанию до этого момента? Три человека. Это, действительно, интересно. У нас маленькая компания. Примерно 5 человек. Но при этом я выступаю на сцене с людьми из Netflix, Google, AWS, что очень приятно. 

![](https://habrastorage.org/webt/sm/vs/tt/smvstt6jtickmmcficj8ncklwuy.jpeg)

И делаю я это выступление не просто так. Я рассказываю о том, что на самом-то деле есть очень классная и простая штука, которая называется Github Actions. И сегодня я вам про нее расскажу. Я вам расскажу о том, как можно просто и быстро автоматизировать разное, а точнее все. 

![](https://habrastorage.org/webt/2a/cv/rb/2acvrbyypghjnwpwyvmmims8wzy.jpeg)

- Я знаю, что на конференцию по DevOops приходят несколько типов людей. Первый тип людей – это девелоперы. В моем докладе сегодня будет много про Dev, про то, как работать с кодом; про то, как делать полезные для девелоперов штуки быстро и просто. 

- Но с другой стороны мой доклад будет еще и для опсов, потому что опсы присматривают за нерадивыми девами. Они делают так, чтобы все работало и вообще они замечательные люди. 

Про инфраструктуру тоже сегодня будет. У этого доклада очень сложная судьба. Это один из самых сложных докладов, который я готовил в плане подготовки. 

![](https://habrastorage.org/webt/aq/vl/tp/aqvltpibpvbwf2gjmccmgp5styu.jpeg)

Все знают это лого. Это [GitLab](https://gitlab.com/). И мы живем на [GitLab](https://gitlab.com/), у нас все хорошо. На самом деле у меня все прекрасно с GitLab. Я не жалуюсь, есть [GitLab CI](https://docs.gitlab.com/ee/ci/), платный аккаунт. Все хорошо. 

Я сначала хотел рассказать про наши инструменты, которые мы используем для работы с [GitLab](https://gitlab.com/).

![](https://habrastorage.org/webt/_a/xz/ty/_axztyeguqyw5hautydx2tvpqf0.jpeg)

Мы написали целую кучу разных велосипедов для того, чтобы автоматизировать все в GitLab:

- У нас свои боты в [GitLab CI](https://docs.gitlab.com/ee/ci/), которые выполняются на каждый commit, на каждый pull request и т. д. У нас их там с пяток самых разных. 
- У нас есть огромное количество ботов, которые выполняются в Cron, которые живут на [Heroku](https://www.heroku.com/) и т. д. Их, наверное, штуки три. 
- И нас есть свои собственные боты. Это большие приложения. Некоторые из них, действительно, большие приложения, некоторые из них чуть поменьше. Это монолиты, которые слушают [вебхуки](https://ru.wikipedia.org/wiki/Webhook) и живут непонятно где. Я это все поддерживаю, у нас есть инфраструктура. Один даже в [Kubernetes](https://kubernetes.io/) живет. Это единственный раз, когда я про него сказал и все.

Всей этой историей я хотел бы поделиться с вами, т. е. как мы много всего сделали, и какие вещи вы можете взять, поставить себе и использовать. Это все в open source. 

![](https://habrastorage.org/webt/he/kw/uz/hekwuz5vshbpwfytvym-ucfenie.jpeg)

А потом, где-то месяца 3 назад, мне пришло письмо счастья. И в нем [GitHub](https://github.com/) говорит: «Пожалуйста, примите участие в нашей закрытой Бете [Github Actions](https://github.com/features/actions)». Я такой: «Пацаны, все меняется. Отменяем все. GitLab, до свидания, теперь мы будем рассказывать про [Github Actions](https://github.com/features/actions)».

![](https://habrastorage.org/webt/mj/rs/py/mjrspyl-at1do3a3kevgeactjcw.jpeg)

Многие могут задаться вопросом: «В чем разница?». Потому что эта технология новая. И я не буду вам подробно рассказывать про GitLab, потому что про GitLab многие знают, а про Github Actions расскажу подробней. 

![](https://habrastorage.org/webt/-k/ia/cu/-kiacufum7rgd0gsn-l5lhfi7r4.jpeg)

В чем между этими двумя технологиями разница концептуально? Концептуально разница в платформе и сообществе. Как вы понимаете, платформа и сообщество – это две важные составляющие одного и того же. 

Когда мы говорим, что у нас есть хорошая платформа, то мы подразумеваем, что эта платформа позволяет делать какие-то автоматические вещи просто и быстро. 

Когда мы говорим, что у нас есть сообщество, то мы подразумеваем, что есть большое количество инструментов, которое сообщество делает, поддерживает, а также есть лучшие практики по внедрению этих инструментов и по работе с ними. 

Кто-нибудь из вас может похвастаться тем, что у GitLab CI есть хорошая платформа? Т. е. у них нет никакой платформы для работы с ботами. Это правда. У них есть прекрасный CI, но все, что нужно автоматизировать вокруг, - это очень больно. А сообщества у GitLab нет вообще. Т. е. если вы попытаетесь загуглить «GitLab бот», то вы не найдете практически ничего. А если что-то найдете, то это, скорее всего, сделал я. Это грустная история. 

А с GitHub это не так, потому что еще GitHub в приватной Бете, но у них уже есть хорошие платформы, хорошее сообщество. 

![](https://habrastorage.org/webt/dg/as/5s/dgas5s7tm_wxwmtsynl9ezpe9wy.jpeg)

Чтобы вы понимали уровень перегиба того, что сейчас происходить на Github, покажу вам смешную картинку. 

На этой картинке некий бот. Пока нам не важно, что он делает. Это просто бот, который присылает pull request с обновлением зависимости. Мы сейчас будем подробно об этом говорить. Но это просто юмор. Это сложный технический юмор.

![](https://habrastorage.org/webt/xc/ul/rj/xculrjak7skezq3_ywl7v-hqun8.jpeg)

А на следующем шаге другой бот принимает этот pull request, а третий бот празднует. Вы понимаете, насколько абсурдно, но с другой стороны, круто происходит эта автоматизация. 

И мне кажется, что это, действительно, очень круто. Я, когда подавал этот доклад, у меня коленки тряслись от радости. Т. е. я большой фанат технологий. И когда я вижу что-то классное, то думаю: «Блин, как круто!». И этим чувством я хочу с вами поделиться. Мне GitHub, к сожалению, не платит, поэтому я это делаю бесплатно, поэтому я это делаю искренне. 

![](https://habrastorage.org/webt/tq/bx/vv/tqbxvvq6zju35r70usmshfo6oia.jpeg)

И начну с рассказа про платформу. 

Что у нас есть? Github Actions – это, действительно, большая платформа, чтобы запускать всякое. 

![](https://habrastorage.org/webt/af/xh/hx/afxhhxgccdh-pgqwc3wht_vsd48.jpeg)

Если у вас есть приватный репозиторий, то 2 000 минут Github Actions вам даются бесплатно вместе с этим пакетом. 

Дальше идут вот такие цены. 

Каждая дополнительная минута в зависимости от типа платформы: Linux, Windows, MacOs идет за определенную стоимость. 

![](https://habrastorage.org/webt/86/yr/5e/86yr5eq-maqbw9ktyew-lg_x--i.jpeg)

Но самое главное, что для open source это все бесплатно. Вообще все и навсегда. Это то, за что мы любим GitHub. И так как у меня там куча open source, то я очень люблю, что все бесплатно. И это круто. 

![](https://habrastorage.org/webt/na/5h/mg/na5hmglveuoxm_i4mw8ilgxehlc.jpeg)

Давайте перейдем к конкретике. Вот у нас есть конкретная задача: нам нужно прогнать линтер для Python проекта.

Я очень люблю линтер. Я много линтеров пишу сам, в также использую чужие. 

В общем, задача понятна: прогнать линтер на существующие Python-код.

![](https://habrastorage.org/webt/ou/nk/yl/ounkylfb7i1eccne3q2fiagojzw.jpeg)

Как мы будем это делать? Вот наш проект - https://github.com/wemake-services/wemake-python-styleguide . Мы заходим в него. 

![](https://habrastorage.org/webt/u8/j2/vj/u8j2vjz0gdbj2p5xmqgs5t44pje.jpeg)

Видим в главном меню кнопку «Actions». И переходим туда.

![](https://habrastorage.org/webt/wc/n7/ir/wcn7irpavnv_wol0388bq0gic0m.jpeg)

Дальше у нас появляется дефолтное окно, когда еще никакие actions не настроены и вообще ничего нет. И мы можем выбрать дефолтный набор действий.

![](https://habrastorage.org/webt/qq/qi/kd/qqqikd2c7bsy25emmn66r-0f-jw.jpeg)

Например, мы выбираем Python-пакет. Нажимаем эту кнопку.

![](https://habrastorage.org/webt/9h/np/wd/9hnpwd-p3h4x8fody2wytejssyw.jpeg)

У нас появляется следующий экран. На этом экране слева мы можем видеть код, который будет выполняться. Это некий [YAML](https://ru.wikipedia.org/wiki/YAML), который будет определять то, что вы будете делать. А справа расшифровка, что это такое. 

![](https://habrastorage.org/webt/sy/rb/m9/syrbm9q4dpyywrp0tiugvyhbg0u.jpeg)

Если вы думаете, что будут еще какие-то действия, то нет. Мы нажимаем на зеленую кнопку «Start commit» и готово. 

Понятно, что на этом этапе у нас создался очень базовый, сферический Python-проект в вакууме. Но на самом деле этого шаблона достаточно для того, чтобы кастомизировать все по себе очень и очень подробно. 

А теперь, когда мы сделали это простое действие, прогнали линтер для нашего Python-кода, давайте посмотрим на то, какие термины используются в этой структуре, что есть и как это работает. 

![](https://habrastorage.org/webt/eu/wp/uq/euwpuqbacghmqym0xdkzqqxv5mo.jpeg)

[Workflow](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#workflows) – это высокоуровневый набор правил для того, чтобы делать какие-то действия при определенных условиях. 

[Job](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#jobs) – это сущность, которая запускает какую-то задачу в рамках вашего workflow. 

[Step](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#steps) – это шаг. Т. е. один job может состоять из нескольких шагов. 

[Action](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#actions) – это тот переиспользуемый unit, который попал в название GitHub Actions, т. е. это самая минимальная часть, которую можно переиспользовать в нашем CI.

[Event](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#events) – это термин, который обозначает, что случилось какое-то действие. А если действие случилось, то мы запускаем какой-то конкретный workflow.

![](https://habrastorage.org/webt/zz/cc/3h/zzcc3hkdox2ji90e-5q5kgr0uzo.jpeg)

Давайте посмотрим на тот код, который сгенерировался в нашем прошлом примере с Python линтером и посмотрим, что там вообще есть. Его там было довольно много, но он был достаточно простой. 

В начале мы определяем workflow. Для того чтобы определить workflow, нам потребуется несколько пунктов:

- Название.
- Event. В нашем случае мы работаем только на push.
- И также определение jobs, т. е. какие jobs у нас будут.

![](https://habrastorage.org/webt/_r/0x/zy/_r0xzyh36l0lovfpssdgl_ufvai.jpeg)

Первый job у нас будет build. 

И здесь мы можем выбрать операционную систему, которую хотим запустить. В нашем случае я выбираю Ubuntu. 

![](https://habrastorage.org/webt/ge/9b/mx/ge9bmx42yts7q0e8gv1zcwilbgk.jpeg)

Но есть варианты выбора из Linux, MacOS и Windows, т. е. на всех трех платформах можно запускать. Если вы запустите на трех, то у вас появится матричная сборка, т. е. [build matrix](https://docs.github.com/en/actions/learn-github-actions/managing-complex-workflows#using-a-build-matrix).

![](https://habrastorage.org/webt/en/8y/xa/en8yxajwufp7pwnuxefcmtn_yg0.jpeg)

Что делаем дальше? Дальше мы определяем стратегию. 

Для того чтобы у нас все это запустилось матрично, мы хотим проверить разные версии Python. 

Версия [python 2.7](https://pythonclock.org/) скоро умрет, не проверяйте ее. 

Проверяем 3.5, 3.6, 3.7. 

А 3.8 мы не проверяем, потому что там оператор маржа. 

![](https://habrastorage.org/webt/wl/d8/po/wld8poh4ci6z0bhzpdmn0rcyxr8.jpeg)

Дальше мы определяем шаги для того, чтобы сказать, что конкретно делать надо. 

![](https://habrastorage.org/webt/2n/du/pf/2ndupfoowxdcyubc8dag_pdmj_m.jpeg)

Как выглядит это на практике? У нас появляется вот такое большое количество шагов. Раньше у GitHub был красивый визуализатор, но они его выпилили. 

Мы можем посмотреть и потрогать то, как будет выглядеть наш CI дальше. 

![](https://habrastorage.org/webt/he/fe/uc/hefeucskfsagxsnvfn6rmte28bq.jpeg)

Переходим к этапу подготовки. Перед тем как прогнать непосредственно проверки, нам нужно подготовиться к тому, чтобы все работало: чтобы код был на месте, все зависимости были установлены и т. д. 

![](https://habrastorage.org/webt/ao/zv/mu/aozvmuzonyzjlnq4jkajy1uejvo.jpeg)

Первый шаг – это использование action, который называется checkout.

Checkout – это всегда первый action, т. е. это тот action, который копирует ваш код и делает git pull в текущую папку.

![](https://habrastorage.org/webt/iw/uh/2x/iwuh2xaw3nypmmclz3gb2qmeaji.jpeg)

Следующий action имеет имя: «Пожалуйста, разверни мне Python» и дальше какая-то абракадабра. И использует [action](https://github.com/actions/setup-python), который называется [setup-python](https://github.com/actions/setup-python). Это то, что устанавливает вам Python. 

![](https://habrastorage.org/webt/dx/ew/oz/dxewozisbbn1dqulopca1l5uifi.jpeg)

Обратите внимание на эту абракадабру. Это не просто так. Это указание того, какую версию Python мы указали в нашей матрице. 

![](https://habrastorage.org/webt/qk/i5/bx/qki5bxendoospztr-dn6zwkp_d8.jpeg)

Помните, что у нас вот здесь был matrix и python-version. Вот это – это тоже самое. Мы таким образом можем передавать параметры внутрь action.

![](https://habrastorage.org/webt/ye/xe/w8/yexew8f1gow_asjytraqmrxgqfu.jpeg)

И последний шаг. Мы говорим: `pip install –r requirements` и т. д.

Как вы понимаете, это не специфично для Python. Это может точно так же работать с Java, Ruby и т. д. Я привожу пример на Python, потому что я к нему привык. 

![](https://habrastorage.org/webt/hv/fw/wl/hvfwwldgoorlblwe7vrcmfb9yim.jpeg)

С подготовкой мы закончили. Что здесь нужно увидеть и запомнить? Что у нас есть два ключевых слова, которые помогают нам либо использовать готовые actions – это слово `uses`, т. е. мы можем указать имя готового action, либо ключевое слово «run» - это значит выполнить какую-то shell-команду. 

В принципе, все понятно и просто. 

![](https://habrastorage.org/webt/26/0s/hn/260shn3civdx8big9aq0y2ujbyq.jpeg)

Все подготовили. Теперь надо проверить, что наш код соответствует правилам линтера, который мы выбрали. 

![](https://habrastorage.org/webt/qx/ic/er/qxicer9febo-te0z_xha884i0wc.jpeg)

Для того чтобы это сделать, нам нужно будет дописать еще два steps.

В первом мы говорим: «Пожалуйста, установите мне [flake8](https://pypi.org/project/flake8/) (это линтер для Python) и прогоните его».

![](https://habrastorage.org/webt/l1/lx/pq/l1lxpqhurmbym7wucceo3tkuduk.jpeg)

А во втором давайте еще тесты прогоним, т. е.: «Установите тесты и прогоните тесты». 

Все, это полное описание того, что мы только что проделали с первого до последнего шага. Больше там ничего нет. 

![](https://habrastorage.org/webt/0o/hh/7m/0ohh7mbbx7difhtbvltokczyl1g.jpeg)

Теперь мы можем посмотреть на результаты. Как вы видите, тесты успешно прошли. Одни красненькие крестики. И мы можем посмотреть, что конкретно случилось. 

Слева список наших workflow, а справа список workflow run, т. е. когда наш workflow запускается, создается график справа.

![](https://habrastorage.org/webt/3f/km/we/3fkmweztegscbeisuehmqrxwucu.jpeg)

Мы выбираем какой-то конкретно. Нажимаем на него. Переходим. 

![](https://habrastorage.org/webt/fj/cl/yj/fjclyjo1v0wlilwk-ax4be4mdn4.jpeg)

И можем посмотреть, что внутри происходит. 

Слева список jobs, а справа вывод нашего конкретного приложения. В нашем случае он ругается и говорит, что твой python-код плохой. 

![](https://habrastorage.org/webt/m-/85/4b/m-854bc1rbc0z5h_yxipzo2-xo8.jpeg)

Этот интерфейс достаточно простой и понятный. Практически в каждом CI так. 

![](https://habrastorage.org/webt/kl/ln/gx/kllngxktqalfettjk7osnqkzpva.jpeg)

Я бы хотел подвести промежуточный вывод. У нас инфраструктура, действительно, написана как код даже на этапе CI.

![](https://habrastorage.org/webt/_a/ku/lf/_akulf8dvi7pzkqtrda9anhuyce.jpeg)

Но есть один ключевой нюанс, который я бы хотел особо выделить. Это не просто инфраструктура как код, это инфраструктура как переиспользуемый код. 

Видите, там даже есть слово uses, т. е. используй какую-то готовую компоненту для того, чтобы делать хорошо. Используй action для того, чтобы установить Python. Используй action для того, чтобы сделать checkout твоего кода. Используй action для чего угодно – вот основная идея, т. е. используй action, чтобы делать все, а community позаботиться о том, чтобы наполнить GitHub этими actions. А платформа GitHub позаботится о том, чтобы это все работало. 

На мой взгляд, основная идея GitHub Actions – это как раз в этом, т.е. переиспользуемый код для инфраструктуры. 

Об этом очень часто говорят, но реализация все время разная. И [Helm](https://helm.sh/) иногда тоже называют таким инструментом для переиспользования кода. Но, мне кажется, что здесь сравнение не в пользу Helm.

![](https://habrastorage.org/webt/sm/vs/tt/smvstt6jtickmmcficj8ncklwuy.jpeg)

Давайте теперь посмотрим, как можно все автоматизировать с GitHub Actions. Что такое все и как это можно автоматизировать?

![](https://habrastorage.org/webt/qg/9r/w7/qg9rw7wyk1wmwetqxajsizyebl8.jpeg)

Для того чтобы разобраться с этим вопросом, давайте посмотрим на типы событий, которые у нас есть внутри. Какие?

- Push, commit. Когда мы делаем push или commit, то происходят какие-то действия в нашем CI. Мы можем проверять на тесты, гонять линтеры и т. д.
- Issues, pull_request. Когда кто-то создает issues или pull_request, мы тоже можем выполнять какие-то действия. Например, проверять правописание, чтобы человек перед заказчиком не опозорился и слово «доделать» написал правильно. Или, например, он открывает pull_request, и мы можем проверить, что он открыть pull_request в нужную ветку, с нужным оформлением, с нужным названием и т. д. 
- Issue_comment, label, Gollum. Также мы можем проверять, что произошло, например, какое-то действие: кто-то прокомментировал, добавил какой-то label или поправил wiki. Не знаю, почему wiki называется Gollum. У них события, которые правят wiki, называются Gollum.
- Вы можете даже onboarding привязать. Когда вы добавляете нового человека в репозиторий, у вас случается событие member, у вас может начаться onboarding.
-  И еще есть событие по расписанию, есть webhook и много всего. 

![](https://habrastorage.org/webt/zv/z0/hn/zvz0hnkgqheumkjhuatm9sy6fmq.jpeg)

Но самое интересно, что можно пойти еще глубже.

![](https://habrastorage.org/webt/jt/a-/q5/jta-q5zxhezlmhps8_0bjx0exjw.jpeg)

И у каждого события есть не просто событие, но и тип события. Например, у pull_request. Смотрите их сколько. Я не буду их все зачитывать, но их очень-очень много. 

По сути, любое изменение state pull_request создает событие с определенным типом. Вы можете сказать: «Кода pull_request готов к review, найди reviewer» и т. д. 

На мой взгляд, это делает автоматизацию очень простой, потому что мы можем точно знать, когда и что случилось.

![](https://habrastorage.org/webt/xr/af/gw/xrafgwkn67de-60uyoeqljmf7dc.jpeg)

Естественно, вы можно создавать свое. Основная фишка, что вы можете насоздавать своих actions и переиспользовать их внутри своих проектов. 

![](https://habrastorage.org/webt/vg/oy/x8/vgoyx8eb52ur537iw03l_yglqne.jpeg)

Потребуется:

- Dockerfile.
- Action.yml.

Иногда достаточно Dockerfile, иногда достаточно action.yml. Это две основные компоненты, которые плюс-минус всегда нужны. 

![](https://habrastorage.org/webt/kj/hv/s7/kjhvs78jkjuuaf6zrytsgms2cou.jpeg)

Давайте посмотрим на пример. Возьмем Dockerfile.

![](https://habrastorage.org/webt/rs/uo/gq/rsuogqzber1bbaklzgtfma3amqw.jpeg)

Что там будет внутри? Мы делаем не просто какую-то штуку, мы делаем реальный action, который вы можете потом найти в [Marketplace](https://github.com/marketplace). Он работает, он существует. Это настоящий action.

Это action для моего линтера. Я уже говорил, что я достаточно много линтеров пишу. 

![](https://habrastorage.org/webt/ok/yg/xw/okygxw2u0ssaf7kwjjxckxtgpzk.jpeg)

Он называется [wemake-python-styleguide](https://github.com/marketplace/actions/wemake-python-styleguide). И он проверяет код на то, что он соответствует нашему стандарту внутри компании. 

Как это выглядит? Все очень просто. У нас есть базовый образ Python. Мы устанавливаем туда bash, потому что это зависимость инфраструктурная. Говорим: `pip install wemake-python-styleguide`, потому что это основная зависимость, которая будет делать логику.

![](https://habrastorage.org/webt/yy/kk/lm/yykklmhagu1p_tpfhxl-f3tyk1c.jpeg)

И определяем [entrypoint](https://docs.docker.com/engine/reference/builder/#entrypoint). Это, по сути, то, где будет храниться наша логика, как это выполнять. Определяем, говорим: «Entrypoint».

![](https://habrastorage.org/webt/71/s-/cj/71s-cjkb9_jvsy7xpfbk6ynt2ia.jpeg)

И вот сам `entrypoint.sh`. В нем я говорю: «Прогони мне линтер при помощи flake8, забери статус».

![](https://habrastorage.org/webt/fb/ei/u8/fbeiu8t2qjyow-9saq7ud-9y_l0.jpeg)

Говорю: «Сделай мне такую интересную штуку». Это GitHub специфичная вещь, попрошу вас на это обратить внимание. Мы говорим: «Задай, пожалуйста, вывод вот таким». Это специальный синтаксис для того, чтобы action знал, какой у него был вывод. 

![](https://habrastorage.org/webt/ae/_q/me/ae_qmedxuijw2j5ihtqrprbtbly.jpeg)

Соответственно: «Если у него статус неравен нулю, то, пожалуйста, упади, скажи, что все плохо и взорвись», чтобы CI провалился. Все, наш action готов.

![](https://habrastorage.org/webt/n4/u9/ti/n4u9tiqndqdakcmz4hgp7apb1ki.jpeg)

Для того чтобы его опубликовать, нам потребуется зайти на страничку GitHub, создать релиз. 

![](https://habrastorage.org/webt/cq/rn/go/cqrngotdqn613v1ehkwqlzjigaq.jpeg)

Указать галочку: «Пожалуйста, этот релиз мне опубликуй в Marketplace». Нажать ее и готово. 

![](https://habrastorage.org/webt/pi/fp/oj/pifpojiiegz590jade-ynqiqki0.jpeg)

Теперь давайте посмотрим, как его использовать. Использовать мы его будем вот так. У нас есть какие-то jobs, внутри них есть steps. И один из шагов будет: «Прогони, пожалуйста, мой линтер, который мы только что создали».

![](https://habrastorage.org/webt/nt/wc/8h/ntwc8hdzior9buslyc2oo8jio1i.jpeg)

Мы можем указать название организации/название репозитория и `@latest`.

`@latest` означает – используй последнюю версию. На самом деле – это не лучший вариант использовать последнюю версию. Здесь ее, естественно, можно пинить: используй версию `1`, `2`, `2.5` и т. д. 

![](https://habrastorage.org/webt/x1/fu/ut/x1fuut84us8fhhxzrcj8qup6tyy.jpeg)

Также можно передавать параметры. Вы часто хотите линтить не весь свой код, который у вас есть в репозитории, а какой-то subset этого кода. Например, вы говорите: «Передай мне, пожалуйста, параметр, чтобы путь был равен src».

![](https://habrastorage.org/webt/a9/l0/wa/a9l0wauxr7ja2eplhktlthd9kc8.jpeg)

И дальше можете сказать: «Покажи мне, пожалуйста, outputs, который сделал наш линтер». 

![](https://habrastorage.org/webt/oa/gz/mh/oagzmhvxjzkaeilvappi6eu0tjq.jpeg)

Помните, что мы создавали outputs со специальным синтаксисом?     

Вы можете outputs выдать в какой-нибудь файл log.txt, который, например, вам нужен. 

Обратите внимание, что у нас есть входные и выходные параметры. 

![](https://habrastorage.org/webt/7w/nx/6b/7wnx6bd3mpizhh86jnly1hidbiw.jpeg)

Да у нас же композиция! В чем основная идея? Если у вас есть входные и выходные параметры, то у вас появляется композиция. Вы можете спокойно делать композицию GitHub Actions, как композицию функции.

![](https://habrastorage.org/webt/dv/vb/fq/dvvbfqjajg5tqcams5jwbumxrju.jpeg)Как это будет выглядеть? Вывод первого action становится входным параметром для другого action, причем необязательно делать композицию подряд. А из-за того, что у вас есть именованные параметры, вы можете делать композицию так, как вам нужно. Это круто и очень удобно. 

![](https://habrastorage.org/webt/xo/-z/vb/xo-zvb9c69sc78yerogkbuudnbg.jpeg)

Теперь посмотрим на то, что такое action.yml. Action.yml – это файл с метаданными. И в этом файле с метаданными хранится информация, как называется, какая картиночка, описание и т. д. Но не только.

![](https://habrastorage.org/webt/1q/jc/mf/1qjcmflwllzvycm-fsgh5mvmdua.jpeg)

Имя понятно.

![](https://habrastorage.org/webt/6n/wq/f8/6nwqf8ejmukwaj87kivahcc6zqy.jpeg)

Мы можем указать список параметров, которые мы ожидаем. Помните, что я говорил, что у меня есть входной параметр, который называется `path`? Вот здесь я его определяю. У меня путь – это список путей, которые я проверяю. Он необязательный, по умолчанию точка, т. е. дефолтная директория. 

![](https://habrastorage.org/webt/c1/af/bo/c1afbozarddftydc6yj4zibtwtq.jpeg)

Также нужно определить выводы, если мы их используем. Вывод у нас один, он называет `output`. И описание. 

![](https://habrastorage.org/webt/l8/vy/em/l8vyemwsddewq0beckua4o3ut0g.jpeg)

И последний момент – это то, как запускать наш GitHub Actions. В нашем случае мы будем использовать Docker. Вы можете использовать Python и т. д. Но чаше всего советуют использовать Docker, потому что есть переносимость, удобство. И передать аргументы, которые нам нужны. Все. Наш action готов, опубликован. Он полностью работают.

![](https://habrastorage.org/webt/ho/rh/d1/horhd1vxg4dbx2cxssfdxsdy6wq.jpeg)

Но можно взять фреймворк и написать другой action.

![](https://habrastorage.org/webt/r6/lu/hg/r6luhgc4g9ttx2avp0baty37m70.jpeg)

Например, готовые npm компоненты:

- [@actions/core](https://github.com/actions/toolkit/tree/main/packages/core) - работает со всякими входными и выходными параметрами. 
- [@actions/exec](https://github.com/actions/toolkit/tree/main/packages/exec) – для того, чтобы запускать бэкграунд-задачи. 
- [@actions/io](https://github.com/actions/toolkit/tree/main/packages/io) – для того, чтобы создавать файлы и папки.
- [@actions/tool-cache](https://github.com/actions/toolkit/tree/main/packages/tool-cache) – для того, чтобы создавать cache.
- https://github.com/actions/toolkit/tree/main/packages/github – для того, чтобы общаться с API GitHub.

И вы можете написать на этом целый фреймворк поверх и сделать супер action.

![](https://habrastorage.org/webt/lp/eg/7h/lpeg7huc1zmivvb5x90vz86rqda.jpeg)

Например, я сделал action, который запрещает людям материться в pull_request. Не потому что я плохой, а потому что это весело. 

Давайте посмотрим, как это делается. Сверху мы пишем в javascript два import: github и core. 

![](https://habrastorage.org/webt/om/67/i8/om67i8itpf9rmf6ry7fxqww_kd4.jpeg)

А здесь мы пишем: «Пожалуйста, создай мне новый клиент GitHub от такого-то токена». И: «Посмотри, если у нас пришел eventName, который issue_comment, то, пожалуйста, проверь, что в этом issue_comment нет каких-то плохих слов».

![](https://habrastorage.org/webt/cs/sm/-x/cssm-x6uvuv6sy-6fanjle1rcly.jpeg)

«А если есть, то, пожалуйста, обнови комментарий, который равняется Input дефолтному с именем `text`». 

![](https://habrastorage.org/webt/s7/mv/8p/s7mv8p_y70_7umph_hvun1lfo7i.jpeg)

И, соответственно, мы это можем использовать.

![](https://habrastorage.org/webt/sb/kx/rx/sbkxrxe7cbv5yjo8xaumlw_rm8a.jpeg)

Как? Передать все события, в которых может случится с их подтипами. 

![](https://habrastorage.org/webt/zl/7u/1l/zl7u1l2u2krttucttf1hnjpqwkk.jpeg)

И сказать: «Если кто-то матерится, накажи их». Давайте посмотрим, как это будет выглядеть на практике.

![](https://habrastorage.org/webt/w5/nb/ws/w5nbwsuu0f0ulzolr0sjbqgkf5o.jpeg)

Кто-то пишет плохое слово. И его комментарий меняется на: «Мне очень жаль». Это весело и забавно. Вы можете себе поставить и троллить коллег. Это весело. 

![](https://habrastorage.org/webt/44/xn/rl/44xnrlwvu08fjwq7yio2omnp8ti.jpeg)

Вот ссылка - https://github.com/sobolevn/restrict-cursing-action. Он тоже опубликован. Вы можете посмотреть, как это работает внутри. Это демонстрационный вариант того, как можно писать свои сложные actions, которые ходят в API и делают всякое. 

![](https://habrastorage.org/webt/ck/mc/ss/ckmcss-4kxnqoaen5zkl_h6bz2m.jpeg)

Естественно, можно использовать готовое. Фишка как раз в сообществе, в большом количестве проектов, в том, что можно брать и использовать.

![](https://habrastorage.org/webt/kv/h8/ng/kvh8ngvrobsdpmdrbbcgxlvzmzo.jpeg)

Скажу сразу, что я буду говорить не только про GitHub Actions, но и про GitHub Apps. Это очень близкие концепции, Но они немножко разные. Но для нас это сейчас не играет роли. Все одно и тоже. 

![](https://habrastorage.org/webt/fx/bz/vp/fxbzvp-kqmqusruxhoz9zxmm9ne.jpeg)

во-первых, очень удобно, когда вам кто-то говорит, где конкретно вы ошиблись. Если у вас пропущена запятая, то вам в коде покажут, что тут. Для этого есть прекрасный [Reviewdog](https://github.com/apps/reviewdog).

![](https://habrastorage.org/webt/18/ht/nt/18htntochtgpttafimgetieok6a.jpeg)

Это приложение. Можно зайти в Marketplace и установить одной кнопкой. 

![](https://habrastorage.org/webt/xp/hw/oj/xphwojyoofben2qjrtjbuwzseva.jpeg)

В принципе, все приложения, которые буду показывать, можно установить одной кнопкой. 

![](https://habrastorage.org/webt/in/tf/uf/intfufbd2xrs_8_1ne54-ek7ab0.jpeg)

И вот, как он будет работать. Он находит какое-то место, которое ему не нравится и говорит: «Вот здесь плохо». И создает комментарий, он говорит: «Вот тут плохо. Поправь». И говорит, что поправить. И вы можете с ним подискутировать, если вам нравится разговаривать с ботами. 

![](https://habrastorage.org/webt/5q/vi/lb/5qvilbyxoont3cnw1epz0aolbuo.jpeg)

Работает с любым языком или любым линтером. 

![](https://habrastorage.org/webt/qu/tu/8u/qutu8uabxf3ybzdh-qx_1w6oxgy.jpeg)

Почему? Потому что у него есть специальный формат того, как он может работать. У него есть заготовленные форматы для популярных ошибок, например, для Python, Java Script и т. д.

![](https://habrastorage.org/webt/k0/ju/rf/k0jurfjeimamjiz92jzfrcdtdme.jpeg)

Но вы можете задать свой. Например, вы можете отформатировать и сказать, где у него конкретно название файла, где номер строчки, где номер символа. И таким образом любой линтер туда подключается. Очень круто. 

![](https://habrastorage.org/webt/lw/_0/mb/lw_0mbmrql8xvovqtav4w5pxs3g.jpeg)

Особенно удобно, если вы занимаетесь InfoSec, то там есть особые плагины для этого:

- Для Python – это наш собственный [wemake-python-styleguide](https://github.com/marketplace/actions/wemake-python-styleguide), который кучу ошибок в безопасности находит. А также есть bandit и [dlint](https://github.com/marketplace/actions/dlint-check). Они работают с этим tool замечательно. 
- Для JS – это плагин eslint-plugin-security.
- Для Elixir – это [sobelow](https://github.com/marketplace/actions/sobelow).

Так как я работаю с тремя этими языками, я знаю про них. Для других, наверное, что-то тоже есть. 

![](https://habrastorage.org/webt/zv/sk/22/zvsk22iy_5epntclrcl888cnaem.jpeg)

Следующий офигенный tool. Я состою в сообществе этого инструмента, но не очень активно в него коммитчу. Это Danger. 

Что такое [Danger](https://github.com/apps/Danger)? Это платформа для автоматического code review. Мы дожили, можно делать code review автоматически. 

![](https://habrastorage.org/webt/_-/0n/xj/_-0nxjw6o9hjjn0lqxutupawqjo.jpeg)

Показываю. Сначала у нас происходят push code, прогоняются линтеры, тесты. Потом работает Danger. И только потом код попадает на review человеку. 

![](https://habrastorage.org/webt/6o/lb/p-/6olbp-p-ufbrlarn_soqmd87xxg.jpeg)

Для каких платформ это работает? Вы можете писать код для того, чтобы Danger его понял и понял, как проверять ваш другой код, на трех языках. Это Java Script, Swift и Ruby. 

Я пробовал Java Script и Ruby. Swift не пробовал. 

![](https://habrastorage.org/webt/i7/p5/nv/i7p5nv7-tmgblkg7dxy1kzptsk0.jpeg)

Что это такое? Давайте посмотрим. Заходите репозиторий [Danger](https://github.com/danger/danger). 

![](https://habrastorage.org/webt/xc/hj/ri/xchjri6yhgjtsgzw-an3qru2ljo.jpeg)

Видите плашку «View on MarketPlace», нажимаете на нее.

![](https://habrastorage.org/webt/6f/lg/q1/6flgq1nndwi4ollitncyidvsxok.jpeg)

![](https://habrastorage.org/webt/gn/5s/mm/gn5smm5jomqtg2if-c5e8q3hhiq.jpeg)

Переходите на страничку Marketplace, нажимаете «Use latest version». 

![](https://habrastorage.org/webt/1x/om/n1/1xomn1jnuwf5k0x0qwujc9oiv1g.jpeg)

Появляется такое окошко. Копируете эту штуку себе в step какого-то GitHub Actions. И все, работает. 

![](https://habrastorage.org/webt/ek/a9/g8/eka9g8yd7fmmdprqekflrcodl3k.jpeg)

Давайте посмотрим, как писать код. 

Я часто прошу разработчиков не делать некоторые вещи. Когда эти разработчики часто делают эти ошибки, я просто вешаю в CI какую-то проверку, чтобы больше этого не говорить. Потому что, зачем, когда можно автоматизировать?

Например, разработчики часто ошибаются и делают pull_request не к мастеру, а к какой-то другой ветке. Я говорю, что если ты делал pull_request не к мастеру, то, пожалуйста, CI упади. Т. е. не надо, не туда ты делаешь pull_request. 

Соответственно, если pull_request нельзя автоматически отребейзить, т. е. там есть какие-то конфликты, то тоже упади, пусть разработчик пойдет и отребейзит и пришлет мне нормальный. Соответственно, можно упасть.

![](https://habrastorage.org/webt/p6/t4/ds/p6t4ds89_nb8704u11dtw9qrg0c.jpeg)

Можно не падать, а просто сделать предупреждение. Например, если человек не написал body к вашему pull_request, т. е. не написал, что он делал, но предупредил: «Может быть, ты хотел поделиться? Как-то что-то сделать?». 

Или, например, у него в pull_request не закрыт никакой issues, то, в принципе, это нормально, если он опечатку поправил, а если он какую-то задачу сделал, то он, скорее всего, забыл, то пусть тоже поправит. Он не упадет, а просто предупредит. 

![](https://habrastorage.org/webt/zn/8y/lo/zn8ylovtsszjog6ar8oattundn8.jpeg)

https://www.npmjs.com/package/detect-secrets

Есть всякие полезности:

- Если у вас какие-то секреты используются в коде, и разработчики часто забывают, и коммитят туда AWS tokens, а вы их потом бьете по рукам, то есть такая автоматическая штука. Вы можете написать: «import detect-secrets» и бить их по рукам автоматически.
- Можно валидировать конфиги, что пришел валидный YAML, JSON.
- Проверять какие-то повторяющиеся действия, специфичные для вашего проекта. 

Вот зачем это нужно. 

![](https://habrastorage.org/webt/gn/t8/vh/gnt8vhrbviuzxuvytmc334rkj3w.jpeg)

Как это будет выглядеть? 

Сверху будут ошибки, что ты ошибся вот здесь и вот здесь. А какие-то поправил и отметил как resolve, они ставятся с зеленой галочкой, что это поправлено.

Посерединке – warnings, которые вы просто предупреждаете. 

Внизу – сообщение, если вы хотите чем-то поделиться. Например, я здесь пишу о том, как нужно работать: вот наши правила, вот наш styleguide, чтобы у людей была информация. 

![](https://habrastorage.org/webt/gj/p1/f_/gjp1f_rjk3vscpujvae5x0ugiae.jpeg)

Следующая штука – это вообще огонь – это Dependabot. Я везде его использую, всем его советую. Без него я уже не знаю, как жить. В 2021 году Dependabot - Security внутри платформы Github. Зайдите в `Settings` -> `Security & analysis` 

![](https://habrastorage.org/webt/2r/u2/au/2ru2auazxqraajp-su_7zxy3pqy.jpeg)

Что это такое? Это штука, которая умеет автоматически обновлять пакеты с уязвимостями. 

![](https://habrastorage.org/webt/ar/dx/_j/ardx_jdp3fha2ymnnph_z6f8i0m.jpeg)

Как это происходит? Вы заходите в GitHub и видите вот такую плашку, что в вашем пакете уязвимость. 

![](https://habrastorage.org/webt/dk/hl/ai/dkhlaih-mv1offb2yrlkt7s6ifm.jpeg)

Нажимаете кнопку.

![](https://habrastorage.org/webt/ck/k_/eh/ckk_ehvezltwnksmluukdaeuvl8.jpeg)

Он показывает, какие у вас уязвимости по разным уровням и т. д. 

![](https://habrastorage.org/webt/-y/nm/ug/-ynmugqdfpnsdwgduhlegjrlha0.jpeg)

Вы нажимаете еще одну кнопочку «Пожалуйста, поправь». 

![](https://habrastorage.org/webt/cv/c6/ff/cvc6ff4hs0krrimfg74horydyas.jpeg)

Он говорит: «Хорошо» и присылает вам pull_request, где все поправилось. Готово, все работает. Больше ничего делать не надо. Прислались pull_requests, вы их приняли, все, ваше приложение безопасно. 

Конечно, это не так работает, потому что нужно все проверять и тестировать, но маркетинговый ход довольно неплохой. 

![](https://habrastorage.org/webt/9j/x0/kt/9jx0kt2suisgbtwmwfbx33ggi4c.jpeg)

Как выглядит pull_request внутри?

Он меняет версию вашего пакета, меняет hashes и все остальное. 

![](https://habrastorage.org/webt/wc/--/gz/wc--gzlehvy7s7p61refpotdqiw.jpeg)

Но на самом деле он умеет обновлять не только проблемы безопасности, но и вообще любые пакеты. 

![](https://habrastorage.org/webt/aq/y1/_w/aqy1_wfr3f4zqig2-3myuajt9q8.jpeg)

Как это выглядит?

- Сначала он прочитает список ваших зависимостей, что у вас вообще установлено. Он поддерживает разные форматы, разные языки.
- Смотрит на наличие обновлений по этим пакетам.
- Делает много PR с изменениями одной зависимости. 
- Ждет прогона CI.
- Готово.

Вы можете указать определенные правила, что обновлять только минорные версии, только path или вообще все и т. д. В общем, куча всего. 

![](https://habrastorage.org/webt/vh/jg/vl/vhjgvld3ksphs9qbdh9wl16tcjq.jpeg)

Проверяем. Например, у нас есть JS и там установлены плагины для Eslint. И мы хотим, чтобы он нам его обновил. И он присылает нам обновление, где слева старый код, а справа новый код. Он бампнул версию, и мы теперь живем с новой версией. Круто. 

![](https://habrastorage.org/webt/ty/vx/by/tyvxbylqs7jesdbz3wbucwajrhu.jpeg)

Вот такие платформы поддерживаются, их очень много. Тут и Java, тут и Rust, тут и Python, тут и Docker, он умеет обновлять базовые образы, он умеет обновлять сами GitHub Actions. И даже странные языки типа Elm. 

![](https://habrastorage.org/webt/pp/fd/sf/ppfdsfekbmvt-8yke3qynsgynoc.jpeg)

Устанавливается все тоже в один клик. 

![](https://habrastorage.org/webt/9s/fr/z8/9sfrz8k7gac9pziwg1tkunu5fec.jpeg)

Заходите в GitHub Marketplace. 

![](https://habrastorage.org/webt/xl/03/qg/xl03qgltyrcvinqfmswieqsfzq8.jpeg)

И это абсолютно бесплатно для всех. Нажимаете «Установить». Готово. 

![](https://habrastorage.org/webt/ne/he/xe/nehexepoape_vhzx70eyehfyhbg.jpeg)

Дальше нам нужно что-то релизить. И когда мы раньше релизили, у нас было много боли, потому что релизили мы иногда с ошибкой версии, иногда забывали changelog написать. В общем, бывало всякое. Я думаю, что у всех у нас такое было. 

И я подумал, что надо как-то с этой проблемой бороться. А как с ней бороться? Автоматизировать. 

И для этого есть замечательный инструмент, который называется [semantic-release](https://github.com/semantic-release/semantic-release/tree/master/docs). Что это такое? Semantic-release состоит из нескольких шагов. 

![](https://habrastorage.org/webt/xs/pm/wm/xspmwmiy2wgihsh5hhxiagtmswo.jpeg)

И первый шаг – это валидация сообщения к коммитам.

![](https://habrastorage.org/webt/bg/db/2w/bgdb2wdhudxjtyvlxkodyzc6uvg.jpeg)

Вы создаете некую строку в RegEx, которая описывает, как вы можете описать сообщение к коммитам.

В нашем случае – это вот такая строка. Т. е. сначала мы можем указать, что какой-то коммит откатываем. Это опционально. Обычно мы редко откатываем коммиты, но иногда откатываем. Дальше мы указываем тип коммита. Это новая фича, новый fix, исправление документации или что-то неважное: отпечатки, refactoring или еще что-то, что не влияет на версию. 

А дальше мы указываем scope, например, API, infrastructure, documentation, в общем любые scopes, которые есть в вашем проекте. Они специфичные для проекта, вы можете придумать список своих собственных или использовать готовые. У Angular есть хороший guide, как эти scopes определять. 

И после коротенькое описание, на 50 символов максимум, что мы делаем в этом конкретном коммите, т. е. что он меняет. 

Последние символы – это какую конкретную issues он затрагивает, потому что нам важно знать для истории с какой конкретно issues этот коммит работал.  

![](https://habrastorage.org/webt/qb/cj/jb/qbcjjbhblrrxfivkcjbvvhkod3c.jpeg)

Дальше мы ставим новый action для того, чтобы проверять, что все это действительно так. Он называется «[Commitlint](https://github.com/marketplace/actions/commit-linter)».

![](https://habrastorage.org/webt/pf/gb/yz/pfgbyzlpwrnadnpmdk8u48qywyo.jpeg)

И когда мы будем присылать наши коммиты в GitHub, если мы будем не соответствовать этой версии, он будет падать и говорить: «Поправь, пожалуйста, сообщение к коммиту. Оно неверное, оно не соответствует стандарту. Вот стандарт». 

![](https://habrastorage.org/webt/y0/k9/59/y0k959lifhrshyve6yvvpzfhqvs.jpeg)

Теперь мы можем переходить ко второй части – это правильно релизить. Как вы понимаете, релиз основывается на том, какие коммиты и какие изменения были сделаны от предыдущей версии. 

![](https://habrastorage.org/webt/61/_p/ne/61_pne4giqmsgusk5oy1xpnvrrc.jpeg)

И теперь мы можем правильно их посчитать. Как?

Во-первых, у нас есть специальный конфигурационный файл. Это весь файл, в нем больше ничего нет. Он описывает нашу последовательность релиза софта. 

Сначала мы используем готовый компонент, который называется [semantic-release/commit-analyzer](https://github.com/semantic-release/commit-analyzer). Соответственно, мы анализируем те коммиты, которые мы написали. И смотрим, какие конкретно фичи и багфиксы мы сделали.

Если мы сделали только багфиксы, то мы релизим батч-версию. Если мы добавили новые фичи, то мы релизим минорную версию. А если мы сделали какие-то изменения, которые все сломали, то мы релизим мажорную версию. Он за нас автоматически это может понять. 

Дальше мы генерируем release-notes из наших коммитов, чтобы они были красивые и понятные. 

Генерируем changelog. Этот changelog мы коммитим обратно в репозиторий, чтобы он остался вместе с нами. И генерируем разные assets. 

Что такое assets? Это некая переменная. Это может быть docker-образы, статические файлы, бинарники в зависимости от того, что вы релизите. Есть плагины почти под все. 

Мы релизим docker-образы. Для нас assets – это docker-образ.

![](https://habrastorage.org/webt/tf/ht/xn/tfhtxn27ayxhoiwguutjza572m0.jpeg)

Дальше мы устанавливаем [semantic-release-action](https://github.com/marketplace/actions/semantic-release-action), который занимается непосредственно запуском этого благородства. 

![](https://habrastorage.org/webt/3m/dr/ec/3mdrecdtgo09-8r9opuohhalso4.jpeg)

Используем эту зеленую кнопку.

![](https://habrastorage.org/webt/0v/to/dq/0vtodqks9ngjqgfpvesdyfcj8qo.jpeg)

И вот такой pipeline 

![](https://habrastorage.org/webt/un/q_/dv/unq_dva3hewwo1tdxk-uy5qeqfk.jpeg)

Сначала мы копируем наш код, а потом указываем: `semantic-release-action` и что-то туда закладываем. Что это? Это секреты. Я привел два секрета в качестве иллюстрации. 

![](https://habrastorage.org/webt/7t/b8/ml/7tb8mlztea_kvdognc3a2qxueok.jpeg)

Например, мы релизим что-то на NPM внутренне или внешне – это неважно. И здесь два секрета: [GITHUB_TOKEN](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) и NPM_TOKEN.

![](https://habrastorage.org/webt/f3/wl/_w/f3wl_w22x4asxza3gn-rtnjk9ro.jpeg)

Что такое [GITHUB_TOKEN](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)? Это переменная, которая создается для нас автоматически на каждый PR. Мы о ней не заботимся, она такая, какая есть. Она хорошая. Мы ее не трогаем и нигде специально не делаем. 

![](https://habrastorage.org/webt/4s/hu/cs/4shucsg8i1h9-rnoi9m8d2mv1zy.jpeg)

А NPM_TOKEN – это наш собственный секрет, который мы обязаны правильно определить. Как?

![](https://habrastorage.org/webt/1f/1z/1c/1f1z1cxn3ji34szw-lkuy1xsvha.jpeg)

Заходим в настройки нашего репозитория. 

![](https://habrastorage.org/webt/1w/kp/_f/1wkp_fsgxio89omv3jr-giijo0m.jpeg)

Находим галочку «Settings». 

![](https://habrastorage.org/webt/7f/aj/nn/7fajnnbnj0uj8kfcz6lgjkrpwrm.jpeg)

Находим галочку «Secrets». 

![](https://habrastorage.org/webt/pg/ny/o3/pgnyo33570yb93sdxptshatq6ts.jpeg)

И потом находим имя секрета. Указываем имя секрета - `NPM_TOKEN`, значение секрета. Готово. 

![](https://habrastorage.org/webt/8y/bb/cf/8ybbcfhh4s8ontkrkugq3z2ogqk.jpeg)

В нашем случае, когда мы указываем NPM_TOKEN, то все начинает заводиться и паблишить пакеты в NPM. И такие changelogs у нас генерируются. 

Сверху номер версии, которая определяется автоматически. Дата. Далее список bug fixes, список features. Ссылки на коммиты. И issues, которые мы закрывает. 

![](https://habrastorage.org/webt/7j/gg/gv/7jgggv-bf2h8p4knjtkcctkwduw.jpeg)

И то же самое с changelog. Бот за нас автоматически этот changelog закоммитит обратно. Нам ничего не надо делать. 

![](https://habrastorage.org/webt/1r/wz/20/1rwz20zb1-lyiql0b7nz0buzr5s.jpeg)

Следующая большая сфера – это умные комментарии. Те, кто работает с GitLab плотно, наверное, знают, что в GitLab это называется «GitLab smart action». Там можно закрыть какой-то issues, написать CLOSE. Можно копию метадаты сделать какой-то конкретной issues в PR или наоборот. Там можно делать много всего слеш-командами, которые выглядят так:

![](https://habrastorage.org/webt/l0/le/1s/l0le1sqi0imqnjp273qh1jrsc7e.jpeg)

На GitHub такой штуки не было. И это было больно, потому что есть повторяющиеся действия, которые очень удобно делать этими GitLab smart actions. А на GitHub никак. 

Теперь с помощью GitHub Actions можно делать эти smart-комментарии. Как? При помощи переиспользуемых actions.

![](https://habrastorage.org/webt/2d/8b/lw/2d8blw98tfzr3eci_oddawvt6mm.jpeg)

Давайте посмотрим, как это выглядит. 

Например, мы хотим что-то отребейзить. Соответственно, мы пишем: `/rebase`. GitHub Actions поймал, что мы написали эту штуку. Он отребейзил наш PR. Все замечательно. Одной командой, которую мы сами не пишем, мы получили возможность использовать rebase внутри нашего репозитория. Круто. 

Для того чтобы его установить, заходим в marketplace - https://github.com/marketplace/actions/automatic-rebase. 

![](https://habrastorage.org/webt/wf/fp/dm/wffpdmjqh5dcihesyanxdkf65yc.jpeg)

Нажимаем кнопку. Установлено. 

Я это делаю для того, чтобы каждый раз показать, насколько это просто. 

![](https://habrastorage.org/webt/uv/l5/a0/uvl5a0nedpsfg_2dvc8ntdnqidg.jpeg)

Мы можем также отменить какой-то коммит. Случайно что-то не то запушили, подумали, что это лишнее, можем отменить. `/revert` – отменяем. 

![](https://habrastorage.org/webt/ws/th/wy/wsthwymyxyjf7qgvj8zm7zlzkcm.jpeg)

Мы говорим: `revert` и номер коммита, который отменяет». GitHub Actions немножко подумает и отменит предыдущий коммит. 

![](https://habrastorage.org/webt/8s/rq/nk/8srqnkovtqlpvs4euwboatl3c6s.jpeg)

Здесь нажимаем установить и готово. 

![](https://habrastorage.org/webt/dl/gu/rb/dlgurb1xgpsyet4r0sekthyu0ie.jpeg)

Теперь у нас есть возможность использовать готовые и писать свои собственные. Фишка в том, что мы можем придумывать свои собственные actions и умные сообщения к коммитам, которые будут делать всякое. Это же круто. 

Есть куча повторяющихся задач, когда нужно было сказать какому-то человеку или запустить какой-то внешний tool, то теперь это можно делать внутри комментариев на GitHub.

И мы можем попробовать сделать такой умный комментарий для того, чтобы деплоить. Представьте, что вы пишите: «/deploy», и все деплоится. Красота. 

![](https://habrastorage.org/webt/_r/jm/na/_rjmna5hyrsdrckxmn3yugkpdlk.jpeg)

Давайте посмотрим, как это может выглядеть. 

Деплой – это всегда сложно, потому что деплоить нужно разное, в разные места, с разными правилами. По-разному это все работает с точки зрения организации, т. е. как, когда и т. д. 

![](https://habrastorage.org/webt/hv/u7/t7/hvu7t7ju-nermdkkmee2nal3_vk.jpeg)

Я покажу, что есть готового и не буду концентрироваться на конкретных примерах. Я расскажу про свой опыт и покажу, что есть. 

Готовые провайдеры:

- [AWS](https://github.com/aws-actions), [Azure](https://github.com/Azure/actions).
- [Zeit](https://github.com/marketplace/actions/zeit-now-deployment)/ Now, [Heroku](https://github.com/marketplace/actions/deploy-to-heroku).
- [K8S](https://github.com/marketplace/actions/deploy-to-kubernetes-cluster), [DockerSwarm](https://github.com/marketplace/actions/docker-swarm-deploy), [Dokku](https://github.com/marketplace/actions/dokku-deploy-action).
- [Ansible](https://github.com/marketplace/actions/run-ansible-playbook), [Chef](https://github.com/marketplace/actions/actionshub-chef-install), [Terraform](https://github.com/marketplace/actions/hashicorp-setup-terraform).
- [Rsync](https://github.com/marketplace/actions/rsync-deployments-action). Можно даже Rsync использовать, уже есть готовый action для этого. 
- [Docker registry](https://github.com/marketplace/actions/build-and-push-docker-images). Можно прямо в Docker registry все это релизить, если у вас дальше pipeline настроен от этого места. 

Вариантов куча. Это все есть уже готовое. Все это плюс-минус как-то работает. Я думаю, что все с багами, но тем не менее. Вы можете взять и поставить, например, для Azure или Terraform и деплоить.

![](https://habrastorage.org/webt/re/lo/u6/relou6r8xxxaxlgcotomfunhcvk.jpeg)

Естественно, есть куча вспомогательных средств. Какие? Если это Ansible, то вы можете прогнать [Ansible-lint](https://github.com/ansible-community/ansible-lint), если Terraform, то можете прогнать Format, если это Rsync, то вы можете посмотреть доступность того места, куда вы деплоите предварительно. Т. е. есть куча tooling вокруг, а именно не только сам процесс деплоя, но и предварительные шаги, чтобы проверить, что все правильно. 

![](https://habrastorage.org/webt/bf/tp/dn/bftpdndrjz9oebrr6uo8pkghuri.jpeg)

Приведу пример с [Docker](https://github.com/marketplace/actions/build-and-push-docker-images), как деплоим мы, и как я могу наш pipeline сейчас изменить и сделать при помощи GitHub Actions.

![](https://habrastorage.org/webt/mm/34/d-/mm34d-jzevd6jjwh2ux0ikervno.jpeg)

Мы возьмем готовый GitHub Action, который деплоит докер-образы.

![](https://habrastorage.org/webt/vy/l6/pl/vyl6plmc2aub3mlyzwtajfr6b2c.jpeg)

Нажмем кнопочку. И все заработает.

![](https://habrastorage.org/webt/xx/ux/8g/xxux8gdseojohuvmvm-tqsd-fdw.jpeg)

Первый шаг – копируем код.

![](https://habrastorage.org/webt/6q/j3/sr/6qj3srbgdbuat-re3zg5tlgibzk.jpeg)

Второй шаг – делаем publish to registry.

Как мы это делаем? Мы указываем имя, в какой registry мы пушим наш DOCKER_USERNAME, DOCKER_PASSWORD и готово. 

Таким образом, GitHub сам соберет образ, сам его запушит и все сделает хорошо. 

![](https://habrastorage.org/webt/rw/d0/3h/rwd03hdm4iitrjza41sffpl_krk.jpeg)

Как у нас это выглядит на одном из проектов?

![](https://habrastorage.org/webt/nf/2z/ds/nf2zdsu6tutl1zyesnulevywc5m.jpeg)

На одном из проектов у нас есть GitLab. У него есть registry. Из registry [Drone](https://www.drone.io/) забирает образ, начинает делать свои небольшие собственные действия. И потом куда-то деплоит.

![](https://habrastorage.org/webt/5j/wo/de/5jwodetkbpsgnlamhjbogvvewcw.jpeg)

Теперь это можно будет легко заменить на GitHub. Теперь у нас будет только GitHub. У него есть собственный registry. Тоже самое будет забирать Drone и куда-то деплоить. Красота. 

![](https://habrastorage.org/webt/gx/-a/6l/gx-a6lztog_dr6rvlhofi-abofm.jpeg)

Естественно, можно триггерить другие CI. 

![](https://habrastorage.org/webt/2y/cb/vs/2ycbvs_w1njkwzamc84n3z5gpxu.jpeg)

Какие? Также как мы триггерим Drone, мы можем триггерить:

- GitLab CI,
- Azure,
- Travis,
- Drone,
- Circle.

В общем, action, чтобы триггернуть другую CI, тоже есть. Иногда это бывает важной частью деплоя. 

![](https://habrastorage.org/webt/1b/k9/jn/1bk9jnlxbr1gu3ooui-os8yxorg.jpeg)

Теперь давайте рассмотрим пример с Terraform от начала до конца.

![](https://habrastorage.org/webt/pn/qx/9y/pnqx9yf8fxfjri2m-uv6itszekk.jpeg)

Как это будет выглядеть? У нас есть workflow, который работает на каждый PR. Допустим, мы деплоим каждый PR. 

![](https://habrastorage.org/webt/ge/hc/w7/gehcw7istxbudllvezgkb8lcsrs.jpeg)

У нас будет первый action, который будет говорить: «Пожалуйста, проверь, что форматирование Terraform хорошее», потому что мы хотим держать наши файлы в консистентности, чтобы они все были красивые.

![](https://habrastorage.org/webt/c-/xo/wm/c-xowmrtfcpvjn-yw9bskjfyunq.jpeg)

Как это будет выглядеть? Когда мы будем использовать этот action, то GitHub bot нам будет говорить: «Ты скобку здесь не там поставил. Иди и поправь скобку, а потом деплой». 

![](https://habrastorage.org/webt/bv/gb/xb/bvgbxbnmpe6popzqrukmqjtvyfc.jpeg)

Мы говорим: «Хорошо. Формат проверен, теперь создай мне базовую рабочую папку для Terraform». Создали.

![](https://habrastorage.org/webt/ht/eu/ff/hteuffrun-azbkg0ytrellkfeme.jpeg)

И дальше говорим: «Пожалуйста, провалидируй, что у меня все правильно. Что у меня есть все переменные, все корректно, все работает». 

![](https://habrastorage.org/webt/8u/dc/qi/8udcqihjvqdyzen0-fvitqr_bak.jpeg)

Он говорит: «Хорошо», смотрит, а переменной myvar нет. Она нужна для деплоя, а ее нет. И говорит: «Пожалуйста, поправь. Я деплой делать пока не буду».

![](https://habrastorage.org/webt/du/n3/4m/dun34mno_kdax95b49oqgccxoq4.jpeg)

Говорим: «Хорошо», поправили переменную. И планируем деплой для Terraform, говорим: «Plan».

![](https://habrastorage.org/webt/kz/cd/rv/kzcdrve4az5adtqcp4m73lohc5k.jpeg)

И GitHub bot пишет вот такую простыню с его plan. Он говорит, что будет что-то создавать и ресурс вот такой-то. 

![](https://habrastorage.org/webt/wu/qb/j3/wuqbj3gvvz0sf2wexvveiw_d0ki.jpeg)

Круто! И осталось последнее действие. Чтобы применить этот plan, мы используем еще один готовый action, который называется «apply».

![](https://habrastorage.org/webt/vj/jv/o8/vjjvo8ozhj66ro7k2sjvnwwsya8.jpeg)

И Terraform делает apply, говорит: «Я Null сервис создал. Все хорошо. Вот тебе log».

Готово, наш проект при помощи Terraform задеплоен из GitHub Actions.

![](https://habrastorage.org/webt/7q/6y/ua/7q6yuaefo4wik0bzzda4c9razea.jpeg)

Вот ссылка на официальную штуку для Terraform - https://github.com/hashicorp/terraform-github-actions. Более актуальная ссылка - https://learn.hashicorp.com/tutorials/terraform/github-actions. Она поддерживается HashiCorp. И я думаю, что можно ею смело пользоваться.

![](https://habrastorage.org/webt/g8/2s/d-/g82sd-qlgcduugxcorjsqxcdwja.jpeg)

Я бы здесь еще вставил несколько слайдов про управление разработчиками, потому что я очень люблю, когда автоматизируется процесс управления, т. е. распределение задач, приоритизация, создание новых и т. д. 

А мне организаторы говорят: «Какое управление разработчиками? Тут умные люди сидят».



И еще много всего другого. Их тех примерах, которые я показал, надеюсь, вы уловили главное, что автоматизировать можно все. И это очень просто: деплой, проверки, умные комментарии, можно делать всякие интеграции с [Sentry](https://github.com/getsentry/sentry), с системами мониторинга и т. д. Это отличная платформа для того, чтобы заниматься простой автоматизацией

![](https://habrastorage.org/webt/zr/xx/gn/zrxxgn3vowsiisk2ssnacxbrzam.jpeg)

И хочу напомнить, что автоматизировать – это просто. Это главная ценность моего доклада. Я хочу передать вам идею, что автоматизировать – это не больно, не сложно и не надо лучше это делать руками. Нет, это просто и удобно. Пожалуйста, автоматизируйте, у вас есть для этого все инструменты и теперь все знания. 

Важные несколько объявлений.

Во-первых, это бета! GitHub Actions – это бета. 

Но сегодня утром я открыл свою почту, и мне пришло письмо счастья. Здесь очень много текста. 

Ключевое: 13 ноября это станет доступно для всех. Буквально через две недельки это доступно для всех.

А теперь это есть в публичной бете. Если хотите, то вы можете попробовать прямо сейчас. Зарегистрируйтесь на сайте, нажмите на кнопочку, вам сразу придет приглашение. 

Но с 13 ноября это будет доступно для всех, что очень круто. Я думаю, что многим из вас сегодня такое письмо пришло. 

![](https://habrastorage.org/webt/x6/mv/nu/x6mvnuxqkvs6fg9ccc3zcxufffg.jpeg)

Есть естественно проблемы. Если бы я про них не сказал, то было бы не очень честно.

Проблемы:

- Во-первый не работает кеш для зависимостей. Это спорная точка зрения, потому что кто-то считает, что зависимости кешировать не надо; кто-то сравнивает, сколько с кешем, сколько без, но по моим личным сравнениям Travis работает значительно быстрее с кешем, чем GitHub Actions без кеша. Ребята из GitHub об этом знают, они над этим работают.
- Самая большая проблема, которая делает для меня невозможной работу в open sourse, в том, что они никак не могут поправить проблему с origin/fork. Если ты работаешь внутри одного репозитория, то это все прекрасно работает, потому что у тебя есть права, потому что GITHUB_TOKEN, который создался, он правильный. А если человек прислал тебе PR из fork, то GITHUB_TOKEN у него от fork и к твоему репозиторию он не подходит. И поэтому, когда он пытается все эти действия выполнить он говорит: «Не достаточно прав, извините». И это большая проблема. Это достаточно частый паттерн, когда у тебя есть оригинальный репозиторий. Ты его форкаешь из него присылаешь PR. Пока это не работает, ребята тоже об этом знают. Это, действительно, для них очень важная проблема. Они ее пофиксят, я надеюсь, до 13 ноября.
- Относительно мало готовых компонентов. Тут важное слово – это относительно. Потому что относительно GitLab – их до фига, относительно того, что хочется – их мало. Хочется, чтобы на каждый чих был готовый компонент, чтобы ты собирал себе pipeline из кубиков и все работало. Пока, к сожалению, это не так. Я несколько actions параллельно делал для того, чтобы это все красиво работало. 
- Не у всех есть доступ. 13.11 эту проблему исправят. 
- Шутка: тесты за вас никто не напишет! Какой бы классной платформа не была – все это пока без автоматики. 

![](https://habrastorage.org/webt/bg/jd/p9/bgjdp9-tbsyi1rxt7ewfu-78ba0.jpeg)

Здесь есть несколько полезных ссылок: 

- Первая ссылка – это основная точка входа в документацию по GitHub Actions. 
- Вторая ссылка – это список тех events, которые запускают конкретный workflow в GitHub Actions. 
- Третья ссылка – это список классных GitHub Actions, которые уже есть. Это не все, это список лучших. Все в Marketplace.

![](https://habrastorage.org/webt/xi/to/c_/xitoc_l2qjzqpewwhwpav5cu_pi.jpeg)

Специально для конференции я подготовил 2 GitHub Actions, которые мы сегодня смотрели:

- Первый – это запрещать людям материться. Это забавно. 

- Второй – проверяет python-код на наличие проблем. Это полезно. 

Первый написан для демонстрации при помощи JS и всяких разных готовых компонентов, а второй написан просто как docker-файл и метаинформация.

![](https://habrastorage.org/webt/45/ut/y-/45uty-z-5hnh6fapyoujynagw5u.jpeg)

Последнее объявление. Сегодня у GitHub Actions будет второй доклад. Он будет не про GitHub Actions, но они там тоже будут. Если вам интересно, как взаимодействует GitHub Actions и Jenkins, приходите на доклад Олега. Сильно советую, Олег знает, о чем говорит. Будет интересно. 

![](https://habrastorage.org/webt/jq/jm/cl/jqjmclmhrhzrlpjyibeollsb5cc.jpeg)

Минутка маркетинга: внизу есть ссылка на мой блог. Если вам интересно почитать про автоматизацию, про линтеры, про статический анализ, про сложности, про Python, то вы знаете, куда обратиться. Можете подписаться на GitHub, там тоже все это дублируется. 

Спасибо! Ваши вопросы?

*Ты упоминал про action обновления зависимости и сказал, что он на каждую зависимость делает отдельный PR. Можно ли в один PR это как-то объединить?*

С Dependabot нельзя, но есть другая штука, которая называется [RenovateJS](https://github.com/renovatebot/renovate). Я не уверен, с какими конкретно языками она работает, но у нее есть фича, что она обновляет сразу все одним. На мой взгляд, это не очень удобно, потому что, если вы обновляете сразу несколько, то вы не понимаете, где сломалось. Т. е. что-то сломалось, а где конкретно – вы не понимаете. По одному, на мой личный взгляд, удобнее.

*Возможно ли как-то пропускать actions? По факту у тебя есть steps или несколько actions в step. И ты хочешь сделать skip.*

Хороший вопрос. Не знаю, пока не пробовал. Но, я думаю, что как-то должно быть сделано, иначе это будет большой проблемой. 

*Какая альтернатива для BB script run only?*

Я не знаю, я с bitbucket 100 не работал. 

*Когда уже Jenkins умрет?* 

Это к коллеге вопрос, это не ко мне. 

*Я увидел, что мы можем писать какие-то actions на языке программирования, можем bash скрипт в Run написать. И как я понял, ты можешь, допустим, на JS написать какой-то action, но я не понял, на каких еще языках это можно написать.*

Ты можешь написать на любом языке, который можно запустить и получить статус код. Неважно на каком языке. Если ты его в Docker запакуешь, то гарантированно на любом. Но если ты хочешь использовать готовые компоненты, чтобы работать с кешем, с API и т. д., то это только Java Script/TypeScript. Вот эти два языка готовы для того, чтобы делать все быстро и хорошо. 

*Мы весь доклад говорили с точки зрения бесплатности. Ты показывал ценник, но не понятно, сколько. Может быть, у вас есть приватные репы, на которых вы можете сказать, как это можно по деньгам в месяц получается.*

Хороший очень вопрос. Я пока не могу сказать, потому что у нас пока приватные репы хранятся на GitLab. Я думаю, что некоторое время точно еще будут там храниться. Но в связи с последними гитлабовскими событиями мне хочется переехать от них, потому что телеметрию всовывать не понятно куда – это не очень хороший шаг. 

На GitHub меня очень прельщает идея, что и open source, и закрытые проекты будут в одном месте. Это для меня будет очень просто, потому что сейчас, по сути, мне нужно две инфраструктуры поддерживать: одну для GitHub, другую для GitLab. Это проблематично. 

*Тогда, может быть, ты знаешь кого-то, кто использует на приватных репах GitHub Actions из community?*

Нет, я пока таких людей не знаю.

*Получается, что сторонние CI не нужны. Но, как я видел, ты Drone еще использовал?*

Они не нужны. Если у вас есть какие-то сервисы, которые завязаны на этом, то можете их триггерить из GitHub CI. И это полезно в некоторых случаях. Например, у нас Drone занимается сборкой Docker и дополнительными проверками на стороне клиента. Мы пушим наш docker-образ, он его дополнительно проверяет, что там нет его собственных проблем, т. е. некий аудит происходит. И этот Drone, по сути, не наш. Это просто такая точка, которая работает, чтобы передать куда-то дальше слово, которое я обещал не говорить.

*Все бесплатно. А если все платно? У тебя какой-то action стриггерился, начал работать, но начались считаться деньги, пока он работает. И ему нужно делегировать задачу в стороннюю штуку и дождаться ответа. И чтобы он не ждал синхронной, и мы не платили, пока он ждет, то мы хотим вернуть результат. Можно ли как-то это сделать?*

Точно нет, потому что action – это синхронная штука. Делать асинхронные actions – это было бы неправильно. pipeline – это все-таки синхронная история. 

*Т. е. если мы делегируем куда-то надолго, то стоит подумать – нужно ли вообще делегировать?* 

Да. И нужно ли дожидаться ответа? Т. е. может быть нам ответ не важен: делегировали и делегировали, а на той стороне уже разберемся. 

*Ты показал, как делать кастомные actions, но может быть такая ситуация, когда тебе нужно сделать кастомный event, например. Т. е. у тебя есть корневой event, но в нем есть параметры. И ты как-то хочешь его фильтровать.* 

Свои events делать нельзя. Ты можешь настроить webhook. А по поводу выборки по типам events, то – да, это есть. У PR есть: открыт, закрыт, изменен, удален, назначен reviewer, изменен код и т. д. Там есть подтипы, которые помогают делать выборку. 

*Ты показывал кастомный UI и у тебя появилась какая плашка, кнопка. Действия на эту кнопку тоже что-то триггерят. Там есть отдельная библиотека, чтобы рисовать эти UI и компонеты, либо как происходит это кастомное рисование внутри репозитория, когда ты action подключил?*

Они UI кастомный отключили. Кстати, я хотел об этом пошутить, но забыл. Есть первая версия GitHub Actions, которая написана на HSL. Там все совсем по-другому. Но о ней мы не говорим, ее не было. Это была ошибка, мы ее пропустили. Рисование было там. Там была идея, что ты можешь график делать, соединять стрелочками и т. д. Но это не зашло, потому что это аудитория разработчиков. А для аудитории разработчиков это проще в YAML описать и проверять, чем руками туда-сюда перетаскивать. Я думаю, что поэтому они это свернули. 

*Я имел в виду несколько другое. Мы по зависимостям что-то искали, и там сверху была какая-то кнопка толи смержить, толи подтвердить это все, толи это была какая-то стандартна кнопка внутри PR. Например, у нас есть стандартная сущность: merge_request или pull_request.*

Нет, там никаких дополнительных UI нет, т. е. ничего не меняется. Появляется только новая вкладочка «Action». И внутри action новый UI, а все остальное не меняется. 

*Ты рассказывал про Danger и ты привел Rube, JS, Swift. И я не понял – это языки, на ка которых есть SDK?*

Да, это языки, на которых ты можешь писать конфигурационный файл для Danger. Самое смешное, что он имеет разный бэкенд. Этот разный бэкенд работает с разными плагинами. Иногда ты должен выбирать по тем плагинам, которые тебе нужны. Естественно, javascript’овые имеют больше плагинов к JS, ruby’шные имеют больше плагинов к Ruby, а swift’овые – не знаю. 

*Расскажи подробней про сам Danger. Если есть SDK есть на этих языках, но я хочу валидировать какие-то другие проекты. Что еще можно с помощью него?*

Danger – это просто язык программирования, ты можешь делать все, что угодно в нем. Другое дело, что он тебе в контекст передает какие-то метаданные о текущем контексте: в PR, issues и т. д. Ты можешь делать все. 

Если тебе нужно запустить какую-то утилиту и проверить статус код, то ты говоришь: «process.execSync» в JS и, соответственно, выполняешь какой-нибудь execSync, смотришь статус и что-то делаешь. 

Если тебе нужно проверить контексты, то ты просто там делаешь if, ты импортируешь 2 дополнительные функции: файл, warning и т. д. И ты их используешь. Он наполняет себя этими штуками и в конце рендерит то, что тебе нужно внутри PR, чтобы все люди увидели, что вообще происходит. 

На самом-то деле там нет никакой автоматизации суперширокой. Для меня Danger – это инструмент, который умеет проверять то, что не умеют проверять линтеры. Когда статические тесты прошли, я проверяю только корректность оформления, что человек правильно все правил, что у него правильно все заполнено, что он выполнил все нормативные правила по доставке кода в виде PR. 

А именно суровой проверки внутри Danger я не делаю, потому что есть другие, более подходящие для этого инструменты. 

*Получается, что Danger – это такая верхнеуровневая штука?*

Да. И легче импортировать какой-то существующий JS-код и из него что-то сделать. Если есть линтер, то прогнал линтер. 

*Как защититься от открытия секретов, если в Github action написать `echo secrets`?*

Это очень хороший вопрос. Я задавал этот вопрос и GitLab, и GitHub. И внятного ответа не получил. Почему я не использую сейчас с кастомным GITHUB_TOKEN, который имеет доступ ко всему? Я не знаю, как его люди будут использовать. Т. е. я сейчас выложу token, и люди – хоп, начнут его использовать, чтобы писать от меня комментарии, что я люблю, Go, например. Нет, я не доверяю этому. Я хочу какой-то token, который обладает ограниченными правами и желательно, чтобы он мог запускаться только из GitHub Actions. А сейчас это явная проблема. Люди будут использовать это не по назначению.

Но другое дело, если это open source и у человека нет желания точечно именно этого человека взломать и что-то сделать, то всем по фиг. Лежит там чей-то пароль и лежит. Никому не интересно. Я такие проекты видел. Я даже думал им прислать такой PR, чтобы показать, что есть уязвимость. А потом подумал: «А зачем? Работают ребята и работают, зачем им мешать?». 


P.S. Чат по Github action - https://t.me/github_action_ru
