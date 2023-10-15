# Prometheus

**Prometheus** — это open-source набор инструментов, предназначенный для оповещения о каких-либо событиях, происходящих на хостах, мониторингом которых он занимается

Подходит для:
- обработки данных, представленных во временны́х рядах
- мониторинга аппаратных ресурсов
- мониторинга динамичной сервис- ориентированной архитектуры

**История Prometheus**
- Впервые разработан для внутреннего использования проектом SoundCloud
- С момента своего появления в 2012 г. Prometheus был взят на вооружение многими крупными компаниями: DigitalOcean, Ericsson, CoreOS, Weaveworks, Red Hat и Google
- Имеет активную команду разработчиков и комьюнити
- В настоящий момент проект развивается независимо от какой-либо компании

**Принцип работы Prometheus**
- Prometheus читает таблицу целей (targets) — список объектов, которые надо мониторить, и периодичность их опроса
- После этого Prometheus отправляет HTTP-запрос на каждый эндпоинт и получает ответ с набором из метрик. У каждой метрики есть название, значение и лейблы
- Полученный ответ складывается в базу данных, где к данным о метрике добавляются отметка времени её получения и лейблы объекта, с которого она была взята
- Service Discovery — встроенный в Prometheus механизм обнаружения сервисов, позволяющий «из коробки» получать данные для создания таблицы целей

**Установка Prometheus**

Ansible

https://github.com/jo-os/diplom/tree/main/Ansible/roles/prometheus

Создайте пользователя Prometheus:
```
sudo useradd --no-create-home --shell /bin/false prometheus
```
Найдите последнюю версию Prometheus на :GitHub
```
wget https://github.com/prometheus/prometheus/releases/download/v2.40.1/ prometheus-2.40.1.linux-386.tar.gz
```
Извлеките архив и скопируйте файлы в необходимые директории:
```
mkdir /etc/prometheus
mkdir /var/lib/prometheus**
cp ./prometheus promtool /usr/local/bin
cp -R ./console_libraries/ /etc/prometheus/
cp -R ./consoles/ /etc/prometheus/
cp ./prometheus.yml /etc/promehteus
```
Передайте права на файлы пользователю Prometheus:
```
chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
```
Запустите и проверьте результат:
```
/usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path
/var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --
web.console.libraries=/etc/prometheus/console_libraries
```
Создание сервиса для работы с Prometheus
```
nano /etc/systemd/system/prometheus.service
```
Вставьте в файл сервиса следующее содержимое:
```
[Unit] 
Description=Prometheus Service Netology Lesson 9.4 
After=network.target 
[Service] 
User=prometheus 
Group=prometheus 
Type=simple 
ExecStart=/usr/local/bin/prometheus \ 
--config.file /etc/prometheus/prometheus.yml \ 
--storage.tsdb.path /var/lib/prometheus/ \ 
--web.console.templates=/etc/prometheus/consoles \ 
--web.console.libraries=/etc/prometheus/console_libraries 
ExecReload=/bin/kill -HUP $MAINPID Restart=on-failure 
[Install] 
WantedBy=multi-user.target
```
Передайте права на файл:
```
chown -R prometheus:prometheus /var/lib/prometheus
```
Тестирование сервиса Prometheus

Пропишите автозапуск:
```
sudo systemctl enable prometheus
```
Запустите сервис:
```
sudo systemctl start prometheus
```
Проверьте статус сервиса:
```
sudo systemctl status prometheus
```

Установка Node Exporter

Скачайте архив с и извлеките его:
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
```
Перейдите в появившуюся папку:
```
cd node_exporter-*.*-amd64 
./node_exporter
```
Node Exporter помогает извлекать данные с хоста, на котором установлен.

Скопируйте Node Exporter в папку Prometheus:
```
mkdir /etc/prometheus/node-exporter 
cp ./* /etc/prometheus/node-exporter
```
Передайте права на папку пользователю Prometheus:
```
chown -R prometheus:prometheus /etc/prometheus/node-exporter
```
Создайте сервис для работы с Node Exporter
```
nano /etc/systemd/system/node-exporter.service
```
Вставьте в файл сервиса следующее содержимое:
```
[Unit] 
Description=Node Exporter Lesson 9.4 
After=network.target 
[Service] 
User=prometheus 
Group=prometheus 
Type=simple 
ExecStart=/etc/prometheus/node-exporter/node_exporter 
[Install] 
WantedBy=multi-user.target
```
Пропишите автозапуск:
```
sudo systemctl enable node-exporter
```
Запустите сервис:
```
sudo systemctl start node-exporter
```
Проверьте статус сервиса:
```
sudo systemctl status node-exporter
```
Добавление Node Exporter в Prometheus

Отредактируйте конфигурацию Prometheus:
```
nano /etc/prometheus/prometheus.yml
```
Добавьте в scrape_config адрес экспортёра:
```
scrape_configs: 
— job_name: 'prometheus' 
scrape_interval: 5s 
static_configs: 
— targets: ['localhost:9090', 'localhost:9100']
```
Перезапустите Prometheus:
```
systemctl restart prometheus
```
**Grafana**

**Grafana** — программное обеспечение для визуализации и аналитики с открытым исходным кодом

**Установка Grafana**

Скачайте и установите deb-пакет:
```
wget https://dl.grafana.com/oss/release/grafana_9.2.4_amd64.deb
dpkg -i grafana_9.2.4_amd64.deb
```
Включите автозапуск и запустите сервер Grafana:
```
systemctl enable grafana-server
systemctl start grafana-server
systemctl status grafana-server
```
Проверьте статус, перейдя по адресу: https://<наш сервер>:3000

Стандартный логин и пароль: admin/admin

**Atlermanager**

Программное обеспечение, которое позволяет: обрабатывать оповещения, отправляемые из клиентских приложений консолидировать эти оповещения перенаправлять их по необходимым каналам доставки сообщений

Установка Alertmanager

Скачайте последнюю версию приложения с GitHub
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
tar -xvf alertmanager-*linux-amd64.tar.gz
```
Скопируйте содержимое архива в папки:
```
sudo cp ./alertmanager-*.linux-amd64/alertmanager /usr/local/bin
sudo cp ./alertmanager-*.linux-amd64/amtool /usr/local/bin
```
Скопируйте config-файл в папку Prometheus:
```
sudo cp ./alertmanager-*.linux-amd64/alertmanager.yml /etc/prometheus
```
Передайте пользователю Prometheus права на файл:
```
sudo chown -R prometheus:prometheus /etc/prometheus/alertmanager.ym
```
Создайте сервис для работы с Node Exporter:
```
nano /etc/systemd/system/prometheus-alertmanager.service
```
Вставьте в файл сервиса следующее содержимое
```
[Unit]
Description=Alertmanager Service
After=network.target
[Service]
EnvironmentFile=-/etc/default/alertmanager
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/prometheus/alertmanager.yml
--storage.path=/var/lib/prometheus/alertmanager $ARGS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
Пропишите автозапуск:
```
sudo systemctl enable prometheus-alertmanager
```
Запустите сервис:
```
sudo systemctl start prometheus-alertmanager
```
Проверьте статус сервиса:
```
sudo systemctl status prometheus-alertmanage
```
Подключение Prometheus к Alertmanager

Добавьте в сonfig-файл Prometheus подключение к Alertmanager:
```
sudo nano /etc/prometheus/prometheus.yml
```
Перезапустите Prometheus:
```
alerting:
alertmanagers:
- static_configs:
- targets: # Можно указать как targets: [‘localhost”9093’]
- localhost:9093
```
Приведите раздел Alertmanager configuration к виду:
```
sudo systemctl restart prometheus
systemctl status prometheus
```
Создание правила оповещения

Создайте файл с правилом оповещения:
```
sudo nano /etc/prometheus/netology-test.yml
```
Приведите файл к следующему виду:
```
groups: # Список групп
- name: netology-test # Имя группы
rules: # Список правил текущей группы
- alert: InstanceDown # Название текущего правила
expr: up == 0 # Логическое выражение
for: 1m # Сколько ждать отбоя сработки перед отправкой оповещения
labels:
severity: critical # Критичность события
annotations: # Описание
description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.' # Полное описание алерта
summary: Instance {{ $labels.instance }} down # Краткое описание алерта
```
Подключение правила к Prometheus

Отредактируйте prometheus.yml:
```
sudo nano /etc/prometheus/prometheus.yml
```
Добавьте в раздел **rule_files** запись:
```
- "netology-test.yml"
```
Перезапустите Prometheus:
```
sudo systemctl restart prometheus
systemctl status prometheus
```
Откройте сonfig-файл Alertmanager:
```
sudo nano /etc/prometheus/alertmanager.yml
```
Приведите его к следующему виду:
```
global:
route:
group_by: ['alertname'] # Параметр группировки оповещений — по имени
group_wait: 30s # Сколько ждать восстановления, перед тем как отправить первое оповещение
group_interval: 10m # Сколько ждать, перед тем как дослать оповещение о новых сработках по текущему алерту
repeat_interval: 60m # Сколько ждать, перед тем как отправить повторное оповещение
receiver: 'email' # Способ, которым будет доставляться текущее оповещение
receivers: # Настройка способов оповещения
- name: 'email'
email_configs:
- to: 'yourmailto@todomain.com'
from: 'yourmailfrom@fromdomain.com'
smarthost: 'mailserver:25'
auth_username: 'user'
auth_identity: 'user'
auth_password: 'paS$w0rd
```
После внесения всех изменений перезапустите Alertmanager и выключите экспортёр, стоящий на сервере Prometheus:
```
sudo systemctl restart prometheus-alertmanager
systemctl status prometheus-alertmanager
sudo systemctl stop node-exporter
systemctl status node-exporter
```
Теперь можете проверить интерфейсы Prometheusи Alertmanager, расположенные на стандартных портах 9090 и 9093

**Мониторинг Docker в Prometheus**

Docker «из коробки» поддерживает мониторинг с помощью Prometheus. Чтобы включить выгрузку данных на хосте с Docker, нужно создать файл daemon.json:
```
sudo nano /etc/docker/daemon.json
```
Укажите внутри него IP и порт, на котором Docker будет отдавать метрики, в формате «server_ip:port»
```
{
"metrics-addr" : "ip_нашего_сервера:9323",
"experimental" : true
}
```
Перезапустите Docker:
```
sudo systemctl restart docker && systemctl status docker
```
Для проверки можно открыть адрес http://server_ip:port/metrics

Чтобы поставить только что организованный эндпоинт на мониторинг, необходимо отредактировать файл prometheus.yml:
```
sudo nano /etc/prometheus/prometheus.yml
```
В раздел static_configs добавьте новый эндпоинт
```
static_configs:
- targets: ['localhost:9090', 'localhost:9100', 'server_ip:9323']
```
Перезапустите Prometheus:
```
sudo systemctl restart prometheus
systemctl status prometheus
```
