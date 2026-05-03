# Лабораторна робота №14: підказки

**Тема:** Використання технологій STP для прокладання резервних каналів зв'язку

---

## 📋 Топологія та обладнання

| Пристрій           | Тип у Packet Tracer | Кількість |
| ------------------ | ------------------- | --------- |
| Sw1, Sw2, Sw3, Sw4 | Switch 2960-24TT    | 4         |
| PC21...PC32        | PC-PT               | 12        |

> 💡 Модель **2960-24TT** має 24 FastEthernet-порти (F0/1...F0/24), яких достатньо для цієї лабораторної.

---

## 🔌 Схема з'єднань

### Між комутаторами (full-mesh, trunk-порти)

Кожен комутатор з'єднується з трьома іншими через порти F0/1, F0/2, F0/3 згідно зі схемою топології.

| З'єднання | Порти               |
| --------- | ------------------- |
| Sw1 ↔ Sw2 | Sw1-F0/2 ↔ Sw2-F0/1 |
| Sw1 ↔ Sw3 | Sw1-F0/3 ↔ Sw3-F0/3 |
| Sw1 ↔ Sw4 | Sw1-F0/1 ↔ Sw4-F0/2 |
| Sw2 ↔ Sw3 | Sw2-F0/2 ↔ Sw3-F0/1 |
| Sw2 ↔ Sw4 | Sw2-F0/3 ↔ Sw4-F0/3 |
| Sw3 ↔ Sw4 | Sw3-F0/2 ↔ Sw4-F0/1 |

> 💡 За цією схемою на кожному комутаторі задіяні всі три порти F0/1, F0/2, F0/3. Якщо ваші номери портів відрізняються, нічого страшного: STP працюватиме коректно за будь-якої схеми, просто Root-порти у виводі `show spanning-tree` будуть інші.

### ПК до комутаторів (access-порти)

| Комутатор | Порти F0/4...F0/6 | Підключені ПК    |
| --------- | ----------------- | ---------------- |
| **Sw1**   | F0/4, F0/5, F0/6  | PC30, PC31, PC32 |
| **Sw2**   | F0/4, F0/5, F0/6  | PC21, PC22, PC23 |
| **Sw3**   | F0/4, F0/5, F0/6  | PC24, PC25, PC26 |
| **Sw4**   | F0/4, F0/5, F0/6  | PC27, PC28, PC29 |

### Типи кабелів

| З'єднання             | Тип кабелю              |
| --------------------- | ----------------------- |
| Комутатор ↔ Комутатор | Copper Cross-Over       |
| ПК ↔ Комутатор        | Copper Straight-Through |

> 💡 У сучасних комутаторах функція Auto-MDIX зазвичай дозволяє використати будь-який тип, але для надійності в Packet Tracer беріть саме cross-over між комутаторами.

---

## 💾 Два типи конфігурації Cisco IOS

| Конфіг             | Памʼять | При перезавантаженні |
| ------------------ | ------- | -------------------- |
| **running-config** | RAM     | ❌ стирається        |
| **startup-config** | NVRAM   | ✅ зберігається      |

Команди застосовуються миттєво у `running-config`. Щоб зберегти їх:

```
Switch#copy running-config startup-config
```

> ⚠️ Файл проєкту `.pkt` зберігає **тільки `startup-config`**. Без цієї команди налаштування зникнуть після повторного відкриття проєкту.

---

## ⚙️ Базова конфігурація комутаторів (Завдання 2)

### 🔴 Sw1: повний набір команд

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Sw1
Sw1(config)#no ip domain lookup
Sw1(config)#enable secret cisco

Sw1(config)#line console 0
Sw1(config-line)#logging synchronous
Sw1(config-line)#password cisco2
Sw1(config-line)#login
Sw1(config-line)#exit

Sw1(config)#line vty 0 15
Sw1(config-line)#logging synchronous
Sw1(config-line)#password cisco3
Sw1(config-line)#login
Sw1(config-line)#end

Sw1#copy running-config startup-config
```

### 🔵 Sw2: повний набір команд

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Sw2
Sw2(config)#no ip domain lookup
Sw2(config)#enable secret cisco

Sw2(config)#line console 0
Sw2(config-line)#logging synchronous
Sw2(config-line)#password cisco2
Sw2(config-line)#login
Sw2(config-line)#exit

Sw2(config)#line vty 0 15
Sw2(config-line)#logging synchronous
Sw2(config-line)#password cisco3
Sw2(config-line)#login
Sw2(config-line)#end

Sw2#copy running-config startup-config
```

### 🟢 Sw3: повний набір команд

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Sw3
Sw3(config)#no ip domain lookup
Sw3(config)#enable secret cisco

Sw3(config)#line console 0
Sw3(config-line)#logging synchronous
Sw3(config-line)#password cisco2
Sw3(config-line)#login
Sw3(config-line)#exit

Sw3(config)#line vty 0 15
Sw3(config-line)#logging synchronous
Sw3(config-line)#password cisco3
Sw3(config-line)#login
Sw3(config-line)#end

Sw3#copy running-config startup-config
```

### 🟡 Sw4: повний набір команд

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname Sw4
Sw4(config)#no ip domain lookup
Sw4(config)#enable secret cisco

Sw4(config)#line console 0
Sw4(config-line)#logging synchronous
Sw4(config-line)#password cisco2
Sw4(config-line)#login
Sw4(config-line)#exit

Sw4(config)#line vty 0 15
Sw4(config-line)#logging synchronous
Sw4(config-line)#password cisco3
Sw4(config-line)#login
Sw4(config-line)#end

Sw4#copy running-config startup-config
```

### 📝 Що означають команди

- `no ip domain lookup`: вимикає спроби резолвити друкарські помилки як DNS-запити (дратівлива затримка при випадковому набиранні неправильної команди)
- `enable secret cisco`: пароль для входу в привілейований режим (`enable`), зашифрований
- `logging synchronous`: повідомлення системи не перебивають введення команди
- `line console 0`: фізичне консольне підключення
- `line vty 0 15`: 16 віртуальних терміналів для Telnet/SSH
- `login`: вимагає пароль при підключенні

---

## 🌐 Конфігурація мережевих параметрів (Завдання 3)

> ⚠️ **Важливий нюанс із базовою методичкою:** Завдання 3.1 говорить «вимкніть всі порти», а потім 3.2-3.3 вмикає порти F0/1-F0/6. Команди нижче відповідають цій логіці. Решта портів (F0/7-F0/24) залишаться у стані `shutdown`, що нормально.

### 🔴 Sw1: налаштування портів і VLAN1

```
Sw1>enable
Sw1#configure terminal

Sw1(config)#interface range FastEthernet 0/1-24
Sw1(config-if-range)#shutdown
Sw1(config-if-range)#exit

Sw1(config)#interface range FastEthernet 0/1-3
Sw1(config-if-range)#switchport mode trunk
Sw1(config-if-range)#no shutdown
Sw1(config-if-range)#exit

Sw1(config)#interface range FastEthernet 0/4-6
Sw1(config-if-range)#switchport mode access
Sw1(config-if-range)#no shutdown
Sw1(config-if-range)#exit

Sw1(config)#interface VLAN 1
Sw1(config-if)#ip address 192.168.10.1 255.255.255.0
Sw1(config-if)#no shutdown
Sw1(config-if)#end

Sw1#copy running-config startup-config
```

### 🔵 Sw2: налаштування портів і VLAN1

```
Sw2>enable
Sw2#configure terminal

Sw2(config)#interface range FastEthernet 0/1-24
Sw2(config-if-range)#shutdown
Sw2(config-if-range)#exit

Sw2(config)#interface range FastEthernet 0/1-3
Sw2(config-if-range)#switchport mode trunk
Sw2(config-if-range)#no shutdown
Sw2(config-if-range)#exit

Sw2(config)#interface range FastEthernet 0/4-6
Sw2(config-if-range)#switchport mode access
Sw2(config-if-range)#no shutdown
Sw2(config-if-range)#exit

Sw2(config)#interface VLAN 1
Sw2(config-if)#ip address 192.168.10.2 255.255.255.0
Sw2(config-if)#no shutdown
Sw2(config-if)#end

Sw2#copy running-config startup-config
```

### 🟢 Sw3: налаштування портів і VLAN1

```
Sw3>enable
Sw3#configure terminal

Sw3(config)#interface range FastEthernet 0/1-24
Sw3(config-if-range)#shutdown
Sw3(config-if-range)#exit

Sw3(config)#interface range FastEthernet 0/1-3
Sw3(config-if-range)#switchport mode trunk
Sw3(config-if-range)#no shutdown
Sw3(config-if-range)#exit

Sw3(config)#interface range FastEthernet 0/4-6
Sw3(config-if-range)#switchport mode access
Sw3(config-if-range)#no shutdown
Sw3(config-if-range)#exit

Sw3(config)#interface VLAN 1
Sw3(config-if)#ip address 192.168.10.3 255.255.255.0
Sw3(config-if)#no shutdown
Sw3(config-if)#end

Sw3#copy running-config startup-config
```

### 🟡 Sw4: налаштування портів і VLAN1

```
Sw4>enable
Sw4#configure terminal

Sw4(config)#interface range FastEthernet 0/1-24
Sw4(config-if-range)#shutdown
Sw4(config-if-range)#exit

Sw4(config)#interface range FastEthernet 0/1-3
Sw4(config-if-range)#switchport mode trunk
Sw4(config-if-range)#no shutdown
Sw4(config-if-range)#exit

Sw4(config)#interface range FastEthernet 0/4-6
Sw4(config-if-range)#switchport mode access
Sw4(config-if-range)#no shutdown
Sw4(config-if-range)#exit

Sw4(config)#interface VLAN 1
Sw4(config-if)#ip address 192.168.10.4 255.255.255.0
Sw4(config-if)#no shutdown
Sw4(config-if)#end

Sw4#copy running-config startup-config
```

### 📝 Що означають команди

- `interface range FastEthernet 0/1-24`: групове налаштування діапазону портів за один раз
- `switchport mode trunk`: порт пропускає кадри з кількох VLAN (для з'єднань між комутаторами)
- `switchport mode access`: порт належить одному VLAN (для підключення кінцевих пристроїв)
- `interface VLAN 1`: віртуальний інтерфейс самого комутатора для керування. IP на ньому потрібен для віддаленого доступу та `ping`-діагностики

---

## 🖥️ Налаштування ПК

Налаштування виконується через **Desktop → IP Configuration → Static** на кожному ПК.

| ПК   | IP Address    | Subnet Mask   | Default Gateway |
| ---- | ------------- | ------------- | --------------- |
| PC21 | 192.168.10.21 | 255.255.255.0 | 192.168.10.254  |
| PC22 | 192.168.10.22 | 255.255.255.0 | 192.168.10.254  |
| PC23 | 192.168.10.23 | 255.255.255.0 | 192.168.10.254  |
| PC24 | 192.168.10.24 | 255.255.255.0 | 192.168.10.254  |
| PC25 | 192.168.10.25 | 255.255.255.0 | 192.168.10.254  |
| PC26 | 192.168.10.26 | 255.255.255.0 | 192.168.10.254  |
| PC27 | 192.168.10.27 | 255.255.255.0 | 192.168.10.254  |
| PC28 | 192.168.10.28 | 255.255.255.0 | 192.168.10.254  |
| PC29 | 192.168.10.29 | 255.255.255.0 | 192.168.10.254  |
| PC30 | 192.168.10.30 | 255.255.255.0 | 192.168.10.254  |
| PC31 | 192.168.10.31 | 255.255.255.0 | 192.168.10.254  |
| PC32 | 192.168.10.32 | 255.255.255.0 | 192.168.10.254  |

> 💡 **Чому Default Gateway = 192.168.10.254?** Усі ПК і комутатори знаходяться в одній мережі `192.168.10.0/24`, тому міжмережева маршрутизація тут не потрібна. Шлюз вказується «про запас», на випадок підключення мережі до зовнішнього середовища у майбутньому. Для перевірки роботи STP цей параметр ролі не грає.

### ✅ Перевірка зв'язності (Завдання 3.5)

З будь-якого ПК пропінгуйте інші ПК і комутатори:

```
PC> ping 192.168.10.21
PC> ping 192.168.10.32
PC> ping 192.168.10.1
```

Усі ping-запити мають проходити. Якщо STP працює коректно, мережа не зациклюється навіть за наявності надлишкових з'єднань.

> ⚠️ **Зачекайте 30-50 секунд** після увімкнення trunk-портів перед пінгом. STP проходить стани Listening (15 с) і Learning (15 с) перш ніж порт перейде у Forwarding. До цього моменту порти будуть помаранчеві, а не зелені.

---

## 🌳 Аналіз STP (Завдання 4)

Команда для перегляду стану STP:

```
Sw1#show spanning-tree
```

### 📖 Як читати вивід

**Верхній блок (Root ID)** показує інформацію про **кореневий комутатор:**

```
Root ID    Priority    32769
           Address     0001.9791.82DC
           Cost        19
           Port        1(FastEthernet0/1)
```

- `Priority 32769`: пріоритет кореневого комутатора (32768 + ID VLAN 1)
- `Address`: MAC-адреса кореневого комутатора
- `Cost`: вартість шляху до кореня (для FastEthernet вартість одного хопа = 19)
- `Port`: локальний порт, через який досягається корінь (Root Port)

> 💡 Якщо у виводі видно `This bridge is the root`, це означає, що поточний комутатор сам є кореневим.

**Нижній блок (Bridge ID)** показує параметри **поточного комутатора.**

**Таблиця інтерфейсів:**

```
Interface    Role  Sts  Cost  Prio.Nbr  Type
Fa0/1        Root  FWD  19    128.1     P2p
Fa0/2        Desg  FWD  19    128.2     P2p
Fa0/3        Altn  BLK  19    128.3     P2p
```

### 🎭 Ролі портів STP

| Роль (Role) | Опис                                                |
| ----------- | --------------------------------------------------- |
| **Root**    | Порт із найкоротшим шляхом до кореневого комутатора |
| **Desg**    | Designated, головний порт для свого сегмента мережі |
| **Altn**    | Alternate, резервний порт (заблокований)            |

### 🚦 Стани портів STP

| Стан (Sts) | Опис                                          |
| ---------- | --------------------------------------------- |
| **FWD**    | Forwarding, передає трафік                    |
| **BLK**    | Blocking, заблокований для уникнення петлі    |
| **LRN**    | Learning, вивчає MAC-адреси (тимчасовий стан) |
| **LIS**    | Listening, слухає BPDU (тимчасовий стан)      |

### 🎯 Як визначити кореневий комутатор

Кореневим є той з 4 комутаторів, чий вивід містить `This bridge is the root`. У базовій конфігурації пріоритети у всіх однакові (32769), тож **кореневим стане комутатор із найменшою MAC-адресою**. Заздалегідь передбачити це у Packet Tracer не вийде: MAC-адреси призначаються при додаванні пристрою, і конкретні значення можуть варіюватися.

> 💡 Якщо потрібно явно зробити певний комутатор кореневим, можна вручну зменшити пріоритет: `Sw1(config)#spanning-tree vlan 1 priority 4096`. Чим менший пріоритет, тим вища ймовірність стати коренем.

> 💡 Зверніть увагу: на некореневих комутаторах **завжди один порт буде Root**, і саме той, що веде до кореня найкоротшим шляхом. Інші зайві з'єднання отримають роль `Altn` і стан `BLK`, забезпечуючи відсутність петель.

### 📝 Скласти STP-топологію

За результатами `show spanning-tree` на всіх 4 комутаторах визначте:

1. Який комутатор є **кореневим** (Root Switch)
2. На кожному з некореневих комутаторів: який порт є **Root**
3. На кожному з'єднанні комутатор↔комутатор: який порт є **Designated** (передає трафік), а який буде заблокованим (`Altn` + `BLK`)

Намалюйте схему за прикладом з методички (зі стрілкою ROOT і червоними хрестиками на заблокованих портах).

---

## 🔍 Корисні команди перевірки

| Команда                      | Призначення                             |
| ---------------------------- | --------------------------------------- |
| `show spanning-tree`         | Повний стан STP по всіх VLAN            |
| `show spanning-tree vlan 1`  | Стан STP для конкретного VLAN           |
| `show spanning-tree summary` | Стислий підсумок                        |
| `show interfaces trunk`      | Список trunk-портів                     |
| `show interfaces status`     | Стан і режим (access/trunk) усіх портів |
| `show vlan brief`            | Список VLAN та портів у них             |
| `show running-config`        | Поточна конфігурація                    |

---

## ❗ Типові помилки

| Помилка                                 | Причина                                             | Рішення                                                      |
| --------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| Порти між комутаторами `down/down`      | Неправильний кабель (straight замість cross)        | Замінити на Copper Cross-Over                                |
| Порт у стані `administratively down`    | Не виконано `no shutdown`                           | `no shutdown` на потрібному порту                            |
| Між ПК пінги не проходять               | Не налаштовано `switchport mode access`             | Зайти в інтерфейс → `switchport mode access` + `no shutdown` |
| Усі порти показують `Desg FWD`          | Аналізуєте кореневий комутатор                      | Запустіть `show spanning-tree` на інших комутаторах          |
| Усі комутатори вважають себе кореневими | Trunk-порти ще не активні (загорілися помаранчевим) | Зачекати 30-50 секунд для конвергенції STP                   |
| Налаштування зникли після перезапуску   | Не виконано `copy running-config startup-config`    | Зберігати після кожного значного блоку                       |
