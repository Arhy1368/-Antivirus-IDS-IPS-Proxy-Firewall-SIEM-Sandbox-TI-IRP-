# -Antivirus-IDS-IPS-Proxy-Firewall-SIEM-Sandbox-TI-IRP-
Understanding how GIS works
Используемые инструменты:
Инструмент:	                     Назначение:
Windows Defender / Kaspersky	   Антивирусная защита (сигнатурное и поведенческое обнаружение)
UFW / iptables	                 Брандмауэр (Firewall)
Suricata	                       IDS/IPS (обнаружение и предотвращение вторжений)
ELK Stack	                       SIEM (сбор, нормализация и корреляция логов)
Nmap	                           Сканер безопасности
Kali Linux	                     Симуляция атак
Metasploitable 2	               Уязвимая цель для тестирования
VirtualBox	                     Виртуализация стенда

1. Антивирус (Antivirus / EDR)
Цель: Увидеть, как антивирус обнаруживает вредоносное ПО.
Теория: Антивирусы работают двумя основными способами:
Сигнатурный метод: Сравнивает файлы с базой известных вредоносных сигнатур (как отпечатки пальцев).
Поведенческий метод: Анализирует действия программы (если она пытается изменить системные файлы или зашифровать данные — это подозрительно).

Практика (1): Тест с EICAR (безопасный тестовый вирус).
EICAR — это стандартный тестовый файл, который антивирусы распознают как угрозу, но он абсолютно безопасен.
Создайте текстовый файл test.txt.
Вставьте в него следующую строку (без кавычек):
text
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
Сохраните файл.
<img width="856" height="665" alt="image" src="https://github.com/user-attachments/assets/b41bd52e-d8ff-40dc-8b24-cf5cd81a51e5" />
Ожидаемый результат: Антивирус (Windows Defender, Kaspersky или любой другой) должен немедленно среагировать и поместить файл в карантин. Это демонстрирует работу сигнатурного анализатора.

Практика (2): Проверка файла в VirusTotal.
Перейдите на сайт VirusTotal.
Загрузите любой подозрительный файл (или просто .exe установщика программы).
Посмотрите, сколько антивирусов его обнаружили.
<img width="1888" height="873" alt="image" src="https://github.com/user-attachments/assets/bd735f80-329f-4032-9f1f-e22ad76d29c6" />
Что показывает: VirusTotal показывает результат сканирования 60+ антивирусами одновременно. Это демонстрирует, как работает облачная антивирусная проверка.

2. Брандмауэр (Firewall)
Цель: Научиться создавать правила, разрешающие и блокирующие трафик.
Теория: Брандмауэр — это "дверь" в вашу сеть. Он пропускает или блокирует трафик на основе правил (IP-адреса, порты, протоколы).

Практика (1): Блокировка порта (UFW на Linux).
На Ubuntu Server (10.0.0.1):
Проверьте статус:
bash
sudo ufw status
Разрешите SSH и веб-трафик:
bash
sudo ufw allow ssh
sudo ufw allow 80
Включите брандмауэр:
bash
sudo ufw enable
Проверьте, что порт 80 открыт:
bash
sudo ufw status verbose
<img width="704" height="250" alt="image" src="https://github.com/user-attachments/assets/7e0dfae5-6a3e-49f1-a722-436bf1d3e20f" />
Заблокируйте доступ к порту 80 с конкретного IP (например, с вашего атакующего Kali):
bash
sudo ufw deny from 10.0.0.2 to any port 80
Проверьте с Kali доступность порта 80 через nmap:
bash
nmap -p 80 10.0.0.1
Ожидаемый результат: Nmap покажет, что порт 80 фильтруется (filtered) или закрыт.
<img width="581" height="254" alt="image" src="https://github.com/user-attachments/assets/ab80dfc2-e618-4a8d-9d7a-fa130e888281" />
Проверьте, что именно работает на порту 80 на Ubuntu.
bash
curl http://10.0.0.1:80
Если вы увидите HTML-код или ответ, значит, это ваш веб-интерфейс (например, Kibana или что-то еще).

Проверьте, открыт ли порт 443 (HTTPS) на Ubuntu.
bash
nmap -p 443 10.0.0.1
Если порт 443 открыт, значит, веб-интерфейс доступен по защищенному протоколу. Это хорошо для безопасности.

Если порт 80 на Ubuntu не нужен (например, вы используете только HTTPS), закройте его.
bash
sudo ufw deny 80

3. IDS/IPS (Suricata)
Цель: Увидеть, как система обнаруживает сетевые атаки.
Теория: IDS (Intrusion Detection System) — обнаруживает и уведомляет. IPS (Intrusion Prevention System) — обнаруживает и блокирует. Suricata умеет делать и то, и другое.

Практика: Установка Suricata и обнаружение сканирования.
На Ubuntu Server установите Suricata:
bash
sudo apt install suricata -y
sudo suricata-update
Запустите Suricata:
bash
sudo suricata -i eth0 -c /etc/suricata/suricata.yaml
<img width="703" height="79" alt="image" src="https://github.com/user-attachments/assets/76f97a02-2433-46a2-adb3-0b81f2a532b1" />
На Kali запустите сканирование портов:
bash
nmap 10.0.0.1
<img width="592" height="306" alt="image" src="https://github.com/user-attachments/assets/7c544fd4-1a6a-4950-8e02-320013b2543a" />
Проверьте логи Suricata:
bash
sudo tail -f /var/log/suricata/fast.log
<img width="876" height="349" alt="image" src="https://github.com/user-attachments/assets/e117ba00-b058-4914-a321-56989d338977" />
Ожидаемый результат: Вы увидите сообщения о сканировании портов в логах Suricata.

4. SIEM на базе ELK Stack (Windows + Docker)

Шаг 1: Установка Docker Desktop на Windows

1.1. Скачивание и установка
Перейдите на официальный сайт Docker: https://www.docker.com/products/docker-desktop/
Скачайте установщик для Windows.
Запустите установку, следуя инструкциям.
Важно: После установки перезагрузите компьютер.
Запустите Docker Desktop. Вы должны увидеть иконку в системном трее.
1.2. Проверка установки Docker
Откройте PowerShell (или командную строку) и выполните:
powershell
docker --version
Вы должны увидеть версию Docker.

🛠️ Шаг 2: Развертывание ELK Stack

2.1. Клонирование репозитория
Откройте PowerShell.
Перейдите в папку, где вы будете хранить проект ELK (например, C:\ELK):
powershell
mkdir C:\ELK
cd C:\ELK
Клонируйте официальный репозиторий ELK:
powershell
git clone https://github.com/deviantony/docker-elk.git
2.2. Настройка пароля Elasticsearch
Перейдите в папку с ELK:
powershell
cd docker-elk
Откройте файл .env в Блокноте:
powershell
notepad .env
Найдите строку ELASTIC_PASSWORD и задайте свой пароль:
text
ELASTIC_PASSWORD=changeme
(Замените changeme на свой надёжный пароль и запишите его!)
2.3. Запуск контейнеров
powershell
docker-compose up -d
Дождитесь загрузки трёх контейнеров: elasticsearch, logstash, kibana.
<img width="895" height="143" alt="image" src="https://github.com/user-attachments/assets/82da9639-4459-417b-b6a8-e7272214498b" />
2.4. Проверка работы контейнеров
powershell
docker ps
Вы должны увидеть три запущенных контейнера.
2.5. Проверка Kibana
Откройте браузер и перейдите по адресу:
text
http://localhost:5601
Логин: elastic
Пароль: тот, который вы задали в .env (например, changeme).
Если вы видите страницу Kibana — всё работает!

🛠️ Шаг 3: Создание пользователя для Winlogbeat

Этот шаг безопаснее, чем использовать суперпользователя elastic.
Откройте PowerShell (или командную строку).
Войдите в контейнер Elasticsearch:
powershell
docker exec -it docker-elk-elasticsearch-1 bash
Создайте пользователя winlogbeat_user с ролью beats_admin:
bash
elasticsearch-users useradd winlogbeat_user -p your_password -r beats_admin
Замените your_password на надёжный пароль и запишите его!
Выйдите из контейнера:
bash
exit

🛠️ Шаг 4: Скачивание и установка Winlogbeat

4.1. Скачивание Winlogbeat
Перейдите на официальную страницу загрузки:
https://www.elastic.co/downloads/beats/winlogbeat
Скачайте ZIP-архив для Windows (выберите версию, совместимую с вашей системой).
Распакуйте архив в папку C:\Program Files\Winlogbeat.
4.2. Установка Winlogbeat как службы
Откройте PowerShell от имени администратора.
Перейдите в папку с Winlogbeat:
powershell
cd "C:\Program Files\Winlogbeat"
Установите службу:
powershell
.\install-service-winlogbeat.ps1
или PowerShell.exe -ExecutionPolicy UnRestricted -File .\install-service-winlogbeat.ps1

🛠️ Шаг 5: Настройка Winlogbeat

5.1. Редактирование файла конфигурации
Откройте файл winlogbeat.yml в Блокноте:
powershell
notepad winlogbeat.yml
Настройте сбор логов (секция winlogbeat.event_logs):
yaml
winlogbeat.event_logs:
  - name: Application
  - name: System
  - name: Security
  - name: Windows PowerShell
Настройте отправку в Elasticsearch (секция output.elasticsearch):
yaml
output.elasticsearch:
  hosts: ["localhost:9200"]
  username: "winlogbeat_user"
  password: "ваш_пароль_который_вы_создали"
Настройте Kibana (секция setup.kibana):
yaml
setup.kibana:
  host: "localhost:5601"
Сохраните файл (Ctrl+S) и закройте Блокнот.
5.2. Проверка конфигурации
powershell
.\winlogbeat.exe test config -c .\winlogbeat.yml -e
Ожидаемый вывод: Config OK.
5.3. Загрузка дашбордов
powershell
.\winlogbeat.exe setup -e
5.4. Запуск службы
powershell
Start-Service winlogbeat
5.5. Проверка статуса службы
powershell
Get-Service winlogbeat
Статус должен быть Running.
Быстрое восстановление
Если ничего не помогает, просто перезапустите всё:
powershell
cd C:\Program Files\Winlogbeat\docker-elk
docker-compose down
docker-compose up -d
<img width="930" height="366" alt="image" src="https://github.com/user-attachments/assets/bf58c588-94b3-41fa-9e6e-a71aaa6281e5" />
Подождите 3 минуты и попробуйте снова открыть http://localhost:5601
<img width="1880" height="867" alt="image" src="https://github.com/user-attachments/assets/f122c47e-d032-4204-a8cf-5f6c7ad70092" />
<img width="1879" height="861" alt="image" src="https://github.com/user-attachments/assets/82224f86-041c-40db-be9c-f556fedc238a" />
Add integrations — предлагает подключить готовые источники данных (например, облачные сервисы, AWS, Azure и т.д.). Это актуально, если у вас есть такие сервисы.
Explore on my own — откроет главный интерфейс Kibana, где вы сможете вручную настроить всё, что вам нужно: создать индекс, посмотреть логи, настроить дашборды.
Если ничего не помогает
Проверьте, занят ли порт 5601 другой программой:
powershell
netstat -ano | findstr :5601
Если есть процесс, который слушает этот порт, завершите его или настройте другой порт для Kibana в docker-compose.yml

🖥️ Шаг 6: Создание дашборда в Kibana

6.1. Проверка данных
Откройте Kibana: http://localhost:5601
В левом меню выберите Discover.
Выберите индекс winlogbeat-*. Если вы видите события — Winlogbeat работает и логи поступают.
6.2. Создание дашборда
Перейдите в раздел Dashboard → Create dashboard.
Нажмите Create new → Add from library или создайте визуализацию с нуля.
Рекомендуемые визуализации:
Визуализация:	              Запрос/Параметры:	                                 Цель:
События по времени	        X-axis: @timestamp, Y-axis: Count of records	     Показывает общую активность в системе.
Топ-пользователи	          Breakdown by: user.name	                           Выявляет, какие пользователи чаще всего фигурируют в событиях.
Успешные/Неудачные входы	  Breakdown by: winlog.event_id (4624 и 4625)	       Позволяет сразу увидеть всплеск неудачных входов (брутфорс).
Топ-процессы	              Breakdown by: process.name	                       Показывает, какие процессы генерируют больше всего событий.
Сохраните дашборд с именем Windows Events Overview.

Шаг 7: Симуляция атаки (Брутфорс RDP)

Чтобы увидеть SIEM в действии:
На вашем Windows-клиенте включите удалённый рабочий стол (RDP).
С Kali Linux (если есть в сети) выполните брутфорс-атаку с помощью Hydra:
bash
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://<IP_Windows>
Если Kali нет, можно просто вручную несколько раз ввести неверный пароль в RDP.
Через минуту проверьте ваши логи в Kibana: в Discover вы увидите множество событий Event ID 4625 (неудачный вход).

5. Сканер безопасности (Nmap / Nessus)
Цель: Увидеть разницу между сканированием портов и сканированием уязвимостей.
Теория: Nmap сканирует порты и определяет версии служб. Nessus проверяет службы на наличие известных уязвимостей (CVE).

Практика (1): Сканирование портов Nmap.
bash
nmap -sV 10.0.0.30
Практика (2): Сканирование уязвимостей через NSE (Nmap Scripting Engine).

bash
nmap --script=vuln 10.0.0.30
Ожидаемый результат: Nmap найдёт конкретные CVE, например, уязвимость в vsftpd на Metasploitable.
