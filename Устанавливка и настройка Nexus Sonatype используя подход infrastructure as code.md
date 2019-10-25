### Введение

Sonatype Nexus – интегрированная платформа управления, с помощью которой разработчики могут проксировать и управлять зависимостями Java, образами Docker, Python, Ruby, NPM, Bower, RPM-пакетами и gitlfs, а также распространять свое программное обеспечение. 

### Зачем нужен Sonatype Nexus?

- Для хранения приватных артефактов;

- Для кеширования артефактов, которые скачиваются из интернета;

  

### Артефакты поддерживаемые в базовой поставке Sonatype Nexus:

- Java (jar)
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

### Установка Sonatype Nexus используя https://github.com/ansible-ThoTeam/nexus3-oss

### Требования

- Эта роль протестирована на CentOS 7, Ubuntu Xenial (16.04) and Bionic (18.04), Debian Jessie and Stretch

- `jmespath` библиотека должна быть установлена на рабочей станции где запускается playbook. Для установка выполните sudo pip install -r `requirements.txt`

### Пример ansible-playbook для установки 

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



### Скриншоты:

![](https://habrastorage.org/webt/mb/7a/e7/mb7ae7qibkvkuhq_vfrce9hjzoo.png)

![](https://habrastorage.org/webt/jp/vw/pq/jpvwpqijp9ccgw5gzkz6ytv0iyy.png)