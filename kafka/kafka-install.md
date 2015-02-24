#Настройка кластера Kafka

Для начала нам нужны минимум три хоста. Я взял их в dogitalocean.com. Взял ubuntu 14.10 с гигабайтом пямяти. Гигабайта все равно было мало.
Трем машинам я дал три доменных имя kf1.dom.ru, kf2.dom.ru, kf3.dom.ru. Чтобы было удобнее и не вбивать ip адреса.

##Перед началом.

1. Проапгрейдил все пакеты:
 ```
 sudo apt-get update
 sudo apt-get upgrade
 ```
2. Создал пользователя kafka.
3. Поставил java. Я вобщем не большой знаток отличий разных версий java, по этому:
```
sudo apt-get openjdk-7-jre-headless
```

На этом предварительные настройки закончились.

##Настройка.

Это будет кластер с тримя брокерами kafka и тримя zookeeperами. Все дальнейшие команды выполняем из под пользователя kafka.

###Качаем распаковываем.
```
 wget http://apache-mirror.rbc.ru/pub/apache/kafka/0.8.2.0/
 tar zxvf kafka_2.9.2-0.8.2.0.tgz
```
Настраивать будем в директории kafka_2.9.2-0.8.2.0.

###Zookeeper
```
mkdir -p /tmp/zookeeper
echo номер_ноды > /tmp/zookeeper/myid
```
Номер ноды это машины в кластере 1,2,3. То есть для первой машины: ```echo 1 > /tmp/zookeeper/myid```. Для остальных аналогично. Если этого не сделать zookeeper не запустится.

Редактируем конфиг config/zookeeper.property. Надо в коне добавить строки:
```
server.1=kf1.dom.ru:2888:3888
server.2=kf2.dom.ru:2888:3888
server.3=kf3.dom.ru:2888:3888
#add here more servers if you want
initLimit=5
syncLimit=2
```
Можно запустить zookeeper:
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```
Если у вам много памяти то все заработает. Если меньше, надо умерить аппетиты zookeepera, выставляем переменную KAFKA_HEAP_OPTS:``` export KAFKA_HEAP_OPTS="-Xmx256M -Xms256M"```. Для экспериметов этого будет достаточно.
Теперь повторяем эту настройку на всех хостах (не забываем менять id). Запускаем zookeeper на оставшихся машинах. В логах на консоль он будет выводить много разных сообщений о выборах мастера и все такое прочее. 

###Kafka
Редактируем config/server.properties:
```
broker.id=номер_ноды
....
zookeeper.connect=kf1.dom.ru:2181,kf2.dom.ru:2181,kf3.dom.ru:2181
```
Запускаем kafkу:
```
bin/kafka-server-start.sh config/server.properties
```
Повторяем на оставшихся машинах кластера и все должно заработать.

###Работа
Протестируем.
Создаем топик на первой машине:
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test2
```
Создали топик тест. 
Теперь на другой машине скажем на kf3:
```
bin/kafka-topics.sh --zookeeper localhost:2181 --list
```
Видим список наших топиков. 
Один test2. Опиши нам его:
```
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test2
Topic:test2	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: test2	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 3,1,2
	Topic: test2	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
	Topic: test2	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 1,3,2
```
Вроде все ок.
Запускаем producer:
```
bin/kafka-console-producer.sh --broker-list kf1.miared.ru:9092,kf2.miared.ru:9092,kf3.miared.ru:9092 --topic test2
```
Набираем на клавиатуре все уходит в kafkу. 
Запускаем consumer - на другой машине естественно :):
```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic test2
````
Должен выводить то, что вы вводите на консоле. Можно запустить несколько consumeroв на нескольких машинах. 

Все работает ! 

Дальше читайте kafka.apache.org там все есть.

И на последок скриптец который все пускает:
```
#!/bin/bash

export KAFKA_HEAP_OPTS="-Xmx256M -Xms256M"

nohup bin/zookeeper-server-start.sh config/zookeeper.properties > zookeeper.console &
nohup bin/kafka-server-start.sh config/server.properties > kafka.console &

```











