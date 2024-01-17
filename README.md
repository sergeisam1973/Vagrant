# Домашнее задание 1. Vagrant-стенд для обновления ядра и создания образа системы
## Описание домашнего задания
1) Обновить ядро ОС из репозитория ELRepo
2) Создать Vagrant box c помощью Packer
3) Загрузить Vagrant box в Vagrant Cloud
#### Ссылка на репозиторий GitHub:
https://github.com/sergeisam1973/Vagrant
####Описание команд и их вывод
##### Обновление ядра
В домашнем каталоге создаем каталог vm и переходим в него:
mkdir vm
cd vm

Создаём Vagrantfile (копируя его содержимое из репозитория https://github.com/Nickmob/vagrant_kernel_update/tree/main)

Запускаем виртуальную машину, подключаемся к ней, проверяем версию ядра: 
vagrant up
vagrant ssh
[vagrant@kernel-update ~]$ uname -r
Вывод команды: 4*

Подключаем репозиторий, откуда берём необходимую версию ядра, устанавливаем ядро и настраиваем загрузчик:
sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
sudo yum --enablerepo elrepo-kernel install kernel-ml -y
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo grub2-set-default 0

Проверяем версию ядра:
[vagrant@kernel-update ~]$ uname -r
6.6.11-1.el8.elrepo.x86_64

##### Создание Vagrant box c помощью Packer

В каталоге ~/vm создаём папку packer и переходим в нее:
mkdir packer
cd packer

Создаём файл centos.json (копируя его содержимое из репозитория https://github.com/Nickmob/vagrant_kernel_update/tree/main/packer). После создания изменяем параметр iso_checksum на полученный с сервера (https://mirror.linux-ia64.org/centos/8-stream/isos/x86_64/). Кроме того, пришлось увеличить значение параметра ssh_timeout с 30 до 180 минут, поскольку за 30 минут образ не успевал собраться.

Далее создаем в каталоге packer каталог http и переходим в него: 
mkdir http
cd http

Копируем в созданный каталог файл vagrant.ks из репозитория https://github.com/Nickmob/vagrant_kernel_update/tree/main/packer/http

Аналогичным образом создаем каталог для скриптов и копируем в него скрипты из репозитория https://github.com/Nickmob/vagrant_kernel_update/tree/main/packer/scripts:
cd ..
mkdir scripts
cd scripts

Переходим в каталог packer и создаём образ системы:
cd ..
packer build centos.json

Импортируем полученный vagrant box в Vagrant:
vagrant box add --name centos8-kernel6_ centos-8-kernel-6-x86_64-Minimal.box

Проверяем, что образ теперь есть в списке имеющихся образов vagrant:
serg@serg-VirtualBox:~/vm/packer$ vagrant box list
centos8-kernel6_ (virtualbox, 0)
generic/centos8s (virtualbox, 4.3.4, (amd64))
ubuntu/focal64   (virtualbox, 20231207.0.0)

Создаём Vagrantfile на основе образа centos8-kernel6_:
vagrant init centos8-kernel6_

Запускаем ВМ и подключаемся к ней: 
vagrant up
vagrant ssh

Проверяем версию ядра:
[vagrant@otus-c8 ~]$ uname -r
6.6.11-1.el8.elrepo.x86_64

##### Загрузка Vagrant box в Vagrant Cloud

Загрузить в облако предложенным в методичке способом (vagrant cloud publish) не получилось, выгрузка доходила до 100% и вылетала с ошибкой. Поэтому загрузка проведена через веб-интерфейс Vagrant Cloud в своем аккаунте (https://app.vagrantup.com/boxes/new).

Ссылка на загруженный в облако Vagrant box:
https://app.vagrantup.com/sergeisam1973/boxes/centos8-kernel6_/versions/1.0

Далее импортируем Vagrant box из облака:
serg@serg-VirtualBox:~/vm/packer$ vagrant box add --name centos8-kernel6_ https://app.vagrantup.com/sergeisam1973/boxes/centos8-kernel6_/versions/1.0/providers/virtualbox/amd64/vagrant.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos8-kernel6_' (v0) for provider: 
    box: Downloading: https://app.vagrantup.com/sergeisam1973/boxes/centos8-kernel6_/versions/1.0/providers/virtualbox/amd64/vagrant.box
==> box: Successfully added box 'centos8-kernel6_' (v0) for ''!

Запускаем ВМ и подключаемся к ней: 
vagrant up
vagrant ssh

#### Особенности проектирования и реализации решения:
При выполнении домашнего задания использовались Vagrant 2.4.0, VirtualBox 7.0.12 r159484 и Packer 1.10.0, хостовая ОС: Ubuntu 22.04 Desktop (виртуальная машина под VirtualBox).