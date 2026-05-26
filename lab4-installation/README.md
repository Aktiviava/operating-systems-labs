# Лабораторная работа №4: RabbitMQ — общение двух процессов

## Цель работы

Изучить брокер сообщений RabbitMQ и реализовать обмен сообщениями между двумя независимыми процессами (отправитель и получатель) с использованием очереди.

## Выбор технологии

Для реализации межпроцессного взаимодействия выбран **RabbitMQ** — популярный message broker, поддерживающий протокол AMQP. Альтернатива (Kafka) более сложна в установке и настройке, RabbitMQ легче подходит для учебных целей.

## Ход выполнения

Установка RabbitMQ

sudo apt update
sudo apt install -y rabbitmq-server
sudo systemctl enable rabbitmq-server
sudo systemctl start rabbitmq-server

Скриншот 1: Установка и запуск RabbitMQ

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab4-installation/photo_2026-05-26%2022.53.07.jpeg?raw=true

Скриншот 2: Установка Python и библиотеки pika

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab4-installation/photo_2026-05-26%2022.53.11.jpeg?raw=true

Создание отправителя (producer.py)

#!/usr/bin/env python3
import pika
import time

print("=" * 50)
print("RabbitMQ - Отправитель (Producer)")
print("=" * 50)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='lab_queue')

for i in range(1, 11):
    message = f"Сообщение №{i} | Время: {time.strftime('%H:%M:%S')}"
    
    channel.basic_publish(
        exchange='',
        routing_key='lab_queue',
        body=message
    )
    
    print(f" [x] Отправлено: {message}")
    time.sleep(1)

print("\n[+] Все 10 сообщений отправлены!")
connection.close()

Создание получателя (consumer.py)

#!/usr/bin/env python3
import pika
import time

print("=" * 50)
print("RabbitMQ - Получатель (Consumer)")
print("=" * 50)
print("Ожидание сообщений... (нажмите Ctrl+C для остановки)")
print("-" * 50)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='lab_queue')

def callback(ch, method, properties, body):
    message = body.decode('utf-8')
    print(f" [✓] Получено: {message}")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='lab_queue', on_message_callback=callback)

print(" [*] Ожидание сообщений...")
print("")

try:
    channel.start_consuming()
except KeyboardInterrupt:
    print("\n\n[!] Получатель остановлен")
    connection.close()

Скриншот 3: Были открыты два окна терминала, в каждом подключение к ВМ

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab4-installation/photo_2026-05-26%2022.53.17.jpeg?raw=true

Скриншот 4: Отправитель отправил 10 сообщений в очередь. Получатель успешно получил все сообщения

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab4-installation/photo_2026-05-26%2022.53.21.jpeg?raw=true

Вывод

В ходе работы был настроен брокер сообщений RabbitMQ, написаны два независимых процесса (отправитель и получатель), которые успешно обмениваются сообщениями через очередь. Демонстрируется асинхронное взаимодействие процессов без прямого соединения друг с другом — только через брокера.
