# Лабораторна робота №13: підказки

**Тема:** Конфігурування списків доступу маршрутизаторів (ACL)

---

## 📋 Топологія та обладнання

| Пристрій   | Тип у Packet Tracer | Кількість |
| ---------- | ------------------- | --------- |
| R1, R2, R3 | Router 1841         | 3         |
| PC1, PC2   | PC-PT               | 2         |
| Web-server | Server-PT           | 1         |

> ⚠️ Роутер **1841 не має serial-портів** за замовчуванням. Додайте модуль `WIC-2T`: клацнути по роутеру → вкладка **Physical** → вимкнути живлення → перетягнути модуль у вільний слот → увімкнути живлення.

---

## 🔌 Типи кабелів

| З'єднання                   | Тип кабелю              |
| --------------------------- | ----------------------- |
| PC1 ↔ R1 (Fa0/0)            | Copper Straight-Through |
| PC2 ↔ R2 (Fa0/0)            | Copper Straight-Through |
| Web-server ↔ R3 (Fa0/0)     | Copper Straight-Through |
| R1 (Fa0/1) ↔ R2 (Fa0/1)     | Copper Cross-Over       |
| R1 (Se0/0/0) ↔ R3 (Se0/0/0) | Serial DCE              |
| R3 (Se0/0/1) ↔ R2 (Se0/0/1) | Serial DCE              |

### Serial DCE vs DTE

У палітрі є два схожі serial-кабелі. Для лабораторної потрібен **Serial DCE** (з іконкою годинника ⏱️). Сторона DCE визначається тим кінцем, з якого ви почали з'єднання, тому **починайте з'єднання з R3**, щоб обидва DCE-кінці опинилися на R3.

> 💡 DCE-сторона задає тактову частоту (`clock rate`). Без неї serial-інтерфейс залишиться у стані `down`.

---

## 💾 Два типи конфігурації Cisco IOS

| Конфіг             | Памʼять | При перезавантаженні |
| ------------------ | ------- | -------------------- |
| **running-config** | RAM     | ❌ стирається        |
| **startup-config** | NVRAM   | ✅ зберігається      |

Команди застосовуються миттєво у `running-config`, але зникнуть після перезавантаження. Щоб зберегти їх назавжди:

```
Router#copy running-config startup-config
```

> ⚠️ Файл проєкту `.pkt` зберігає **тільки `startup-config`**. Без виконання цієї команди налаштування зникнуть після повторного відкриття проєкту.

Скорочені варіанти: `write memory` або `wr`.

---

## 🔌 Команда `no shutdown`

На маршрутизаторах Cisco усі інтерфейси за замовчуванням вимкнені (`administratively down`). Команда `no shutdown` активує інтерфейс.

**Перевірка через `show ip interface brief`:**

| Status                  | Protocol | Що це означає                           |
| ----------------------- | -------- | --------------------------------------- |
| `administratively down` | `down`   | Забули `no shutdown`                    |
| `down`                  | `down`   | Немає фізичного зʼєднання               |
| `up`                    | `down`   | На serial відсутній `clock rate` на DCE |
| `up`                    | `up`     | ✅ Усе працює                           |

---

## ⚙️ Повне налаштування маршрутизаторів

> ⚠️ На стороні DCE для serial-інтерфейсів обовʼязково задається `clock rate 64000`. Обидві DCE-сторони знаходяться на **R3**.

### 🔴 R1: повний набір команд

```
Router>enable
Router#configure terminal
Router(config)#hostname R1

R1(config)#interface fastEthernet 0/0
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#interface fastEthernet 0/1
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#interface serial 0/0/0
R1(config-if)#ip address 10.0.0.9 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#exit
R1#copy running-config startup-config
```

### 🔵 R2: повний набір команд

```
Router>enable
Router#configure terminal
Router(config)#hostname R2

R2(config)#interface fastEthernet 0/0
R2(config-if)#ip address 192.168.2.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#interface fastEthernet 0/1
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#interface serial 0/0/1
R2(config-if)#ip address 10.0.0.6 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#exit
R2#copy running-config startup-config
```

### 🟢 R3: повний набір команд

```
Router>enable
Router#configure terminal
Router(config)#hostname R3

R3(config)#interface fastEthernet 0/0
R3(config-if)#ip address 192.168.3.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#interface serial 0/0/0
R3(config-if)#ip address 10.0.0.10 255.255.255.252
R3(config-if)#clock rate 64000
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#interface serial 0/0/1
R3(config-if)#ip address 10.0.0.5 255.255.255.252
R3(config-if)#clock rate 64000
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#exit
R3#copy running-config startup-config
```

---

## 🖥️ Налаштування кінцевих пристроїв

> ⚠️ **Без цього кроку нічого не запрацює.** Особливо важливий **Default Gateway**: без нього ПК не знатиме, куди слати пакети у чужі мережі.

Налаштування виконується через **Desktop → IP Configuration → Static** на кожному пристрої.

| Пристрій       | IP Address   | Subnet Mask   | Default Gateway |
| -------------- | ------------ | ------------- | --------------- |
| **PC1**        | 192.168.1.10 | 255.255.255.0 | 192.168.1.1     |
| **PC2**        | 192.168.2.10 | 255.255.255.0 | 192.168.2.1     |
| **Web-server** | 192.168.3.10 | 255.255.255.0 | 192.168.3.1     |

На **Web-server** додатково: **Services → HTTP** має бути **On** (зазвичай увімкнено за замовчуванням).

### ✅ Перевірка

На кожному ПК у `Desktop → Command Prompt`:

```
PC> ping <свій default gateway>
```

Має проходити успішно. Пінги між мережами на цьому етапі **ще не працюватимуть**, це нормально.

---

## 🌐 Статична маршрутизація

Кожен роутер знає лише про **свої** підключені мережі. Треба навчити його, як дістатися до **двох інших** LAN-мереж.

**Синтаксис:**

```
ip route <мережа> <маска> <next-hop>
```

### 🔴 Маршрути на R1

```
R1#configure terminal
R1(config)#ip route 192.168.2.0 255.255.255.0 10.0.0.2
R1(config)#ip route 192.168.3.0 255.255.255.0 10.0.0.10
R1(config)#exit
R1#copy running-config startup-config
```

### 🔵 Маршрути на R2

```
R2#configure terminal
R2(config)#ip route 192.168.1.0 255.255.255.0 10.0.0.1
R2(config)#ip route 192.168.3.0 255.255.255.0 10.0.0.5
R2(config)#exit
R2#copy running-config startup-config
```

### 🟢 Маршрути на R3

```
R3#configure terminal
R3(config)#ip route 192.168.1.0 255.255.255.0 10.0.0.9
R3(config)#ip route 192.168.2.0 255.255.255.0 10.0.0.6
R3(config)#exit
R3#copy running-config startup-config
```

### ✅ Перевірка

```
R1#show ip route
PC1> ping 192.168.2.10
PC1> ping 192.168.3.10
```

У таблиці маршрутизації мають з'явитися записи з літерою `S` (static). Усі ping-запити мають проходити. Тільки після цього переходити до ACL.

---

## 🛡️ Налаштування ACL

### 🔴 ACL на R1 (Завдання 2, стандартний)

Забороняє PC1 (`192.168.1.10`) взаємодіяти з пристроями з інших мереж.

```
R1#configure terminal
R1(config)#access-list 1 deny 192.168.1.10 0.0.0.0
R1(config)#access-list 1 permit any

R1(config)#interface fastEthernet 0/0
R1(config-if)#ip access-group 1 in
R1(config-if)#exit

R1(config)#exit
R1#copy running-config startup-config
```

> ⚠️ В кінці кожного ACL Cisco автоматично додає неявне `deny any`. Без `permit any` була б заблокована вся мережа, а не лише PC1.

#### ✅ Перевірка ACL (Завдання 3)

На **PC1** → `Desktop → Command Prompt`:

```
PC> ping 192.168.2.10
PC> ping 192.168.3.10
```

| Результат                                            | Що означає              |
| ---------------------------------------------------- | ----------------------- |
| `Reply from ...`                                     | ❌ ACL не працює        |
| `Request timed out` / `Destination host unreachable` | ✅ ACL працює правильно |

> 💡 На R1 виконайте `show access-lists`: лічильник `(N matches)` біля `deny` має зростати після кожного пінгу з PC1.

### 🟢 ACL на R3 (Завдання 4, розширений)

Забороняє PC2 (`192.168.2.10`) звертатися до Web-server (`192.168.3.10`) за HTTP.

```
R3#configure terminal
R3(config)#access-list 101 deny tcp 192.168.2.10 0.0.0.0 192.168.3.10 0.0.0.0 eq www
R3(config)#access-list 101 permit ip any any
R3(config)#access-list 101 permit icmp any any

R3(config)#interface serial 0/0/1
R3(config-if)#ip access-group 101 in
R3(config-if)#exit

R3(config)#exit
R3#copy running-config startup-config
```

> 💡 `eq www` = порт 80 (HTTP). `permit ip any any` дозволяє увесь інший трафік.

#### ✅ Перевірка ACL (Завдання 5)

На **PC2** → `Desktop → Command Prompt`:

```
PC> ping 192.168.3.10
```

Пінг **має проходити**, оскільки ICMP дозволено.

На **PC2** → `Desktop → Web Browser`:

1. URL: `http://192.168.3.10` (або `http://www.site.ua`)
2. Натиснути **Go**

| Результат              | Що означає                      |
| ---------------------- | ------------------------------- |
| Сторінка завантажилась | ❌ ACL не працює                |
| `Request Timeout`      | ✅ ACL працює, HTTP заблоковано |

> 💡 У Packet Tracer будь-який доменний URL у вбудованому браузері резолвиться у локальний HTTP-сервер навіть **без DNS**.

---

## 📝 Загальні нюанси ACL

- **Стандартний ACL** фільтрує лише за IP джерела → ставити **ближче до отримувача**
- **Розширений ACL** фільтрує за джерелом, призначенням, протоколом, портом → ставити **ближче до джерела**
- **Wildcard `0.0.0.0`** = точна адреса (одна машина). Альтернатива: ключове слово `host`:
  ```
  access-list 1 deny host 192.168.1.10
  ```
- **Видалити ACL:**
  ```
  R1(config)#interface fa 0/0
  R1(config-if)#no ip access-group 1 in
  R1(config)#no access-list 1
  ```

---

## 🔍 Корисні команди перевірки

| Команда                   | Призначення                         |
| ------------------------- | ----------------------------------- |
| `show running-config`     | Поточна конфігурація                |
| `show ip interface brief` | Стан усіх інтерфейсів               |
| `show ip route`           | Таблиця маршрутизації               |
| `show access-lists`       | Усі ACL та лічильники співпадінь    |
| `show ip interface fa0/0` | Деталі інтерфейсу та прив'язані ACL |

---

## ❗ Типові помилки

| Помилка                            | Причина                               | Рішення                                |
| ---------------------------------- | ------------------------------------- | -------------------------------------- |
| Інтерфейс `administratively down`  | Не виконано `no shutdown`             | Зайти в інтерфейс → `no shutdown`      |
| Serial-інтерфейс `up/down`         | Немає `clock rate` на DCE             | На R3 → `clock rate 64000`             |
| PC пінгує шлюз, але не іншу мережу | Немає маршрутизації                   | Налаштувати статичні маршрути          |
| ACL заблокував увесь трафік        | Забули `permit any` після `deny`      | Видалити ACL і створити з `permit any` |
| ACL начебто не діє                 | Не прив'язали через `ip access-group` | `show ip interface` для перевірки      |
