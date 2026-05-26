# Лабораторная работа №2: Параллелизм и примитивы синхронизации в C++

## Цель работы

Изучить механизмы параллельного программирования в C++: создание потоков, синхронизацию доступа к общим ресурсам с помощью мьютексов и условных переменных. Реализовать классическую задачу "Производитель-Потребитель".

## Теоретическая часть

### Используемые примитивы синхронизации

| Примитив | Назначение |
|----------|------------|
| `std::mutex` | Взаимное исключение — защита общего ресурса от одновременного доступа |
| `std::condition_variable` | Условная переменная — ожидание выполнения определённого условия |
| `std::unique_lock` | RAII-обёртка для мьютекса с автоматическим захватом/освобождением |
| `std::thread` | Создание и управление потоками |

### Задача

Два потока-производителя генерируют случайные числа и помещают их в буфер ограниченного размера (5 элементов). Три потока-потребителя забирают числа из буфера и обрабатывают их. Если буфер полон, производители ждут. Если буфер пуст, потребители ждут.

## Ход выполнения

### 1. Создание папки и файла

mkdir -p ~/labs/lab2
cd ~/labs/lab2
nano producer_consumer.cpp

Скриншот 1: Создание папки и файла для 2 лабораторной

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab2-installation/photo_2026-05-26%2022.46.15.jpeg?raw=true

Исходный код

#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>
#include <random>

using namespace std;

// Общие данные для потоков
queue<int> buffer;
const int MAX_BUFFER = 5;
mutex mtx;
condition_variable cv_prod;
condition_variable cv_cons;
bool finished = false;

// Генератор случайных чисел
random_device rd;
mt19937 gen(rd());
uniform_int_distribution<> dis(1, 100);

// Функция производителя
void producer(int id) {
    for (int i = 0; i < 10; i++) {
        int value = dis(gen);
        
        unique_lock<mutex> lock(mtx);
        
        cv_prod.wait(lock, []{ return buffer.size() < MAX_BUFFER; });
        
        buffer.push(value);
        cout << "[Producer " << id << "] added: " << value 
             << " | buffer size: " << buffer.size() << endl;
        
        cv_cons.notify_one();
        
        lock.unlock();
        
        this_thread::sleep_for(chrono::milliseconds(100 + rand() % 200));
    }
}

// Функция потребителя
void consumer(int id) {
    while (!finished) {
        unique_lock<mutex> lock(mtx);
        
        cv_cons.wait(lock, []{ return !buffer.empty() || finished; });
        
        if (!buffer.empty()) {
            int value = buffer.front();
            buffer.pop();
            cout << "[Consumer " << id << "] took: " << value 
                 << " | buffer size: " << buffer.size() << endl;
            
            cv_prod.notify_one();
        }
        
        lock.unlock();
        
        this_thread::sleep_for(chrono::milliseconds(150 + rand() % 300));
    }
}

int main() {
    cout << "=== Producer-Consumer with Mutex and Condition Variable ===" << endl;
    cout << "Buffer size: " << MAX_BUFFER << endl;
    cout << "============================================================" << endl;
    
    thread prod1(producer, 1);
    thread prod2(producer, 2);
    thread cons1(consumer, 1);
    thread cons2(consumer, 2);
    thread cons3(consumer, 3);
    
    prod1.join();
    prod2.join();
    
    {
        lock_guard<mutex> lock(mtx);
        finished = true;
        cv_cons.notify_all();
    }
    
    cons1.join();
    cons2.join();
    cons3.join();
    
    cout << "\n=== Program finished ===" << endl;
    
    return 0;
}

Скриншот 2: Код 2 лабораторной в редакторе

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab2-installation/photo_2026-05-26%2022.46.25.jpeg?raw=true

Скриншот 3: Проверка работы и запуск программы

https://github.com/Aktiviava/operating-systems-labs/blob/main/lab2-installation/photo_2026-05-26%2022.46.30.jpeg?raw=true

Программа успешно отработала без ошибок. Потоки синхронизированы — производители ждали, когда буфер освободится, потребители ждали появления данных.

Вывод

В ходе работы была реализована программа, демонстрирующая корректную синхронизацию потоков. Использование мьютекса предотвратило состояние гонки, а условные переменные позволили потокам эффективно ожидать изменения состояния. Отсутствуют взаимоблокировки — программа завершается корректно.
