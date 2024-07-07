# Балансировщик нагрузки на кластер
## Формулировка задачи
### Цель 
Необходимо организовать параллельную обработку заданий на нескольких компьютерах.

### Условия
1) Есть список неделимых заданий, каждое из которых может быть выполнено независимо от другого.
    - Задания могут иметь различный вычислительный вес, т.е. требовать при одних и тех же вычислительных ресурсах различного времени для выполнения. 
    - Считается, что этот вес нельзя узнать, пока задание не выполнено. 
2) После того, как все задания из списка выполнены, появляется новый список заданий. 
3) Количество заданий существенно превосходит количество процессоров. Программа не должна зависеть от числа компьютеров.

## Используемые технологии
  - MPI
  - POSIX Threads (threads, mutex)

## Описание решения
  1. Создаём список задач на выполнение и распределяем их почти равномерно (по количеству) на все процессы.
  2. На каждом процессе создаём три потока.
        - 2.1. Поток - рабочий.
             Выполняет таски в своём пуле, пока они не закончатся.
        - 2.2. Поток - слушатель.
             Принимает запросы на получение новых таск.
        - 2.3. Поток - запрашиватель.
             Когда в пуле остаётся 20% или менее от изначального числа задач, он начинает запрашивать задачи у других процессов, начиная со случайного и двигаясь по ним всем.

     **Правила передачи задач:**
         Запрашиваемый процесс передаёт задачи, если:
     -  Количестве задач процесса, что запрашивает в 2 и более раз меньше количества задач процесса, что отдаёт.
     -  Количество оставшихся задач в запрашиваемом процессе больше трёх.
 
        **Иначе таски не передаются и процесс отправляет запрос следующему по очереди процессу.**

3. Если поток, запрашивающий таски опросив все процессы понял, что у всех осталось 3 или меньше задач в пуле, то он перестаёт запрашивать задачи и переходит в "режим ожидания остальных запрашивающих потоков" (MPI_Barrier(MPI_COMM_WORLD)).
4. Как только все потоки, запрашивающие задачи дошли до барьера, они отправляют сигнал потоку - слушателю о необходимости завершиться, после чего завершаются сами.
5. Как только поток - рабочий завершил все таски (и остальные потоки тоже завершили свою работу), он закаанчивает своё выполнение и всё начинается заново.
