### Table of Contents
**
- [[#Overview|Overview]]
- [[#Private Network A:|Private Network A:]]
	- [[#Private Network A:#MicroTik 1|MicroTik 1]]
		- [[#MicroTik 1#MicroTik 1|MicroTik 1]]
	- [[#Private Network A:#Host 1 (Linux)|Host 1 (Linux)]]
- [[#Private Network B:|Private Network B:]]
	- [[#Private Network B:#MicroTik 2|MicroTik 2]]
	- [[#Private Network B:#Host 2 (Linux)|Host 2 (Linux)]]
	- [[#Private Network B:#MicroTik 1 and MicroTik 2 connection|MicroTik 1 and MicroTik 2 connection]]
		- [[#MicroTik 1 and MicroTik 2 connection#MicroTik 1|MicroTik 1]]
		- [[#MicroTik 1 and MicroTik 2 connection#MicroTik 2|MicroTik 2]]
		- [[#MicroTik 1 and MicroTik 2 connection#Connection between MicroTik 1 and MicroTik 2|Connection between MicroTik 1 and MicroTik 2]]
		- [[#MicroTik 1 and MicroTik 2 connection#Check that NAT is working|Check that NAT is working]]
- [[#Public Host|Public Host]]
- [[#Download Coturn on PublicHost|Download Coturn on PublicHost]]
- [[#Настроим Coturn|Настроим Coturn]]
- [[#Настроим STUN сервер|Настроим STUN сервер]]
	- [[#Настроим STUN сервер#Настроить Coturn на Host1|Настроить Coturn на Host1]]
- [[#Set up a TURN server|Set up a TURN server]]
- [[#Установка Wireshark|Установка Wireshark]]
- [[#Проанализируем пакеты с помощью Wireshark|Проанализируем пакеты с помощью Wireshark]]
	- [[#Проанализируем пакеты с помощью Wireshark#Проанализируем STUN|Проанализируем STUN]]
		- [[#Проанализируем STUN#Binding Request|Binding Request]]
			- [[#Binding Request#Message Type|Message Type]]
			- [[#Binding Request#Message Length|Message Length]]
			- [[#Binding Request#Message Cookie|Message Cookie]]
			- [[#Binding Request#Transaction ID|Transaction ID]]
		- [[#Проанализируем STUN#Binding Success Response|Binding Success Response]]
			- [[#Binding Success Response#Attributes|Attributes]]

### Overview
В виртуальной среде мы будем настраивать следующий стенд:
![Pasted image 20261101183745](images/Pasted%20image%2020261101183745.png)

### Private Network A:

#### MicroTik 1
MicroTik 1 является роутером для внутренней подсети, так что у него должно быть 2 интерфейса:
1) первый для локальной сети
2) второй для выхода в интернет
Для маршрутизации трафика настроем NAT

##### MicroTik 1
С помощью команды узнаем какие интерфейсы есть у нас MicroTik:

```python
interface/print
```

![Pasted image 20261101150146](images/Pasted%20image%2020261101150146.png)

`ether1` - наша локальная сеть. В настройках MicroTik данный интерфейс имеет локальный сегмент **host1**.
![Pasted image 20261101150804](images/Pasted%20image%2020261101150804.png)

`ether2` - публичная сеть. В настройках MicroTik данный интерфейс имеет локальный сегмент **network**.
![Pasted image 20261101150931](images/Pasted%20image%2020261101150931.png)

Добавляем на интерфейс для локальной сети пул адресов и указываем шлюз `192.168.1.1`. 

```python
ip/address/add address=192.168.1.1/24 interface=ether1
ip/address/print
```

![Pasted image 20261101151335](images/Pasted%20image%2020261101151335.png)

#### Host 1 (Linux)
В настройках виртуальной машины **Network Adapter** установлен в **LAN Segment** и выбран сегмент `host1`
![Pasted image 20261101153639](images/Pasted%20image%2020261101153639.png)

Открываем настройки подключения к сети:
![Pasted image 20261101152935](images/Pasted%20image%2020261101152935.png)

Переходим к настройкам проводного подключения:
![Pasted image 20261101152950](images/Pasted%20image%2020261101152950.png)

Открываем настройки IPv4:
![Pasted image 20261101153003](images/Pasted%20image%2020261101153003.png)

В метод IPv4 указываем "вручную". 
**Адрес:** `192.168.1.10`
**Маска сети:** `255.255.255.0`
**Шлюз:** `192.168.1.1`

Применяем изменения:
![Pasted image 20261101153209](images/Pasted%20image%2020261101153209.png)

После этого перезапускаем службу `NetworkManager`:
![Pasted image 20261101153709](images/Pasted%20image%2020261101153709.png)

Проверяем, что соединение с MicroTik 1 существует с помощью команды `ping`:
![Pasted image 20261101153812](images/Pasted%20image%2020261101153812.png)

![Pasted image 20261101162202](images/Pasted%20image%2020261101162202.png)
В моем случае ping проходит, значит все настроено верно. 


### Private Network B:

#### MicroTik 2
MicroTik 2 является роутером для внутренней подсети, так что у него должно быть 2 интерфейса:
1) первый для локальной сети
2) интерфейс для выхода в интернет
Для маршрутизации трафика настроем NAT

Как и в случае с MicroTik 1, у нас имеется 2 интерфейса: 
`ether1` - локальная сеть (host2).  
`ether2` - публичная сеть (network). 

```python
interface/print
```

![Pasted image 20261101154342](images/Pasted%20image%2020261101154342.png)

Добавляем пул адресов на интерфейс `ether1`:

```python
ip/address/add address=192.168.1.1/24 interface=ether1
ip/address/print
```

![Pasted image 20261101154609](images/Pasted%20image%2020261101154609.png)

#### Host 2 (Linux)
Убеждаемся, что в настройках виртуальной машины **Host 2** правильно настроен интерфейс: 
![Pasted image 20261101155047](images/Pasted%20image%2020261101155047.png)

Так как на **Host 1** мы настроили статический адрес с помощью графической оболочки, на **Host 2** сделаем это с помощью терминала:

С помощью команды `ip -c a` узнаем интерфейсы:
![Pasted image 20261101155452](images/Pasted%20image%2020261101155452.png)
Нас интересует интерфей1с **ens33**. Сейчас на нем нет ip.

Открываем файл `00-installer-config.yaml`:
```bash
sudo nano /etc/netplan/00-installer-config.yaml 
```

Так как мы используем статический IPv4, то отключаем `dhcp` на IPv4 и IPv6.
Добавляем адрес `192.168.1.10` и указываем маску подсети с помощью `/24`.
Добавляем информацию о шлюзе с помощью `routes`. Добавляем информацию о том, какому демону передать управление сетью, в нашем случае **NetworkManager**.

В конце должен получиться следующее содержание файла
![Pasted image 20261101161148](images/Pasted%20image%2020261101161148.png)

```python
network:
  ethernets:
    ens33:
      dhcp4: false
      dhcp6: false
      match:
        macaddress: 00:0c:29:a4:ff:53
      set-name: ens33
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.1
  version: 2
```
*!!! Важно, чтобы `macaddress` совпадал с macaddress сетевой карты (команда `ip -c a`)* 

Применяем изменения с помощью команды:
```bash
sudo netplan apply
```
![Pasted image 20261101161206](images/Pasted%20image%2020261101161206.png)

Проверяем, что ip адрес появился: 
![Pasted image 20261101161828](images/Pasted%20image%2020261101161828.png)

Проверяем, что microtik и linux имеют соединение:

Linux:
![Pasted image 20261101161945](images/Pasted%20image%2020261101161945.png)

MicroTik:
![Pasted image 20261101162040](images/Pasted%20image%2020261101162040.png)


#### MicroTik 1 and MicroTik 2 connection

##### MicroTik 1
Добавляем ip адреса на интерфейс `ether2`:

```python
ip/address/add address=203.0.113.1/24 interface=ether2
ip/address/print
```

![Pasted image 20261101170155](images/Pasted%20image%2020261101170155.png)

Теперь настроим NAT:

```python
/ip/firewall/nat/add action=masquerade chain=srcnat out-interface=ether2
```

![Pasted image 20261101171534](images/Pasted%20image%2020261101171534.png)

- `/ip/firewall/nat/add`: добавить новое правило NAT.
- `action=masquerade`: заменить исходный IP-адрес пакета на IP-адрес выходного интерфейса. Если необходимо, также изменить порт.
- `chain=srcnat`: применять правило к исходящему трафику, то есть менять адрес отправителя.
- `out-interface=ether2`: применять правило только к пакетам, которые выходят через интерфейс ether2.

##### MicroTik 2

Добавляем ip адреса на интерфейс `ether2`:

```python
ip/address/add address=203.0.113.2/24 interface=ether2
ip/address/print
```

![Pasted image 20261101172018](images/Pasted%20image%2020261101172018.png)

Теперь настроим NAT:

```python
/ip/firewall/nat/add action=masquerade chain=srcnat out-interface=ether2
```

![Pasted image 20261101172059](images/Pasted%20image%2020261101172059.png)

##### Connection between MicroTik 1 and MicroTik 2
Проверяем, что **MicroTik1** и **MicroTik2** соединяются c помощью команды `ping`:

**MicroTik1** ping:
![Pasted image 20261101172653](images/Pasted%20image%2020261101172653.png)

**MicroTik2** ping:
![Pasted image 20261101172725](images/Pasted%20image%2020261101172725.png)

Ping проходит, а значит соединение есть. 

##### Check that NAT is working

Попробуем пропинговать MicroTik2 с Host1 и после MicroTik1 с Host2.

**Host1:**
![Pasted image 20261101173008](images/Pasted%20image%2020261101173008.png)

**Host2:**
![Pasted image 20261101173050](images/Pasted%20image%2020261101173050.png)
**Ping** проходит, значит **NAT** работает. 


### Public Host

В настройках виртуальной машины в **Network Adapter** ставим натсройки **LAN segment** как `network`:
![Pasted image 20261101173559](images/Pasted%20image%2020261101173559.png)

В настройках **Public Host** ставим статический адрес:

![Pasted image 20261101173724](images/Pasted%20image%2020261101173724.png)

Пробуем пропинговать **Microtik1** и **Microtik2**:
![Pasted image 20261101173831](images/Pasted%20image%2020261101173831.png)

Попробуем с **Host1** пропинговать **PublicHost**:
![Pasted image 20261101173949](images/Pasted%20image%2020261101173949.png)

Попробуем с **Host2** пропинговать **PublicHost**:
![Pasted image 20261101174241](images/Pasted%20image%2020261101174241.png)

Попробуем с **PublicHost** пропинговать **Host1**/**Host2**:
![Pasted image 20261101174329](images/Pasted%20image%2020261101174329.png)
*Network Unreachable.*

### Download Coturn on PublicHost
Для скачивания нам понадобиться реальный выход в интернет.
Для этого заходим в настройки виртуальной машины и добавляем еще один сетевой адаптер:
![Pasted image 20261101174645](images/Pasted%20image%2020261101174645.png)

Оставим его в режиме NAT:
![Pasted image 20261101174724](images/Pasted%20image%2020261101174724.png)


Внутри виртуальной машины проверяем, что появился доступ в интернет c помощью пинга:

```bash
ping 8.8.8.8
```

![Pasted image 20261101174810](images/Pasted%20image%2020261101174810.png)

Теперь устанавливаем `coutun`:

```bash
sudo apt update
sudo apt install coturn
```

![Pasted image 20261101175157](images/Pasted%20image%2020261101175157.png)

После успешной установки, можно удалить сетевой адаптер:
![Pasted image 20261101175354](images/Pasted%20image%2020261101175354.png)

### Настроим Coturn
Разрешим запускать `turnserver` автоматически как системную службу при загрузке Ubuntu. Для этого отредактируем файл `/etc/default/coturn` и расскоментируем следующую строку:
![Pasted image 20261101180025](images/Pasted%20image%2020261101180025.png)

### Настроим STUN сервер

На **PublicHost** откроем `/etc/turnserver.conf` и добавим туда следующие настройки:

```python
listening-ip=203.0.113.10
listening-port=3478
fingerpring
```

`listening-ip`: указываем ip, с которого будем принимать запросы.
`listening-port`: указываем порт, на который будут приходить запросы

открываем порты в фаерволе:

```bash
sudo ufw allow 3478/udp
sudo ufw allow 3478/tcp
```

![Pasted image 20261101181339](images/Pasted%20image%2020261101181339.png)

Перезапускаем `coturn` с помощью команды:
```bash
sudo systemctl restart coturn
```

#### Настроить Coturn на Host1

В настройках виртуальной машины добавляем сетевой адаптер с типом NAT:
![Pasted image 20261101182248](images/Pasted%20image%2020261101182248.png)

Установим **coturn**:

```bash
sudo apt update
sudo apt install coturn
```

![Pasted image 20261101182549](images/Pasted%20image%2020261101182549.png)

Удаляем сетевой адаптер:
![Pasted image 20261101182638](images/Pasted%20image%2020261101182638.png)

Выполним запрос к внешнему серверу (STUN):

```bash
turnutils_stunclient -p 3478 203.0.113.10
```

![Pasted image 20261101182912](images/Pasted%20image%2020261101182912.png)

Можно заметить, что **STUN сервер** выдал нам наш публичный ip и port, который присвоил NAT, который настроен на **MicroTik1**.
### Set up a TURN server

Аналогично скачиванию клиента **cuturn** на **host1** в разделе [Настроить Coturn на Host1](#Настроить%20Coturn%20на%20Host1) скачиваем клиент на **host2**.

На **PublicHost** меняем конфигурацию turn сервера: 

```bash
sudo nano /etc/turnserver.conf 
```

```conf
listening-ip=203.0.113.10
listening-port=3478
fingerpring

# Add Relay port range 
min-port=49000
max-port=50000
lt-cred-mech
realm=testturnserver
```

![](Pasted%20image%2020261110172403.png)

Мы добавили пару новых полей: 
**Min-port** и **Max-port** определяют диапазон портов, которые будут выдаваться пользователям. 

**lt-cred-mech** расшифровывается как **Long-Term Credential Mechanism** - механизм долговременных учётных данных. Простыми словами: включает проверку логина и пароля клиента при подключении к TURN серверу.

**realm** - это область аутентификации (Authentication Realm). Простыми словами, название области, к которой относятся учётные данные.

После обновляем правила firewall, чтобы пропускать трафик на нужные порты.

```bash
sudo ufw allow 49000:50000/udp
```

![](Pasted%20image%2020261110162214.png)

Теперь добавим пользователей. 

```bash
sudo turnadmin -a -u host1 -p password123 -r testturnserver
sudo turnadmin -a -u host2 -p password123 -r testturnserver
```

![](Pasted%20image%2020261110172733.png)
![](Pasted%20image%2020261110172928.png)
Перезапускаем `coturn` службу:

``` bash
sudo systemctl restart coturn
```

![](Pasted%20image%2020261110162354.png)

Проверяем, что служба запущена без ошибок и работает:

![](Pasted%20image%2020261110162528.png)

На **Host1** и **Host2** установим `https://github.com/umerghafoor/P2P-STUN-TURN-client.git`:

```
git clone https://github.com/umerghafoor/P2P-STUN-TURN-client.git
```

![](Pasted%20image%2020261110184707.png)

На хостах узнаем публичный ip:
- Host1: `203.0.113.1`
- Host2: `203.0.113.2`

```bash
turnutils_stunclient 203.0.113.10
```

![](Pasted%20image%2020261110185233.png)

В данной приложении мы можем увидеть список аргументов:

![](Pasted%20image%2020261110185838.png)

Мы будем запускать программу следующей командой:

**Host1:**

```bash
python3 ./client.py answer --turn "turn:203.0.113.10:3478" --turn-user host1 --turn-pass password123
```

![](Pasted%20image%2020261110190237.png)

**Host2:**

```bash
python3 ./client.py offer --turn "turn:203.0.113.10:3478" --turn-user host1 --turn-pass password123
```

![](Pasted%20image%2020261110190258.png)

Копируем сгенерированный LOCAL SDP из терминала Host2:

![](Pasted%20image%2020261110190612.png)

И вставляем ее в терминал **Host1**: 

![](Pasted%20image%2020261110190756.png)

После этого **Host1** генерирует свой LOCAL SDP. Его нужно скопировать и вставить в терминал **Host2**:

![](Pasted%20image%2020261110190903.png)

Вставляем в терминал Host2:

ОСТАНОВИЛСЯ ТУТ... НАДО ФИКСИТЬ, СОЕДИНЕНИЯ НЕТ!

### Установка Wireshark
Установим Wireshark для того, чтобы восстановить хронологию соединения и отправки пакетов.  Процесс установки wireshark на все хосты ubuntu един. 

```bash
sudo apt install wireshark
```

Выбираем "Да":
![Pasted image 20261109154746](images/Pasted%20image%2020261109154746.png)

### Проанализируем пакеты с помощью Wireshark
#### Проанализируем STUN
С помощью команды `ip -c a` узнаем название интерфейса. В моем случае это `ens33`:
![Pasted image 20261109155344](images/Pasted%20image%2020261109155344.png)

Запускаем **wireshark** с правами администратора:
![Pasted image 20261109155213](images/Pasted%20image%2020261109155213.png)

Выбираем интерфейс `ens33`:
![Pasted image 20261109155634](images/Pasted%20image%2020261109155634.png)

Начинается захват пакетов:
![Pasted image 20261109155706](images/Pasted%20image%2020261109155706.png)

В новом терминале отправляем запрос к STUN серверу:

```bash
turnutils_stunclient -p 3478 203.0.113.10
```

![Pasted image 20261109155755](images/Pasted%20image%2020261109155755.png)

Возвращаемся в Wireshark и фильтруем пакеты по ip stun сервера:

```wireshark
ip.addr == 203.0.113.10
```

![Pasted image 20261109155956](images/Pasted%20image%2020261109155956.png)

Мы видим два пакета. Давайте проанализируем их ближе:

##### Binding Request 
Первый пакет отправляет клиент. Он называется `Binding Request`.
![Pasted image 20261109160334](images/Pasted%20image%2020261109160334.png)
![Pasted image 20261109161754](images/Pasted%20image%2020261109161754.png)

Пакет STUN состоит следующим образом:
![Pasted image 20261109160917](images/Pasted%20image%2020261109160917.png)
Source: https://www.rfc-editor.org/info/rfc8489/#section-5

Первые два бита `00` говорят о том, что перед нами STUN пакет. 

###### Message Type
Следующие 14 бит - это **STUN Message Type**. Это тип сообщения. Он разделен на класс (2 бита) и метод (12 бит). 
![Pasted image 20261109165443](images/Pasted%20image%2020261109165443.png)
Source: https://www.rfc-editor.org/info/rfc8489/#section-5

**Classes**
Два бита класса определяют направление и суть транзакции (запрос это, ответ или просто уведомление):
	-- **0b00** (Request): Клиент отправляет запрос серверу, ожидая обязательного ответа.
	-- **0b01** (Indication): Сообщение отправляется в одну сторону без ожидания ответа (например, для поддержания NAT-сессии: Keep-Alive).
	-- **0b10** (Success Response): Сервер подтверждает успешную обработку запроса и возвращает данные (например, ваш публичный IP).
	-- **0b11** (Error Response): Сервер сообщает, что что-то пошло не так.

**Methods**
В чистом STUN базовым является всего один метод:
	-- **0x001** (Binding): Запрос на определение внешнего адреса.

Если STUN расширяется до протокола TURN (RFC 8656, используется для ретрансляции трафика через прокси-сервер), добавляются новые методы:
	-- **0x003** (Allocate): Запрос на выделение реле-адреса на сервере.
	-- **0x004** (Refresh): Продление времени жизни выделенного адреса.
	-- **0x006** (Send): Отправка данных через сервер.
	-- **0x007** (Data): Получение данных от сервера.
	-- **0x009** (CreatePermission): Запрос прав на обмен данными с конкретным пиром.

**Message Method Assigment**
Это бит, который говорит о том, что мы используем официальные задокументированные метода или специфичные и сторонние. 
Бит 0 говорит, что мы используем официальный стандарт, а 1 наоборот. 

В нашем случае Message Class: **00**, а значит мы отправили `Request` класс.
Метод у нас `0x001`, то есть `Binding`.
Message Method Assigment у нас равен 0, а значит мы используем официальный стандарт, поэтому wireshark определил, что у нас `Binding Request`

###### Message Length
**Message Length** определяет длину **только дополнительной информации (атрибутов)**, которая идет сразу после основной секции 20-байтового заголовка.
В нашем случае длина равна 0, так как у нас нет секции с дополнительной информацией. 

###### Message Cookie
Фиксированное значение `0x2112A442` в сетевом порядке байтов. Это помогает отличать `STUN` от других протоколов.

###### Transaction ID
96-битный идентификатор, уникально определяющий запрос/ответ. Это помогает определять пары запросов и ответов.
Его генерирует пользователь случайным образом и отправляет запрос на сервер. Сервер, когда отправляет ответ, использует тот же самый transaction id. Когда пользователь получает ответ от сервера, он может сделать точное соответствие между запросом и ответом. 

##### Binding Success Response
В ответе мы можем заметить, что:
**Message Type**:
- Classs: `0x10` -  Success Response
- Method: `0x001` - Binding (он единственный для STUN)
**Message Length**: равен 36 байтам, потому что появилось поле Attributes.
**Message Transaction ID**: равен `136137a7cf6942644904ef50`, что равно Message Transaction ID запроса. То есть сервер отправил тот же Message Transaction ID, что сгенерировал пользователь. 
![Pasted image 20261109171830](images/Pasted%20image%2020261109171830.png)

###### Attributes
В ответе появилось поле Attributes. Давайте рассмотрим его подробнее:
![Pasted image 20261109172744](images/Pasted%20image%2020261109172744.png)
Source: https://www.rfc-editor.org/info/rfc8489/#section-14

На верхнем уровне каждый атрибут имеет фиксированную структуру:
	- **Attribute Type** (16 бит / 2 байта): Идентификатор типа.
	- **Length** (16 бит / 2 байта): Длина только поля Value в байтах.
	- **Value** (переменная длина): Полезная нагрузка.

Внутри 16-битного поля Attribute Type заложены важные правила для программы:
	- **Бит 0** (Comprehension): Определяет, что делать, если тип неизвестен. 
		- 0 означает "Required" -  обязательный атрибут к прочтению. Если его невозможно прочитать, пакет считается сломанным. 
		- 1 означает "Optional" - дополнительный атрибут можно проигнорировать, поэтому если его не получается прочитать, то его можно просто пропустить.
	- **Бит 1** (Assignment): 
		- 0 означает "IETF Review" (Официальный стандарт из RFC). 
		- 1 означает "Vendor-specific" (Кастомный атрибут от конкретного производителя).
	- **Биты 2-15** (14 бит): Порядковый номер самого атрибута.

2. Разбор реального дампа из Wireshark

В нашем случае сервер отправил ответ с двумя атрибутами: 
1) Атрибут XOR-MAPPED-ADDRESS
2) Атрибут SOFTWARE

**Атрибут XOR-MAPPED-ADDRESS**:
В дампе пришел тип атрибута `0x0020`. В двоичном виде это `0000 0000 0010 0000`.
- **Самый первый бит равен 0**, это значит, что данный атрибут **обязателен к прочтению**. 
- **Второй бит равен 0**, который означает, что атрибут является официальным из стандарта RFC.
- **Биты 2-15** хранят числовой код `0x0020`. Для программы это четкий сигнал: «Внутри лежат замаскированные порт и IP-адрес».  
- **Length** (Длина): Равна 8. Это значит, что полезные данные в поле Value занимают ровно 8 байт.  
- **Value** (Значение): Содержит информацию о версии протокола IPv4 (`0x01`), а также скрытые с помощью маски XOR порт и IP-адрес.  
- **Расшифровка порта**: В пакете передаются байты `e00b`. Чтобы узнать реальный порт, программа делает операцию XOR со старшими байтами Magic Cookie (`0x2112`). Результат `0xe00b` XOR `0x2112` дает реальный порт **49433**.  
- **Расшифровка IP**: В пакете передаются байты `ea12d543`. Программа делает операцию XOR со всем Magic Cookie (`0x2112A442`). Результат `0xea12d543` XOR `0x2112A442` дает чистый IP-адрес **203.0.113.1**.

**Атрибут SOFTWARE:**
В дампе пришел тип атрибута `0x8022`. В двоичном виде это` 1000 0000 0010 0010`.
- **Самый первый бит равен 1**, это значит, что данный атрибут **является необязательным**. Если бы наше приложение было старым и не знало этот атрибут, оно бы просто пропустило его и продолжило работу.
- **Второй бит равен 0**, который означает, что атрибут является **официальным из стандарта RFC**.
- **Биты 2-15 хранят** числовой код `0x0022`. По этому коду программа понимает, что внутри лежит текстовое название программного обеспечения.
- **Length 20**: Это означает, что полезные данные в поле Value занимают ровно 20 байт.
- **Value**: Содержит чистый текст "Coturn-4.6.1 'Gorst'". Так как длина текста составляет ровно 20 байт и без остатка делится на 4, серверу не пришлось добавлять в конец пустые байты для выравнивания.

