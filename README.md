# Vulnerabilities_and_attacks

**Кolesnikov Aleksandr**  


## Задание 1
### 
Сканирование Nmap

- Базовое сканирование:

```nmap -sV -A 192.168.0.112```

- Дополнительно:
```nmap -p- -T4 192.168.0.112```

Какие сетевые службы разрешены

- В ходе сканирования были обнаружены следующие открытые порты и сервисы:

```Порт	Сервис	Версия
21	FTP	vsftpd 2.3.4
22	SSH	OpenSSH 4.7p1
23	Telnet	Linux telnetd
25	SMTP	Postfix smtpd
53	DNS	ISC BIND
80	HTTP	Apache 2.2.8
111	rpcbind	rpcbind
139	NetBIOS	Samba smbd
445	SMB	Samba smbd 3.0.20
512	rexec	remote exec
513	rlogin	rlogin
514	rsh	remote shell
1099	Java RMI	Java RMI Registry
1524	bindshell	root shell
2049	NFS	NFS service
2121	FTP	ProFTPD 1.3.1
3306	MySQL	MySQL 5.0.51a
5432	PostgreSQL	PostgreSQL 8.3.x
5900	VNC	VNC server
6000	X11	X server
6667	IRC	UnrealIRCd
8009	AJP13	Apache JServ
8180	HTTP	Apache Tomcat
```

###Обнаруженные уязвимости
1. vsftpd 2.3.4 Backdoor

- В версии vsftpd 2.3.4 присутствует встроенный backdoor, позволяющий получить shell-доступ.

2. UnrealIRCd Backdoor

- Вредоносно модифицированная версия UnrealIRCd содержит backdoor для удалённого выполнения команд.

3. Samba "username map script"

- Позволяет выполнить произвольный код от имени root через специально сформированный запрос.


На Metasploitable присутствуют слабые или пустые пароли.

R-services (rlogin/rsh/rexec)

Очень устаревшие и небезопасные сервисы удалённого доступа, передающие данные без шифрования.


- Metasploitable специально содержит большое количество уязвимых сервисов для обучения тестированию безопасности.
Наиболее опасными оказались:

``` vsftpd backdoor
UnrealIRCd backdoor
Samba RCE
```
Также система содержит множество устаревших и небезопасных сетевых служб.

## Задание 2
### SYN Scan

- Команда:

```
nmap -sS 192.168.0.112
```
Особенности трафика

Nmap отправляет TCP SYN-пакет, но не завершает полноценное TCP-соединение.

- Схема:

```
Клиент → SYN
Сервер → SYN/ACK
Клиент → RST
```

Ответ сервера:

- Open - SYN/ACK
- Closed - RST

Это «полуоткрытое» сканирование.
Оно быстрее и менее заметно для логирования.

### FIN Scan

Команда:

nmap -sF 192.168.1.105
Особенности трафика

Отправляется TCP-пакет с установленным флагом FIN без предварительного соединения.

Ответ сервера
Состояние порта	Ответ
Open	Нет ответа
Closed	RST
Особенность

Основано на RFC TCP.
Некоторые ОС игнорируют такие пакеты.

Xmas Scan

Команда:

nmap -sX 192.168.1.105
Особенности трафика

Устанавливаются флаги:

FIN + PSH + URG

Пакет выглядит «аномально», поэтому называется Xmas scan — пакет «светится как ёлка».

Ответ сервера
Состояние порта	Ответ
Open	Нет ответа
Closed	RST
Особенность

Используется для обхода некоторых фильтров и IDS.

UDP Scan

Команда:

nmap -sU 192.168.1.105
Особенности трафика

Nmap отправляет UDP-пакеты на порты.

Так как UDP не использует handshake, определить состояние сложнее.

Ответ сервера
Состояние порта	Ответ
Open	Нет ответа или UDP-ответ
Closed	ICMP Port Unreachable
Особенность

Очень медленное сканирование по сравнению с TCP.

Что видно в Wireshark
SYN Scan

Видны:

SYN
SYN/ACK
RST

Полного TCP handshake нет.

FIN/Xmas Scan

Видны нестандартные TCP-флаги:

FIN
FIN+PSH+URG

Закрытые порты отвечают RST.

UDP Scan

Видны:

UDP datagrams
ICMP Destination Unreachable
Общий вывод

Различные режимы Nmap отличаются:

Тип сканирования	Как определяется открытый порт
SYN	SYN/ACK
FIN	Отсутствие ответа
Xmas	Отсутствие ответа
UDP	ICMP или UDP response

SYN scan — самый практичный и быстрый.
FIN/Xmas применяются для обхода фильтрации.
UDP scan наиболее шумный и медленный, но позволяет находить UDP-сервисы.
