# Домашнее задание к занятию "08.01 Введение в Ansible"

## Подготовка к выполнению
1. Установите ansible версии 2.10 или выше.
2. Создайте свой собственный публичный репозиторий на github с произвольным именем.
3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.
Решение:
- some_fact имеет знание 12 (см.ниже)

```
vagrant@vagrant:~/ansible-lesson0701/playbook$ ansible-playbook --connection=local -i /home/vagrant/ansible-lesson0701/playbook/inventory/test.yml playbook1.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 20.04 on host localhost should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release
will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed
in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [localhost]

TASK [Print fact] ***********************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ******************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
Решение:
Это файл расположенный по пути /home/vagrant/ansible-lesson0701/playbook/group_vars/all

```
vagrant@vagrant:~/ansible-lesson0701/playbook/group_vars/all$ cat examp.yml
---
  some_fact: 'all default fact'
```

3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.

Решение:

sudo docker run --name centos7 -p 22:22  -d eeb6ee3f44bd


sudo docker run --name centos7  --expose 22 -it  centos:centos7

- запустим Docker с Centos7:
```
vagrant@vagrant:~/ansible-lesson0701/playbook$ sudo docker run --name centos7  --expose 22 -it  centos:centos7
[root@04c67bf40f94 /]#
```
- соберем образ с устанволенным python для Ubuntu 
```
vagrant@vagrant:~/ansible-lesson0701$ cat Dockerfile
FROM ubuntu:latest
MAINTAINER fnndsc "dev@babymri.org"

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
  && apt-get install -y python3-pip python3-dev \
  && cd /usr/local/bin \
  && ln -s /usr/bin/python3 python \
  && pip3 install --upgrade pip
vagrant@vagrant:~/ansible-lesson0701$ sudo docker build -t ubuntu_kaplin:v1 ./
```

- Образ убунту доступен
```
vagrant@vagrant:~/ansible-lesson0701$ sudo docker image ls
REPOSITORY                        TAG           IMAGE ID       CREATED          SIZE
ubuntu_kaplin                     v1            e2916377a461   20 minutes ago   477MB
ubuntu                            latest        216c552ea5ba   31 hours ago     77.8MB
ubuntu                            20.04         817578334b4d   31 hours ago     72.8MB
ubuntu                            18.04         71cb16d32be4   31 hours ago     63.1MB
<none>                            <none>        dba3480dd7a7   2 days ago       72.8MB
<none>                            <none>        71ce1d10d18d   2 days ago       72.8MB
postgres                          13            e30fb19f297b   3 weeks ago      373MB
kapelman/06-db-05-elasticsearch   es-image      35b091a078ee   3 weeks ago      2.78GB
my-es-image                       latest        35b091a078ee   3 weeks ago      2.78GB
mysql                             8.0           ff3b5098b416   5 weeks ago      447MB
postgres                          12            f2f1f275f1a1   6 weeks ago      373MB
adminer                           latest        75cd6c93316c   8 weeks ago      90.7MB
kaplin-nginx                      first_ver     11b04307b160   2 months ago     142MB
kapelman/05-virt-02-iaac          first-image   11b04307b160   2 months ago     142MB
nginx                             latest        41b0e86104ba   2 months ago     142MB
debian                            latest        123c2f3835fd   2 months ago     124MB
hello-world                       latest        feb5d9fea6a5   12 months ago    13.3kB
centos                            centos7       eeb6ee3f44bd   12 months ago    204MB
```
- запустим docker с ubuntu
```
vagrant@vagrant:~/ansible-lesson0701$ sudo docker run --name ubuntu  --expose 22 -it  ubuntu_kaplin:v1
root@595e8d68c8ca:/#
```

4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.

Решение:
```
vagrant@vagrant:~/ansible-lesson0701/playbook$ sudo ansible-playbook -i /home/vagrant/ansible-lesson0701/playbook/inventory/prod.yml site.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP ******************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.

Решение:
```
vagrant@vagrant:~/ansible-lesson0701/playbook/group_vars/deb$ cat examp.yml
---
  some_fact: "deb default fact"

vagrant@vagrant:~/ansible-lesson0701/playbook/group_vars/el$ cat examp.yml
---
  some_fact: "el default fact"
```

6. Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
Решение:
```
vagrant@vagrant:~/ansible-lesson0701/playbook$ sudo ansible-playbook -i /home/vagrant/ansible-lesson0701/playbook/inventory/prod.yml site.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.

Решение:
```
vagrant@vagrant:~/ansible-lesson0701/playbook/group_vars/deb$ ansible-vault encrypt examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful

vagrant@vagrant:~/ansible-lesson0701/playbook/group_vars/el$ ansible-vault encrypt examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```

8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.

Решение: Добавим флаг --ask-vault-pass

```
vagrant@vagrant:~/ansible-lesson0701/playbook$ sudo ansible-playbook --ask-vault-pass -i /home/vagrant/ansible-lesson0701/playbook/inventory/prod.yml site.yml
Vault password:

PLAY [Print os facts] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.

```
vagrant@vagrant:~/ansible-lesson0701$ ansible-doc -t 'connection' -l

local        execute on controller
```

10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
Решение:
```
vagrant@vagrant:~/ansible-lesson0701/playbook/inventory$ cat prod.yml
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local
```
12. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.

Решение:
Все переменные показываются корректно.
```
vagrant@vagrant:~/ansible-lesson0701/playbook$ sudo ansible-playbook --ask-vault-pass -i /home/vagrant/ansible-lesson0701/playbook/inventory/prod.yml site.yml
Vault password:

PLAY [Print os facts] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 20.04 on host localhost should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release
will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed
in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

## Необязательная часть

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
2. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
3. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
4. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот](https://hub.docker.com/r/pycontribs/fedora).
5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
6. Все изменения должны быть зафиксированы и отправлены в вашей личный репозиторий.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
