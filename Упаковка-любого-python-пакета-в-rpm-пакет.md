Бывают ситуации, когда нужно упаковать Python пакет c его зависимостями в rpm пакет.

В этом посте будут рассмотрены 2 варианта: FPM и RPMVENV.

В обоих случаях необходимо создание файла requirements.txt.

Файл requirements.txt должен находится в той же директории где выполняется команда упаковки python пакета в rpm пакет.

Также при создании rpm пакет и его установке в системе должен быть linux пользователь airflow.

**Часть 0. Создание файла requirements.txt**

Создавать будем на примере apache-airflow.

Установим зависимости RPM для apache-airflow.

```
sudo yum install -y python3-devel python3-pip gcc gcc-c++ cyrus-sasl-devel krb5-devel
```

Установка apache-airflow используя PIP.

```
sudo pip3 install apache-airflow[postgres,redis,hdfs,hive,kerberos,ldap,ssh]==1.10.12 --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.6.txt"
```

В квадратных скобках вы можете использовать Python зависимости для apache-airflow, которые нужно тоже нужно установить.

После установки apache-airflow смотрим список установленных Python пакетов с помощью команды.

```
pip3 freeze > requirements.txt
```

**Часть 1. Использование FPM**

**Часть 1.1. Попытка вызвать модуль virtualenv при использовании FPM**

Устанавливаем RPM зависимости для fpm.

```
sudo yum install -y ruby-devel gcc make rpm-build rubygems
```

Устанавливаем Python и Ruby зависимости для fpm.

```
gem install --version 1.12.2 --user-install ffi
gem install --version 1.6.0 --user-install git
pip3 install --user virtualenv-tools 
```

Устанавливаем fpm.

```
gem install --no-document fpm
```

Собираем rpm пакет airflow c использованием модуля virtualenv.

```
fpm  --verbose  -s virtualenv -t rpm --name airflow -v 1.10.12 --prefix /home/airflow requirements.txt
...
Successfully installed Babel-2.8.0 Flask-1.1.2 Flask-Admin-1.5.4 Flask-AppBuilder-2.3.4 Flask-Babel-1.0.0 Flask-Caching-1.3.3 Flask-JWT-Extended-3.24.1 Flask-Login-0.4.1 Flask-OpenID-1.2.5 Flask-SQLAlchemy-2.4.4 Flask-WTF-0.14.3 Jinja2-2.11.2 Mako-1.1.3 Markdown-2.6.11 MarkupSafe-1.1.1 PyHive-0.6.3 PyJWT-1.7.1 PyNaCl-1.4.0 PyYAML-5.3.1 Pygments-2.6.1 SQLAlchemy-1.3.19 SQLAlchemy-JSONField-0.9.0 SQLAlchemy-Utils-0.36.8 WTForms-2.3.3 Werkzeug-0.16.1 alembic-1.4.2 apache-airflow-1.10.12 apispec-1.3.3 argcomplete-1.12.0 argparse-1.4.0 attrs-19.3.0 bcrypt-3.2.0 cached-property-1.5.1 cattrs-1.0.0 certifi-2020.6.20 cffi-1.14.2 chardet-3.0.4 click-6.7 colorama-0.4.3 colorlog-4.0.2 configparser-3.5.3 croniter-0.3.34 cryptography-3.0 defusedxml-0.6.0 dill-0.3.2 dnspython-1.16.0 docutils-0.16 email-validator-1.1.1 flask-swagger-0.2.14 funcsigs-1.0.2 future-0.18.2 graphviz-0.14.1 gunicorn-20.0.4 hmsclient-0.1.1 idna-2.10 importlib-metadata-1.7.0 iso8601-0.1.12 itsdangerous-1.1.0 json-merge-patch-0.2 jsonschema-3.2.0 lazy-object-proxy-1.5.1 ldap3-2.8 lockfile-0.12.2 marshmallow-2.21.0 marshmallow-enum-1.5.1 marshmallow-sqlalchemy-0.23.1 natsort-7.0.1 numpy-1.19.1 pandas-1.1.0 paramiko-2.7.1 pendulum-1.4.4 prison-0.1.3 protobuf-3.13.0 psutil-5.7.2 psycopg2-binary-2.8.5 pyasn1-0.4.8 pycparser-2.20 pykerberos-1.2.1 pyrsistent-0.16.0 pysftp-0.2.9 python-daemon-2.2.4 python-dateutil-2.8.1 python-editor-1.0.4 python-nvd3-0.15.0 python-slugify-4.0.1 python3-openid-3.2.0 pytz-2020.1 pytzdata-2020.1 redis-3.5.3 requests-2.24.0 requests-kerberos-0.12.0 sasl-0.2.1 setproctitle-1.1.10 six-1.15.0 snakebite-2.11.0 sshtunnel-0.1.5 tabulate-0.8.7 tenacity-4.12.0 text-unidecode-1.3 thrift-0.13.0 thrift-sasl-0.4.2 typing-3.7.4.3 typing-extensions-3.7.4.2 tzlocal-1.5.1 unicodecsv-0.14.1 urllib3-1.25.10 zipp-3.1.0 zope.deprecation-4.4.0 {:level=>:info}
Traceback (most recent call last): {:level=>:info}
  File "/home/airflow/.local/bin/virtualenv-tools", line 11, in <module> {:level=>:info}
    load_entry_point('virtualenv-tools==1.0', 'console_scripts', 'virtualenv-tools')() {:level=>:info}
  File "/usr/lib/python3.6/site-packages/pkg_resources/__init__.py", line 476, in load_entry_point {:level=>:info}
    return get_distribution(dist).load_entry_point(group, name) {:level=>:info}
  File "/usr/lib/python3.6/site-packages/pkg_resources/__init__.py", line 2700, in load_entry_point {:level=>:info}
    return ep.load() {:level=>:info}
  File "/usr/lib/python3.6/site-packages/pkg_resources/__init__.py", line 2318, in load {:level=>:info}
    return self.resolve() {:level=>:info}
  File "/usr/lib/python3.6/site-packages/pkg_resources/__init__.py", line 2324, in resolve {:level=>:info}
    module = __import__(self.module_name, fromlist=['__name__'], level=0) {:level=>:info}
  File "/home/airflow/.local/lib/python3.6/site-packages/virtualenv_tools.py", line 50 {:level=>:info}
    print 'A %s' % script_filename {:level=>:info}
               ^ {:level=>:info}
SyntaxError: Missing parentheses in call to 'print'. Did you mean print('A %s' % script_filename)? {:level=>:info}
Process failed: virtualenv-tools failed (exit code 1). Full command was:["virtualenv-tools", "--update-path", "/home/airflow"] {:level=>:error}
```

[Issue](https://github.com/jordansissel/fpm/issues/1758) по этой ошибке заведена.

**Часть 1.2. Используем модуль dir при использовании FPM**

Устанавливаем RPM зависимости для fpm и virtualenv.

```
sudo yum install -y ruby-devel gcc make rpm-build rubygems git
sudo yum install -y python3-devel python3-pip gcc gcc-c++ cyrus-sasl-devel krb5-devel
sudo yum -y install epel-release rpmdevtools wget
```

Устанавливаем Python и Ruby зависимости для fpm.

```
gem install --version 1.12.2 --user-install ffi
gem install --version 1.6.0 --user-install git
pip3 install --user virtualenv
```

Устанавливаем fpm.

```
gem install --no-document fpm
```

Создаем virtualenv окружение для установки в него airflow.

```
mkdir -p /home/airflow/airflow
python3 -m venv airflow
source /home/airflow/airflow/bin/activate
```

Устанавливаем airflow в virtualenv окружение.

```
python3 -m pip install apache-airflow[postgres,redis,hdfs,hive,kerberos,ldap,ssh]==1.10.12 --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.6.txt"
deactivate
```

Команда deactivate выходит из virtualenv окружения.

Упаковываем virtualenv окружение с установленным airflow в rpm пакет.

```
fpm -s dir -t rpm -n airflow -v 1.10.12 --rpm-auto-add-directories --no-rpm-use-file-permissions --rpm-user airflow --rpm-group airflow --prefix /home/airflow/airflow -C /home/airflow/airflow
```

Получается rpm пакет. В части 3 его можно установить и запустить airflow из virtualenv окружения.

**Часть 2. Использование RPMVENV**

Устанавливаем RPM зависимости для rpmvenv и virtualenv.

```
sudo yum install -y python3-devel python3-pip gcc gcc-c++ cyrus-sasl-devel krb5-devel
sudo yum -y install epel-release rpmdevtools wget
```

 Устанавливаем Python зависимости для rpmvenv.

```
pip3 install --user virtualenv
```

Устанавливаем fpm.

```
pip3 install --user rpmvenv
```

Копируем requirements.txt в директорию, где будет запускаться rpmvenv.

Запускаем rpmvenv с параметрами.

```
echo "{}" > conf.json && \
LC_ALL=en_US.UTF-8 rpmvenv conf.json \
--core_name="apache-airflow" \
--core_version="1.10.12" \
--core_license="ASL 2.0" \
--core_summary="apache-airflow" \
--extensions_enabled="blocks,python_venv" \
--python_venv_name="apache-airflow" \
--python_venv_path="/home/airflow" \
--python_venv_strip_binaries="false" \
--blocks_desc="apache-airflow" \
--python_venv_require_setup_py="false" \
--file_permissions_user="airflow" \
--file_permissions_group="airflow"
```

Получается rpm пакет. В части 3 его можно установить и запустить airflow из virtualenv окружения.

**Часть 3. Запуск Python пакета из virtualenv окружения**

Устанавливаем apache-airflow-1.10.12-1.x86_64.rpm

```
sudo yum install -y apache-airflow-1.10.12-1.x86_64.rpm
```

Входим в virtualenv

```
source /home/airflow/apache-airflow/bin/activate
```

Запускаем airflow

```
airflow initdb
airflow webserver -p 8080
```

Выходим из virtualenv

```
deactivate
```
