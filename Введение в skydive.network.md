Введение в Skydive

Skydive - это анализатор топологии сети и протоколов с открытым исходным кодом в реальном времени. Он направлен на то, чтобы предоставить исчерпывающий способ понять, что происходит в сетевой инфраструктуре.

</cut>

Для демонстрации установоим на 3 сервера кластер Etcd. Для этого будем использовать galaxy роль frank6866.etcd.

```yaml
- hosts: etcd
  become: yes
  roles:
      - frank6866.etcd
```
файл inventory hosts.multiple
```
frank6866-etcd-1 ansible_ssh_host=172.26.9.78 etcd_public_ip=172.26.9.78
frank6866-etcd-2 ansible_ssh_host=172.26.9.79 etcd_public_ip=172.26.9.79
frank6866-etcd-3 ansible_ssh_host=172.26.9.80 etcd_public_ip=172.26.9.80

[cluster1]
frank6866-etcd-[1:3]

[etcd:children]
cluster1

[etcd:vars]
etcd_tls_enabled='false'
```
Запускаем установку
```
 ansible-playbook -i hosts.multiple etcd-cluster.yaml
```
После скачиваем репозиторий skydive
```
git clone https://github.com/skydive-project/skydive.git
```
Переходим в папку contrib/ansible/inventory
```
cd contrib/ansible/inventory
```
Правим IP в файле hosts.multiple
```
[analyzers]
IP для сервера анализатора

[agents]
Три IP etcd кластера

# Нужно раскоментировать эти строки для skydive-flow-matrix 
# For skydive-flow-matrix add skydive_extra_config:
[agents:vars]
skydive_extra_config={'agent.topology.probes': ['socketinfo',]}
```
Запускаем установку skydive агентов и анализатора
```
ansible-playbook -i inventory/hosts.multiple playbook.yml.sample
```
После со своего компьютера заходим в `IP для сервера анализатора:8082`
И видим примерно такую картину
![](https://habrastorage.org/webt/uq/oe/0z/uqoe0zexu4kmjcozf5dgu-0gh6o.png)
