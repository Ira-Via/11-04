# 11-04
11-04 «Очереди RabbitMQ» Васяева Ирина
# Задание 1
1. Создание Vagrant-приложения  
![image](https://github.com/user-attachments/assets/834c5f7a-25a0-4585-ac3c-2b557f455644)  
![image](https://github.com/user-attachments/assets/8b41d9e9-d26b-4156-af8b-2c774af289e2)  
```
Vagrant.configure("2") do |config|
     config.vm.box = "ubuntu/bionic64"
     
     config.vm.network "forwarded_port", guest: 15672, host: 15672 # Port for RabbitMQ Management Plugin

     config.vm.provision "shell", inline: <<-SHELL

       apt-get update
       apt-get upgrade -y

       apt-get install -y rabbitmq-server

       rabbitmq-plugins enable rabbitmq_management

       systemctl restart rabbitmq-server
     SHELL
   end
```
2.Запуск Vagrant:
```
 vagrant up
```
3. Доступ к веб-интерфейсу RabbitMQ
```
http://localhost:15672
```
# Задание 2
1. Установка необходимых зависимостей  
![image](https://github.com/user-attachments/assets/c2e0fd7b-a6d2-4242-82d2-c61c8a7249c0)
![image](https://github.com/user-attachments/assets/e7df648a-fdde-4eec-b776-1ed7ef114449)
2. Запуск RabbitMQ
```
sudo systemctl start rabbitmq-server
```
3.  Изменение скриптов:  
* изменения в producer.py:  
```
connection = pika.BlockingConnection(pika.ConnectionParameters(host='<192.168.100.1>'))
```
Аналогично, и в consumer.py.  
4. Запуск скриптов:  
`python3 producer.py`  
`http://<192.168.100.1>:15672`  
5. Запуск потребителя:
`python3 consumer.py`
# Задание 3
1. Подготовка виртуальных машин  
Создание Vagrantfile для двух машин:
```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.define "rmq01" do |rmq01|
    rmq01.vm.hostname = "rmq01"
    rmq01.vm.network "private_network", ip: "192.168.0.10"
  end

  config.vm.define "rmq02" do |rmq02|
    rmq02.vm.hostname = "rmq02"
    rmq02.vm.network "private_network", ip: "192.168.0.11"
  end
end
```
Запуск виртуальных машин:  
```
vagrant up
```
2. Настройка файла hosts
Вход в каждую ВМ:  
`vagrant ssh rmq01`  
`vagrant ssh rmq02`  
Редакция и проверка файлов:  
![image](https://github.com/user-attachments/assets/7a1b3f14-1462-4852-952c-2ea6c0e01465)  
3. Установка RabbitMQ  
4. Настройка кластера RabbitMQ  
* в ВМ1:  
```
sudo rabbitmqctl stop_app
sudo rabbitmqctl reset
sudo rabbitmqctl join_cluster rabbit@rmq01
sudo rabbitmqctl start_app
```
* в ВМ2:  
```
sudo rabbitmqctl stop_app
sudo rabbitmqctl reset
sudo rabbitmqctl join_cluster rabbit@rmq02
sudo rabbitmqctl start_app
```
Проверка статуса кластера  
```
sudo rabbitmqctl cluster_status
```
5. Создание политики ha-all
```
sudo rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'
```
6.  Веб-интерфейс RabbitMQ и команды cluster_status
```
sudo rabbitmq-plugins enable rabbitmq_management
```
7. Запуск producer.py и вывод команд  
`python3 producer.py`  
`sudo rabbitmqadmin get queue='hello'`  
8. Отключение одной из нод и тестирование consumer.py  
`vagrant halt rmq01`  
`connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.100.1'))`   
`python3 consumer.py`  
