# Домашнее задание к занятию "13.Системы мониторинга"

## Обязательные задания

1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя 
платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой 
осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы
выведите в мониторинг и почему?

Нагрузка на процессор

Нагрузка на диск (скорость записи/чтения, индексные дескрипторы inodes)

Мониторинг API (доступность, ошибки)

Мониторинг RAM


#
2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, 
что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы 
можете ему предложить?

RAM - утилизация оперативной памяти; 
inodes - кол-во индексных дескрипторов, на каждый файл используется 1 inode. Если будет большое кол-во легковесных файлов-отчётов, то в какой-то момент inodes могут закончиться, при этом на диске может оставаться свободное пространство, но файлы не смогут создаваться; 
CPUia - показатель средней загрузки CPU за промежуток времени.

Такие технические детали больше помогут админам найти проблемные и "узкие" места. 

Менеджерам и клиентам важны не технические показатели, а что они получат в результате работы сервиса. 

Для определения качества обслуживания и выполнения обязанностей перед клиентом необходимо определить целевые показатели производительности. Для этого можно выделить следующие показатели:

Допустимое время недоступности сервиса (единоразовое и/или допустимое время недоступности в определённый промежуток времени)

Допустимый процент ошибок при подготовке отчётов

Среднее время ожидания ответа от сервиса при подготовке отчёта


#
3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою 
очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, 
чтобы разработчики получали ошибки приложения?

rsyslog


#
4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. 
Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 
70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?

Ошибка заключается в неверном расчете показателя SLA. Формула summ_2xx_requests / summ_all_requests учитывает только успешные ответы (2XX), игнорируя другие коды ответов, кроме 5XX и 4XX. Однако в случае отсутствия ошибок сервера (5XX) и клиентских ошибок (4XX), возможно наличие других типов ответов, таких как 3XX (редиректы), которые тоже учитываются в общем количестве запросов (summ_all_requests), но не включаются в расчет успешных запросов (summ_2xx_requests).

Таким образом, проблема заключается в том, что формула расчета SLA некорректна, потому что она включает в знаменатель все запросы, а в числитель — только те, которые завершились успешно с кодами 2XX.

Правильный подход:

Чтобы правильно рассчитать SLA, необходимо исключить из общего количества запросов те, которые не являются успешными (например, редиректы 3XX, информационные ответы 1XX и т.п.). 


#
5. Опишите основные плюсы и минусы pull и push систем мониторинга.

**Pull-системы мониторинга**

Основные принципы работы:

Pull-система собирает данные путем периодического опроса целевых объектов (серверов, приложений, сервисов и т.д.) и получения от них информации о текущем состоянии. Это напоминает ситуацию, когда кто-то "спрашивает" объект, как у него дела.

Преимущества:

Простота реализации: опрос целевого объекта относительно прост в настройке и понимании. Многие инструменты мониторинга, такие как Prometheus, работают именно по принципу pull.

Контроль над частотой опросов: можно легко управлять частотой запросов к объектам, что позволяет гибко настраивать частоту обновления данных.

Централизованное управление конфигурацией: все конфигурации находятся на стороне центрального узла мониторинга, что упрощает управление системой.

Легкость диагностики проблем: при наличии проблем с объектом мониторинга можно сразу увидеть, что он недоступен или не отвечает на запросы.

Недостатки: 

Зависимость от доступности объектов: если целевой объект недоступен, pull-система не сможет собрать данные. Это делает систему уязвимой к временным проблемам с сетевым соединением или состоянием самого объекта.

Проблема с большими инфраструктурами: с увеличением числа объектов мониторинга увеличивается нагрузка на центральный узел, так как ему приходится периодически опрашивать каждый объект. Это может стать проблемой при масштабировании системы.

Задержка в обновлении данных: данные собираются с определенной периодичностью, что может вызывать задержки между моментом возникновения события и его фиксацией в системе мониторинга.

**Push-системы мониторинга**

Основные принципы работы:
Push-система предполагает, что объекты сами отправляют данные на центральный узел мониторинга. Это похоже на ситуацию, когда объект сам сообщает, что у него произошло какое-то событие.

Преимущества:

Масштабируемость: так как каждый объект самостоятельно отправляет данные, нагрузка на центральный узел снижается. Это особенно полезно в больших распределённых системах, где число объектов велико.

Мгновенная доставка данных: объекты отправляют данные сразу после их генерации, что уменьшает задержку между событием и его регистрацией в системе мониторинга.

Независимость от сети: даже если центральный узел временно недоступен, объекты продолжают собирать и сохранять данные локально до восстановления связи.

Гибкое управление данными: объект может отправлять только те данные, которые нужны для мониторинга, что снижает избыточность передачи ненужных данных.

Недостатки: 

Сложность управления конфигурациями: конфигурация мониторинга распределена по всем объектам, что усложняет централизованное управление системой. Необходимо отдельно конфигурировать каждый агент на каждом объекте.

Риски потери данных: если связь с центральным узлом мониторинга нарушена, данные могут теряться, пока не восстановится соединение. Для решения этой проблемы часто используются буферизация и повторные попытки отправки данных.

Безопасность: push-системы требуют дополнительных мер безопасности, так как объекты должны иметь возможность передавать данные на центральный узел. Это увеличивает риск несанкционированного доступа.


#
6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

    - Prometheus 
    - TICK
    - Zabbix
    - VictoriaMetrics
    - Nagios

Prometheus: Pull

Prometheus использует модель pull, собирая метрики с экспортеров, которые запускаются на целевых объектах. Центральный сервер Prometheus периодически опрашивает эти экспортеры и получает актуальные данные.

TICK Stack (Telegraf, InfluxDB, Chronograf, Kapacitor): Push

TICK Stack основан на использовании Telegraf, который собирает данные и отправляет их в InfluxDB. Это типичный пример push-модели, где агенты собирают данные и передают их на центральный сервер.

Zabbix: гибридная

Zabbix поддерживает обе модели: pull и push. Он может опрашивать объекты (pull), а также принимать данные, отправленные агентами (push). Выбор модели зависит от настроек и конкретной конфигурации.

VictoriaMetrics: гибридная

VictoriaMetrics поддерживает оба режима: pull и push. Она может получать данные как посредством опроса (pull), так и принимать их от агентов (push). Это позволяет гибко настраивать систему под конкретные нужды.

Nagios: Pull

Nagios традиционно использует pull-модель, опрашивая целевые объекты для получения данных о состоянии и метриках. Однако существуют плагины и расширения, позволяющие реализовать push-механизмы.


#
7. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, 
используя технологии docker и docker-compose.

В виде решения на это упражнение приведите скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`). 

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`


Пришлось сделать sudo chmod -R 777 ./chronograf/data/

sudo ./sandbox up

![alt text](https://github.com/MaratKN/10-monitoring-02-systems/blob/main/1.png)


#
8. Перейдите в веб-интерфейс Chronograf (http://localhost:8888) и откройте вкладку Data explorer.
        
    - Нажмите на кнопку Add a query
    - Изучите вывод интерфейса и выберите БД telegraf.autogen
    - В `measurments` выберите cpu->host->telegraf-getting-started, а в `fields` выберите usage_system. Внизу появится график утилизации cpu.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации cpu из веб-интерфейса.

![alt text](https://github.com/MaratKN/10-monitoring-02-systems/blob/main/2.png)


#
9. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). 
Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и 
режима privileged:
```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```

После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в 
веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.

Пришлось сделать sudo chmod -R 777 /var/run/docker.sock

sudo ./sandbox up

![alt text](https://github.com/MaratKN/10-monitoring-02-systems/blob/main/3.png)

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

1. Вы устроились на работу в стартап. На данный момент у вас нет возможности развернуть полноценную систему 
мониторинга, и вы решили самостоятельно написать простой python3-скрипт для сбора основных метрик сервера. Вы, как 
опытный системный-администратор, знаете, что системная информация сервера лежит в директории `/proc`. 
Также, вы знаете, что в системе Linux есть  планировщик задач cron, который может запускать задачи по расписанию.

Суммировав все, вы спроектировали приложение, которое:
- является python3 скриптом
- собирает метрики из папки `/proc`
- складывает метрики в файл 'YY-MM-DD-awesome-monitoring.log' в директорию /var/log 
(YY - год, MM - месяц, DD - день)
- каждый сбор метрик складывается в виде json-строки, в виде:
  + timestamp (временная метка, int, unixtimestamp)
  + metric_1 (метрика 1)
  + metric_2 (метрика 2)
  
     ...
     
  + metric_N (метрика N)
  
- сбор метрик происходит каждую 1 минуту по cron-расписанию

Для успешного выполнения задания нужно привести:

а) работающий код python3-скрипта,

б) конфигурацию cron-расписания,

в) пример верно сформированного 'YY-MM-DD-awesome-monitoring.log', имеющий не менее 5 записей,

P.S.: количество собираемых метрик должно быть не менее 4-х.
P.P.S.: по желанию можно себя не ограничивать только сбором метрик из `/proc`.

2. В веб-интерфейсе откройте вкладку `Dashboards`. Попробуйте создать свой dashboard с отображением:

    - утилизации ЦПУ
    - количества использованного RAM
    - утилизации пространства на дисках
    - количество поднятых контейнеров
    - аптайм
    - ...
    - фантазируйте)
    
    ---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

