## Введение

Sonatype Nexus – интегрированная платформа управления, с помощью которой разработчики могут проксировать и управлять зависимостями Java(maven), образами Docker, Python, Ruby, NPM, Bower, RPM-пакетами и gitlfs, а также распространять свое программное обеспечение. 

### Зачем нужен Sonatype Nexus?

- Для хранения приватных артефактов;

- Для кеширования артефактов, которые скачиваются из интернета;

  

### Артефакты поддерживаемые в базовой поставке Sonatype Nexus:

- Java, Maven (jar)
- Docker
- Python (pip)
- Ruby (gem)
- NPM
- Bower
- Yum (rpm)
- gitlfs
- Raw
- Apt (deb)
- Go
- Nuget

### Артефакты поддерживаемые сообществом:

- Composer
- Conan
- CPAN
- ELPA
- Helm
- P2
- R

## Установка Sonatype Nexus используя https://github.com/ansible-ThoTeam/nexus3-oss

### Требования

- Прочитайте про использование ansible в интернете.
- Установите ansible `pip install ansible` на рабочей станции где запускается playbook. 
- Установите [geerlingguy.java](https://galaxy.ansible.com/geerlingguy/java/) на рабочей станции где запускается playbook. 
- Установите [geerlingguy.apache](https://galaxy.ansible.com/geerlingguy/apache/) на рабочей станции где запускается playbook. 
- Эта роль протестирована на CentOS 7, Ubuntu Xenial (16.04) and Bionic (18.04), Debian Jessie and Stretch
- `jmespath` библиотека должна быть установлена на рабочей станции где запускается playbook. Для установки выполните `sudo pip install -r requirements.txt`
- Сохраните файл playbook (пример ниже) в файл nexus.yml
- Запустите установку nexus `ansible-playbook -i host nexus.yml`

### Пример ansible-playbook для установки nexus без LDAP с репозиториями Maven (java), Docker, Python, Ruby, NPM, Bower, RPM и gitlfs.

```
---
- name: Nexus
  hosts: nexus
  become: yes

  vars:
    nexus_timezone: 'Asia/Omsk'
    nexus_admin_password: "admin123"
    nexus_public_hostname: 'apatsev-nexus-playbook'
    httpd_setup_enable: false
    nexus_privileges:
      - name: all-repos-read
        description: 'Read & Browse access to all repos'
        repository: '*'
        actions:
          - read
          - browse
      - name: company-project-deploy
        description: 'Deployments to company-project'
        repository: company-project
        actions:
          - add
          - edit
    nexus_roles:
      - id: Developpers # maps to the LDAP group
        name: developers
        description: All developers
        privileges:
          - nx-search-read
          - all-repos-read
          - company-project-deploy
        roles: []
    nexus_local_users:
      - username: jenkins # used as key to update
        first_name: Jenkins
        last_name: CI
        email: support@company.com
        password: "s3cr3t"
        roles:
          - Developpers # role ID here
    nexus_blobstores:
      - name: company-artifacts
        path: /var/nexus/blobs/company-artifacts
    nexus_scheduled_tasks:
      - name: compact-blobstore
        cron: '0 0 22 * * ?'
        typeId: blobstore.compact
        taskProperties:
          blobstoreName: 'company-artifacts'

    nexus_repos_maven_proxy:
      - name: central
        remote_url: 'https://repo1.maven.org/maven2/'
        layout_policy: permissive
      - name: jboss
        remote_url: 'https://repository.jboss.org/nexus/content/groups/public-jboss/'
      - name: vaadin-addons
        remote_url: 'https://maven.vaadin.com/vaadin-addons/'
      - name: jaspersoft
        remote_url: 'https://jaspersoft.artifactoryonline.com/jaspersoft/jaspersoft-repo/'
        version_policy: mixed
    nexus_repos_maven_hosted:
      - name: company-project
        version_policy: mixed
        write_policy: allow
        blob_store: company-artifacts
    nexus_repos_maven_group:
      - name: public
        member_repos:
          - central
          - jboss
          - vaadin-addons
          - jaspersoft

    # Yum. Change nexus_config_yum to true for create yum repository
    nexus_config_yum: true
    nexus_repos_yum_hosted:
      - name: private_yum_centos_7
        repodata_depth: 1
    nexus_repos_yum_proxy:
      - name: epel_centos_7_x86_64
        remote_url: http://download.fedoraproject.org/pub/epel/7/x86_64
        maximum_component_age: -1
        maximum_metadata_age: -1
        negative_cache_ttl: 60
      - name: centos-7-os-x86_64
        remote_url: http://mirror.centos.org/centos/7/os/x86_64/
        maximum_component_age: -1
        maximum_metadata_age: -1
        negative_cache_ttl: 60
    nexus_repos_yum_group:
      - name: yum_all
        member_repos:
          - private_yum_centos_7
          - epel_centos_7_x86_64

    # NPM. Change nexus_config_npm to true for create npm repository
    nexus_config_npm: true
    nexus_repos_npm_hosted: []
    nexus_repos_npm_group:
      - name: npm-public
        member_repos:
          - npm-registry
    nexus_repos_npm_proxy:
      - name: npm-registry
        remote_url: https://registry.npmjs.org/
        negative_cache_enabled: false

    # Docker. Change nexus_config_docker to true for create docker repository
    nexus_config_docker: true
    nexus_repos_docker_hosted:
      - name: docker-hosted
        http_port: "{{ nexus_docker_hosted_port }}"
        v1_enabled: True
    nexus_repos_docker_proxy:
      - name: docker-proxy
        http_port: "{{ nexus_docker_proxy_port }}"
        v1_enabled: True
        index_type: "HUB"
        remote_url: "https://registry-1.docker.io"
        use_nexus_certificates_to_access_index: false
        maximum_component_age: 1440
        maximum_metadata_age: 1440
        negative_cache_enabled: true
        negative_cache_ttl: 1440
    nexus_repos_docker_group:
      - name: docker-group
        http_port: "{{ nexus_docker_group_port }}"
        v1_enabled: True
        member_repos:
          - docker-hosted
          - docker-proxy

    # Bower. Change nexus_config_bower to true for create bower repository
    nexus_config_bower: true
    nexus_repos_bower_hosted:
      - name: bower-hosted
    nexus_repos_bower_proxy:
      - name: bower-proxy
        index_type: "proxy"
        remote_url: "https://registry.bower.io"
        use_nexus_certificates_to_access_index: false
        maximum_component_age: 1440
        maximum_metadata_age: 1440
        negative_cache_enabled: true
        negative_cache_ttl: 1440
    nexus_repos_bower_group:
      - name: bower-group
        member_repos:
          - bower-hosted
          - bower-proxy

    # Pypi. Change nexus_config_pypi to true for create pypi repository
    nexus_config_pypi: true
    nexus_repos_pypi_hosted:
      - name: pypi-hosted
    nexus_repos_pypi_proxy:
      - name: pypi-proxy
        index_type: "proxy"
        remote_url: "https://pypi.org/"
        use_nexus_certificates_to_access_index: false
        maximum_component_age: 1440
        maximum_metadata_age: 1440
        negative_cache_enabled: true
        negative_cache_ttl: 1440
    nexus_repos_pypi_group:
      - name: pypi-group
        member_repos:
          - pypi-hosted
          - pypi-proxy

    # rubygems. Change nexus_config_rubygems to true for create rubygems repository
    nexus_config_rubygems: true
    nexus_repos_rubygems_hosted:
      - name: rubygems-hosted
    nexus_repos_rubygems_proxy:
      - name: rubygems-proxy
        index_type: "proxy"
        remote_url: "https://rubygems.org"
        use_nexus_certificates_to_access_index: false
        maximum_component_age: 1440
        maximum_metadata_age: 1440
        negative_cache_enabled: true
        negative_cache_ttl: 1440
    nexus_repos_rubygems_group:
      - name: rubygems-group
        member_repos:
          - rubygems-hosted
          - rubygems-proxy

    # gitlfs. Change nexus_config_gitlfs to true for create gitlfs repository
    nexus_config_gitlfs: true
    nexus_repos_gitlfs_hosted:
      - name: gitlfs-hosted

  roles:
    - { role: geerlingguy.java }
    # Debian/Ubuntu only
    # - { role: geerlingguy.apache, apache_create_vhosts: no, apache_mods_enabled: ["proxy_http.load", "headers.load"], apache_remove_default_vhost: true, tags: ["geerlingguy.apache"] }
    # RedHat/CentOS only
    - { role: geerlingguy.apache, apache_create_vhosts: no, apache_remove_default_vhost: true, tags: ["geerlingguy.apache"] }
    - { role: ansible-thoteam.nexus3-oss, tags: ['ansible-thoteam.nexus3-oss'] }
```



## Скриншоты:

![](https://habrastorage.org/webt/mb/7a/e7/mb7ae7qibkvkuhq_vfrce9hjzoo.png)

![](https://habrastorage.org/webt/jp/vw/pq/jpvwpqijp9ccgw5gzkz6ytv0iyy.png)

## Переменные роли

## Role Variables

Переменные со значениями по умолчанию (см. `default/main.yml`):

### General variables
```yaml
    nexus_version: ''
    nexus_timezone: 'UTC'
```

По умолчанию роль установит последнюю доступную версию Nexus. Вы можете исправить версию, изменив переменную `nexus_version`. Смотрите доступные версии на https://www.sonatype.com/download-oss-sonatype.

Если вы измените версию на более новую, то роль попытается обновить ваш установленный Nexus.

Если вы используете более старую версию Nexus, чем последняя, вы должны убедиться, что не используете функции, которые недоступны в установленном выпуске (например, размещенние yum репозиториев доступно для nexus больше чем 3.8.0, git lfs repo для nexus больше чем 3.3.0 и т. д.)

`nexus timezone` - это имя часового пояса Java, которое может быть полезно в сочетании с приведенными ниже выражениями cron для nexus_scheduled tasks.

### Порт Nexus и контекстный путь

```yaml
    nexus_default_port: 8081
    nexus_default_context_path: '/'
```

Порт и контекстный путь процесса Java-соединения. `nexus_default_context_path` должен содержать косую черту, когда он установлен, например. : `nexus_default_context_path: '/nexus/'`.

### Пользователь и группа ОС Nexus

```yaml
    nexus_os_group: 'nexus'
    nexus_os_user: 'nexus'
```

Пользователь и группа, используемые для владения файлами Nexus и запуска службы, будут созданы ролью, если она отсутствует.

```yaml
    nexus_os_user_home_dir: '/home/nexus'
```

Разрешить изменять домашний каталог по умолчанию для пользователя nexus

### Каталоги экземпляров Nexus

```yaml
    nexus_installation_dir: '/opt'
    nexus_data_dir: '/var/nexus'
    nexus_tmp_dir: "{{ (ansible_os_family == 'RedHat') | ternary('/var/nexus-tmp', '/tmp/nexus') }}"
```

Каталоги Nexus.

* `nexus_installation_dir` содержит установленные исполняемые файлы
* `nexus_data_dir` содержит всю конфигурацию, репозитории и загруженные артефакты. Кастомные пути blobstores `nexus_data_dir` могут быть настроены, смотрите ниже `nexus_blobstores`.
* `nexus_tmp_dir` содержит все временные файлы. Путь по умолчанию для redhat был перемещен из `/tmp` для преодоления потенциальные проблемы с автоматическими процедурами очистки. Смотрите #168.

### Настройка использование памяти Nexus JVM
```yaml
    nexus_min_heap_size: "1200M"
    nexus_max_heap_size: "{{ nexus_min_heap_size }}"
    nexus_max_direct_memory: "2G"
```
Это настройки по умолчанию для Nexus. **Пожалуйста, не изменяйте эти значения** _Если вы не прочитали [раздел памяти системных требований nexus](https://help.sonatype.com/repomanager3/system-requirements#SystemRequirements-Memory)_ и не понимаете, что они делают.

Как второе предупреждение, вот выдержка из вышеупомянутого документа:

> Не рекомендуется увеличивать память JVM heap больше рекомендуемых значений в попытке повысить производительность. Это на самом деле может иметь противоположный эффект, приводя к ненужной работе операционной системы.

### Пароль администратора
```yaml
    nexus_admin_password: 'changeme'
```
Пароль учетной записи «admin» для настройки. _Это работает только при первой установке по умолчанию_. Пожалуйста, смотрите [Изменить пароль администратора после первой установки](# change-admin-password-after-first-install), если вы хотите изменить его позже с помощью роли.

**Настоятельно не рекомендуется хранить свой пароль в виде открытого текста в playbook, а использовать [шифрование ansible-vault] (https://docs.ansible.com/ansible/latest/user_guide/vault.html) (либо встроенный или в отдельный файл, загруженный, например, с помощью include_vars)**


### Анонимный доступ по умолчанию
```yaml
    nexus_anonymous_access: false
```

Анонимный доступ по умолчанию вылючен. Подробнее про [анонимный доступ](https://help.sonatype.com/display/NXRM3/Anonymous+Access).

### Публичное имя хоста
```yaml
    nexus_public_hostname: 'nexus.vm'
    nexus_public_scheme: https
```

Полное доменное имя и схема (https или http), по которой экземпляр Nexus будет доступен для его клиентов.

### Доступ API для этой роли

```yaml
    nexus_api_hostname: localhost
    nexus_api_scheme: http
    nexus_api_validate_certs: "{{ nexus_api_scheme == 'https' }}"
    nexus_api_context_path: "{{ nexus_default_context_path }}"
    nexus_api_port: "{{ nexus_default_port }}"
```
Эти переменные контролируют, как роль подключается к API Nexus для предоставления.
**Только для продвинутых пользователей. Скорее всего, вы не хотите изменять эти настройки по умолчанию**

### Настройка обратного прокси
```yaml
    httpd_setup_enable: false
    httpd_server_name: "{{ nexus_public_hostname }}"
    httpd_default_admin_email: "admin@example.com"
    httpd_ssl_certificate_file: 'files/nexus.vm.crt'
    httpd_ssl_certificate_key_file: 'files/nexus.vm.key'
    # httpd_ssl_certificate_chain_file: "{{ httpd_ssl_certificate_file }}"
    httpd_copy_ssl_files: true
```

Установите [Обратный прокси-сервер SSL](https://help.sonatype.com/display/NXRM3/Run+Behind+a+Reverse+Proxy#RunBehindaReverseProxy-Example:ReverseProxySSLTerminationatBasePath).
Для этого нужно установить httpd. Примечание: когда для `httpd_setup_enable` установлено значение` true`, nexus связывается с 127.0.0.1:8081, таким образом *не* будучи напрямую доступным через HTTP-порт 8081 с внешнего IP-адреса.

Используемое имя хоста по умолчанию - `nexus_public_hostname`. Если вам нужны разные имена по какой-либо причине, вы можете установить `httpd_server_name` с другим значением.

С `httpd_copy_ssl_files: true` (по умолчанию) вышеупомянутые сертификаты должны существовать в вашей директории playbook и будут скопированы на сервер и настроены в apache.

Если вы хотите использовать существующие сертификаты на сервере, установите `httpd_copy_ssl_files: false` и предоставьте следующие переменные:

```yaml
    # These specifies to the vhost where to find on the remote server file
    # system the certificate files.
    httpd_ssl_cert_file_location: "/etc/pki/tls/certs/wildcard.vm.crt"
    httpd_ssl_cert_key_location: "/etc/pki/tls/private/wildcard.vm.key"
    # httpd_ssl_cert_chain_file_location: "{{ httpd_ssl_cert_file_location }}"
```

`httpd_ssl_cert_chain_file_location` является необязательным и должен быть оставлен неустановленным, если вы не хотите настраивать файл цепочки

```yaml
    httpd_default_admin_email: "admin@example.com"
```

Установить адрес электронной почты администратора по умолчанию

### Конфигурация LDAP

Соединения LDAP и область безопасности по умолчанию отключены

```yaml
    nexus_ldap_realm: false
    ldap_connections: []
```

[Соединения LDAP](https://help.sonatype.com/display/NXRM3/LDAP), каждый элемент выглядит следующим образом:

```yaml
    nexus_ldap_realm: true
    ldap_connections:
      - ldap_name: 'My Company LDAP' # used as a key to update the ldap config
        ldap_protocol: 'ldaps' # ldap or ldaps
        ldap_hostname: 'ldap.mycompany.com'
        ldap_port: 636
        ldap_use_trust_store: false # Wether or not to use certs in the nexus trust store
        ldap_search_base: 'dc=mycompany,dc=net'
        ldap_auth: 'none' # or simple
        ldap_auth_username: 'username' # if auth = simple
        ldap_auth_password: 'password' # if auth = simple
        ldap_user_base_dn: 'ou=users'
        ldap_user_filter: '(cn=*)' # (optional)
        ldap_user_object_class: 'inetOrgPerson'
        ldap_user_id_attribute: 'uid'
        ldap_user_real_name_attribute: 'cn'
        ldap_user_email_attribute: 'mail'
        ldap_user_subtree: false
        ldap_map_groups_as_roles: false
        ldap_group_base_dn: 'ou=groups'
        ldap_group_object_class: 'posixGroup'
        ldap_group_id_attribute: 'cn'
        ldap_group_member_attribute: 'memberUid'
        ldap_group_member_format: '${username}'
        ldap_group_subtree: false
```

Пример конфигурации LDAP для анонимной аутентификации (анонимная привязка), это также «минимальная» конфигурация:

```yaml
    nexus_ldap_realm: true
    ldap_connection:
      - ldap_name: 'Simplest LDAP config'
        ldap_protocol: 'ldaps'
        ldap_hostname: 'annuaire.mycompany.com'
        ldap_search_base: 'dc=mycompany,dc=net'
        ldap_port: 636
        ldap_use_trust_store: false
        ldap_user_id_attribute: 'uid'
        ldap_user_real_name_attribute: 'cn'
        ldap_user_email_attribute: 'mail'
        ldap_user_object_class: 'inetOrgPerson'
```

Пример конфигурации LDAP для простой аутентификации (с использованием учетной записи DSA):

```yaml
    nexus_ldap_realm: true
    ldap_connections:
      - ldap_name: 'LDAP config with DSA'
        ldap_protocol: 'ldaps'
        ldap_hostname: 'annuaire.mycompany.com'
        ldap_port: 636
        ldap_use_trust_store: false
        ldap_auth: 'simple'
        ldap_auth_username: 'cn=mynexus,ou=dsa,dc=mycompany,dc=net'
        ldap_auth_password: "{{ vault_ldap_dsa_password }}" # better keep passwords in an ansible vault
        ldap_search_base: 'dc=mycompany,dc=net'
        ldap_user_base_dn: 'ou=users'
        ldap_user_object_class: 'inetOrgPerson'
        ldap_user_id_attribute: 'uid'
        ldap_user_real_name_attribute: 'cn'
        ldap_user_email_attribute: 'mail'
        ldap_user_subtree: false
```

Пример конфигурации LDAP для простой аутентификации (с использованием учетной записи DSA) + группы, сопоставленные как роли:

```yaml
    nexus_ldap_realm: true
    ldap_connections
      - ldap_name: 'LDAP config with DSA'
        ldap_protocol: 'ldaps'
        ldap_hostname: 'annuaire.mycompany.com'
        ldap_port: 636
        ldap_use_trust_store: false
        ldap_auth: 'simple'
        ldap_auth_username: 'cn=mynexus,ou=dsa,dc=mycompany,dc=net'
        ldap_auth_password: "{{ vault_ldap_dsa_password }}" # better keep passwords in an ansible vault
        ldap_search_base: 'dc=mycompany,dc=net'
        ldap_user_base_dn: 'ou=users'
        ldap_user_object_class: 'inetOrgPerson'
        ldap_user_id_attribute: 'uid'
        ldap_user_real_name_attribute: 'cn'
        ldap_user_email_attribute: 'mail'
        ldap_map_groups_as_roles: true
        ldap_group_base_dn: 'ou=groups'
        ldap_group_object_class: 'groupOfNames'
        ldap_group_id_attribute: 'cn'
        ldap_group_member_attribute: 'member'
        ldap_group_member_format: 'uid=${username},ou=users,dc=mycompany,dc=net'
        ldap_group_subtree: false
```

Пример конфигурации LDAP для простой аутентификации (с использованием учетной записи DSA) + группы, динамически сопоставленные как роли:

```yaml
    nexus_ldap_realm: true
    ldap_connections:
      - ldap_name: 'LDAP config with DSA'
        ldap_protocol: 'ldaps'
        ldap_hostname: 'annuaire.mycompany.com'
        ldap_port: 636
        ldap_use_trust_store: false
        ldap_auth: 'simple'
        ldap_auth_username: 'cn=mynexus,ou=dsa,dc=mycompany,dc=net'
        ldap_auth_password: "{{ vault_ldap_dsa_password }}" # better keep passwords in an ansible vault
        ldap_search_base: 'dc=mycompany,dc=net'
        ldap_user_base_dn: 'ou=users'
        ldap_user_object_class: 'inetOrgPerson'
        ldap_user_id_attribute: 'uid'
        ldap_user_real_name_attribute: 'cn'
        ldap_user_email_attribute: 'mail'
        ldap_map_groups_as_roles: true
        ldap_map_groups_as_roles_type: 'dynamic'
        ldap_user_memberof_attribute: 'memberOf'
```

### Привилегии
```yaml
    nexus_privileges:
      - name: all-repos-read # used as key to update a privilege
        # type: <one of application, repository-admin, repository-content-selector, repository-view, script or wildcard>
        description: 'Read & Browse access to all repos'
        repository: '*'
        actions: # can be add, browse, create, delete, edit, read or  * (all)
          - read
          - browse
        # pattern: pattern
        # domain: domain
        # script_name: name
```

Список [привилегий](https://help.sonatype.com/display/NXRM3/Privileges) для настройки. Посмотрите документацию и графический интерфейс, чтобы проверить, какие переменные должны быть установлены в зависимости от типа привилегии.

Эти элементы объединяются со следующими значениями по умолчанию:

```yaml
    _nexus_privilege_defaults:
      type: repository-view
      format: maven2
      actions:
        - read
```


### Роли (внутри Nexus имеется виду)
```yaml
    nexus_roles:
      - id: Developpers # can map to a LDAP group id, also used as a key to update a role
        name: developers
        description: All developers
        privileges:
          - nx-search-read
          - all-repos-read
        roles: [] # references to other role names
```

Список [ролей](https://help.sonatype.com/display/NXRM3/Roles) для настройки.

### Пользователи
```yaml
    nexus_local_users: []
      # - username: jenkins # used as key to update
      #   state: present # default value if ommited, use 'absent' to remove user
      #   first_name: Jenkins
      #   last_name: CI
      #   email: support@company.com
      #   password: "s3cr3t"
      #   roles:
      #     - developers # role ID
```
Local (non-LDAP) users/accounts list to create in nexus.

Список локальных (не LDAP) пользователей/учетных записей для создания в Nexus. 

```yaml
      nexus_ldap_users: []
      # - username: j.doe
      #   state: present
      #   roles:
      #     - "nx-admin"
```
Маппинг Ldap пользователей/ролей. Состояние `absent` удалит роли из существующего пользователя, если он уже существует.
Пользователи Ldap не удаляются. Попытка установить роль для несуществующего пользователя приведет к ошибке.


### Селекторы контента
```yaml
  nexus_content_selectors:
  - name: docker-login
    description: Selector for docker login privilege
    search_expression: format=="docker" and path=~"/v2/"
```

Для получения дополнительной информации о селекторе контента см. [Документацию](https://help.sonatype.com/repomanager3/configuration/repository-management#RepositoryManagement-ContentSelectors).

Чтобы использовать селектор контента, добавьте новую привилегию с `type: repository-content-selector` и соответствующим` contentSelector`

```yaml
- name: docker-login-privilege
  type: repository-content-selector
  contentSelector: docker-login
  description: 'Login to Docker registry'
  repository: '*'
  actions:
  - read
  - browse
```

### Blobstores и репозитории
```yaml
    nexus_delete_default_repos: false
```

Delete the repositories from the nexus install initial default configuration. This step is only executed on first-time install (when `nexus_data_dir` has been detected empty).

Удаление репозиториев из исходной конфигурации по умолчанию для Nexus. Этот шаг выполняется только при первой установке (когда `nexus_data_dir` пустой).

```yaml
    nexus_delete_default_blobstore: false
```

Delete the default blobstore from the nexus install initial default configuration. This can be done only if `nexus_delete_default_repos: true` and all configured repositories (see below) have an explicit `blob_store: custom`. This step is only executed on first-time install (when `nexus_data_dir` has been detected empty).

Удаление хранилища блобов (бинарных артефактов) выключено по умолчанию из исходной конфигурации. Для удаления хранилища блобов (бинарных артефактов) выключите  `nexus_delete_default_repos: true` . Этот шаг выполняется только при первой установке (когда `nexus_data_dir` пустой).

```yaml
    nexus_blobstores: []
    # example blobstore item :
    # - name: separate-storage
    #   type: file
    #   path: /mnt/custom/path
    # - name: s3-blobstore
    #   type: S3
    #   config:
    #     bucket: s3-blobstore
    #     accessKeyId: "{{ VAULT_ENCRYPTED_KEY_ID }}"
    #     secretAccessKey: "{{ VAULT_ENCRYPTED_ACCESS_KEY }}"
```

[Blobstores](https://help.sonatype.com/display/NXRM3/Repository+Management#RepositoryManagement-BlobStores) to create. A blobstore path and a repository blobstore cannot be updated after initial creation (any update here will be ignored on re-provisionning).

Configuring blobstore on S3 is provided as a convenience and is not part of the automated tests we run on travis. Please note that storing on S3 is only recommended for instances deployed on AWS.

Cоздание [Blobstores](https://help.sonatype.com/display/NXRM3/Repository+Management#RepositoryManagement-BlobStores). Путь к хранилищу и репозиторию хранилищ не могут быть обновлены после первоначального создания (любое обновление здесь будет игнорироваться при повторной установке).

Настройка хранилища BLOB-объектов на S3 предоставляется для удобства. Обратите внимание, что хранение на S3 рекомендуется только для экземпляров, развернутых на AWS.

```yaml
    nexus_repos_maven_proxy:
      - name: central
        remote_url: 'https://repo1.maven.org/maven2/'
        layout_policy: permissive
        # maximum_component_age: -1
        # maximum_metadata_age: 1440
        # negative_cache_enabled: true
        # negative_cache_ttl: 1440
      - name: jboss
        remote_url: 'https://repository.jboss.org/nexus/content/groups/public-jboss/'
        # maximum_component_age: -1
        # maximum_metadata_age: 1440
        # negative_cache_enabled: true
        # negative_cache_ttl: 1440
    # example with a login/password :
    # - name: secret-remote-repo
    #   remote_url: 'https://company.com/repo/secure/private/go/away'
    #   remote_username: 'username'
    #   remote_password: 'secret'
    #   # maximum_component_age: -1
    #   # maximum_metadata_age: 1440
    #   # negative_cache_enabled: true
    #   # negative_cache_ttl: 1440
```

Выше пример конфигурации [прокси-сервер](https://help.sonatype.com/display/NXRM3/Repository+Management#RepositoryManagement-ProxyRepository) Maven.

```yaml
    nexus_repos_maven_hosted:
      - name: private-release
        version_policy: release
        write_policy: allow_once  # one of "allow", "allow_once" or "deny"
```

Maven [hosted repositories](https://help.sonatype.com/display/NXRM3/Repository+Management#RepositoryManagement-HostedRepository) configuration. Negative cache config is optionnal and will default to the above values if omitted.

Конфигурация [размещенных (hosted) репозиториев](https://help.sonatype.com/display/NXRM3/Repository+Management#RepositoryManagement-HostedRepository) Maven. Конфигурация отрицательного кэша (-1) является необязательной и будет по умолчанию использовать вышеуказанные значения, если не указана.

```yaml
    nexus_repos_maven_group:
      - name: public
        member_repos:
          - central
          - jboss
```

Конфигурация [группы](https://help.sonatype.com/display/NXRM3/Repository+Management#RepositoryManagement-RepositoryGroup) Maven.

Все три типа репозитория объединяются со следующими значениями по умолчанию:

```yaml
    _nexus_repos_maven_defaults:
      blob_store: default # Note : cannot be updated once the repo has been created
      strict_content_validation: true
      version_policy: release # release, snapshot or mixed
      layout_policy: strict # strict or permissive
      write_policy: allow_once # one of "allow", "allow_once" or "deny"
      maximum_component_age: -1  # Nexus gui default. For proxies only
      maximum_metadata_age: 1440  # Nexus gui default. For proxies only
      negative_cache_enabled: true # Nexus gui default. For proxies only
      negative_cache_ttl: 1440 # Nexus gui default. For proxies only
```

Docker, Pypi, Raw, Rubygems, Bower, NPM, Git-LFS and yum repository types:
see `defaults/main.yml` for these options:

Хранилища Docker, Pypi, Raw, Rubygems, Bower, NPM, Git-LFS и yum по умолчанию выключены:
Смотрите `defaults/main.yml` для этих опций:

```yaml
      nexus_config_pypi: false
      nexus_config_docker: false
      nexus_config_raw: false
      nexus_config_rubygems: false
      nexus_config_bower: false
      nexus_config_npm: false
      nexus_config_gitlfs: false
      nexus_config_yum: false
```

Обратите внимание, что вам может потребоваться включить определенные области безопасности, если вы хотите использовать другие типы репозиториев, кроме maven. Это по умолчанию false

```yaml
nexus_nuget_api_key_realm: false
nexus_npm_bearer_token_realm: false
nexus_docker_bearer_token_realm: false  # required for docker anonymous access
```

Remote User Realm также может быть включена с помощью

```yaml
nexus_rut_auth_realm: true
```

и заголовок может быть настроен путем определения

```yaml
nexus_rut_auth_header: "CUSTOM_HEADER"
```

### Запланированные задачи
```yaml
    nexus_scheduled_tasks: []
    #  #  Example task to compact blobstore :
    #  - name: compact-docker-blobstore
    #    cron: '0 0 22 * * ?'
    #    typeId: blobstore.compact
    #    task_alert_email: alerts@example.org  # optional
    #    taskProperties:
    #      blobstoreName: {{ nexus_blob_names.docker.blob }} # all task attributes are stored as strings by nexus internally
    #  #  Example task to purge maven snapshots
    #  - name: Purge-maven-snapshots
    #    cron: '0 50 23 * * ?'
    #    typeId: repository.maven.remove-snapshots
    #    task_alert_email: alerts@example.org  # optional
    #    taskProperties:
    #      repositoryName: "*"  # * for all repos. Change to a repository name if you only want a specific one
    #      minimumRetained: "2"
    #      snapshotRetentionDays: "2"
    #      gracePeriodInDays: "2"
    #    booleanTaskProperties:
    #      removeIfReleased: true
    #  #  Example task to purge unused docker manifest and images
    #  - name: Purge unused docker manifests and images
    #    cron: '0 55 23 * * ?'
    #    typeId: "repository.docker.gc"
    #    task_alert_email: alerts@example.org  # optional
    #    taskProperties:
    #      repositoryName: "*"  # * for all repos. Change to a repository name if you only want a specific one
    #  #  Example task to purge incomplete docker uploads
    #  - name: Purge incomplete docker uploads
    #    cron: '0 0 0 * * ?'
    #    typeId: "repository.docker.upload-purge"
    #    task_alert_email: alerts@example.org  # optional
    #    taskProperties:
    #      age: "24"
```

[Запланированные задачи](https://help.sonatype.com/display/NXRM3/System+Configuration#SystemConfiguration-ConfiguringandExecutingTasks) для настройки. `typeId` и специфичные для задачи` taskProperties`/`booleanTaskProperties` можно угадать либо:

* из иерархии типов Java `org.sonatype.nexus.scheduling.TaskDescriptorSupport`
* проверяя HTML-форму создания задачи в вашем браузере
* от просмотра запросов AJAX в браузере при ручной настройке задачи.

**Свойства задачи должны быть объявлены в правильном блоке yaml в зависимости от их типа**:

* `taskProperties` для всех строковых свойств (т. е. имен репозиториев, имен хранилищ, периодов времени ...).
* `booleanTaskProperties` для всех логических свойств (т. е. в основном чекбоксы в графическом интерфейсе задачи создания нексуса).

### Резервные копии
```yaml
      nexus_backup_configure: false
      nexus_backup_cron: '0 0 21 * * ?'  # See cron expressions definition in nexus create task gui
      nexus_backup_dir: '/var/nexus-backup'
      nexus_restore_log: '{{ nexus_backup_dir }}/nexus-restore.log'
      nexus_backup_rotate: false
      nexus_backup_rotate_first: false
      nexus_backup_keep_rotations: 4  # Keep 4 backup rotation by default (current + last 3)
```

Резервное копирование не будет настроено, пока вы не переключите `nexus_backup_configure` в `true`.
В этом случае запланированная задача сценария будет настроена для запуска в Nexus
с интервалом, указанным в `nexus_backup_cron` (по умолчанию 21:00 каждый день).
См. [Шаблон groovy для этой задачи](templates / backup.groovy.j2) для получения подробной информации.
Это запланированное задание не зависит от других `nexus_scheduled_tasks`, которые вы
объявить в своем playbook.

Если вы хотите ротировать/удалять резервные копии, установите `nexus_backup_rotate: true` и настройте количество бекапов, которое вы хотели бы сохранить с помощью `nexus_backup_keep_rotations` (по умолчанию 4).

При использовании ротации, если вы хотите сэкономить дополнительное дисковое пространство во время процесса резервного копирования,
Вы можете установить `nexus_backup_rotate_first: true`. Это настроит предварительное ротирование/удаление перед бекапом. По умолчанию ротиция происходит после создания резервной копии. Обратите внимание, чем в этом случае старые резервные копии
удалятся до того, как текущий бекап будет сделан.

#### Процедура восстановления
Запустите playbook с параметром `-e nexus_restore_point=<YYYY-MM-dd-HH-mm-ss>`
(например, 2017-12-17-21-00-00 для 17 декабря 2017 в 21:00

#### Удаление nexus

**Предупреждение: это полностью удалит текущие данные. Обязательно сделайте резервную копию ранее, если это необходимо**

Используйте переменную `nexus_purge`, если вам нужно перезапустить с нуля и переустановить экземпляр nexus с удалением всех данных.

```bash
ansible-playbook -i your/inventory.ini your_nexus_playbook.yml -e nexus_purge=true
```

#### Изменить пароль администратора после первой установки

```yaml
    nexus_default_admin_password: 'admin123'
```
**Это не должно быть изменено в вашем playbook**. Эта переменная заполняется стандартным паролем администратора Nexus при первой установке и гарантирует, что мы можем изменить пароль администратора на `nexus_admin_password`.

Если вы хотите изменить пароль администратора после первой установки, вы можете временно изменить его на старый пароль из командной строки. После изменения `nexus_admin_password` в вашей игровой книге вы можете запустить:

```bash
ansible-playbook -i your/inventory.ini your_playbook.yml -e nexus_default_admin_password=oldPassword
```

Телеграм канал по Nexus Sonatype: https://t.me/ru_nexus_sonatype

