# Лабораторна робота №6 — Маршрутизація за замовчуванням

### Cisco Packet Tracer — Покроковий гайд

---

## Зміст

1. [Теорія](#теорія)
2. [Топологія мережі](#топологія-мережі)
3. [Завдання 1 — Побудова топології та базове налаштування](#завдання-1--побудова-топології-та-базове-налаштування)
   - [Встановлення модуля WIC-2T](#101-встановлення-серійного-модуля-wic-2t)
4. [Завдання 2–3 — Перевірка налаштування](#завдання-23--перевірка-налаштування)
5. [Завдання 4 — Налаштування RIP](#завдання-4--налаштування-rip)
6. [Завдання 5 — Маршрут за замовчуванням на R3](#завдання-5--маршрут-за-замовчуванням-на-r3)
7. [Завдання 6 — Розповсюдження маршруту через RIP](#завдання-6--розповсюдження-маршруту-через-rip)
8. [Завдання 7 — Фінальна перевірка](#завдання-7--фінальна-перевірка)

---

## Теорія

**Маршрутизація за замовчуванням** використовується у тупикових мережах (Stub Networks) — мережах, які мають лише один вихідний інтерфейс. Якщо маршрутизатор не знає, куди відправити пакет, він використовує маршрут за замовчуванням.

| Команда                                    | Призначення                       |
| ------------------------------------------ | --------------------------------- |
| `ip routing`                               | Увімкнути маршрутизацію           |
| `no ip routing`                            | Вимкнути маршрутизацію            |
| `ip route 0.0.0.0 0.0.0.0 <інтерфейс\|IP>` | Маршрут за замовчуванням          |
| `show ip route`                            | Переглянути таблицю маршрутизації |
| `show ip interface brief`                  | Короткий статус інтерфейсів       |

---

## Топологія мережі

```
[PC1]──192.168.1.0/24──[R1 Fa0/0]
                         [R1 Fa0/1]──10.0.0.0/30──[R2 Fa0/1]
                         [R1 Se0/0/0]                        [R2 Fa0/0]──192.168.2.0/24──[PC2]
                              │ 10.0.0.8/30                  [R2 Se0/0/1]
                              │                                    │ 10.0.0.4/30
                         [R3 Se0/0/0]──────────────────────[R3 Se0/0/1]
                         [R3 Fa0/0]──192.168.3.0/24──[Web-server]
```

> ⚠️ **Важливо:** R3 є DCE на обох серійних з'єднаннях — clock rate задається тільки на R3.

---

## Завдання 1 — Побудова топології та базове налаштування

### 1.0 Додай пристрої в Packet Tracer

- 3× **Router 1841**
- 2× **PC-PT**
- 1× **Server-PT**

### 1.0.1 Встановлення серійного модуля WIC-2T

> ❗ Роутер 1841 **за замовчуванням не має серійних портів** — їх треба додати вручну перед підключенням кабелів. Без цього кроку при спробі підключити Serial DCE кабель з'явиться помилка `The cable cannot be connected to that port.`

**Повтори для кожного роутера R1, R2 і R3:**

1. Двічі клікни на роутер
2. Перейди на вкладку **Physical**
3. ⚠️ **Вимкни роутер** — клікни кнопку живлення (вона стане сірою)
4. Знайди модуль **WIC-2T** у списку зліва
5. Перетягни **WIC-2T** у **SLOT 0** (перший вільний слот зліва)
6. **Увімкни роутер** назад (та сама кнопка живлення)
7. Натисни **OK** і закрий вікно

✅ Після цього при підключенні Serial DCE кабелю з'являться порти `Se0/0/0` та `Se0/0/1`.

> 💡 **Чому саме SLOT 0?** Якщо встановити модуль у SLOT 1, порти називатимуться `Serial0/1/0` і `Serial0/1/1` — що не відповідає командам у лабораторній роботі. Модуль у SLOT 0 дає правильні назви `Serial0/0/0` і `Serial0/0/1`.

> 💡 **WIC-2T vs WIC-1T** — обов'язково вибирай **WIC-2T** (2 порти). WIC-1T має лише один порт, що недостатньо для R3, якому потрібні два серійних з'єднання.

---

### 1.0.2 Таблиця з'єднань

| Від        | Інтерфейс    | До  | Інтерфейс | Тип кабелю       |
| ---------- | ------------ | --- | --------- | ---------------- |
| PC1        | FastEthernet | R1  | Fa0/0     | Straight-through |
| PC2        | FastEthernet | R2  | Fa0/0     | Straight-through |
| Web-server | FastEthernet | R3  | Fa0/0     | Straight-through |
| R1         | Fa0/1        | R2  | Fa0/1     | Cross-over       |
| R1         | Se0/0/0      | R3  | Se0/0/0   | Serial           |
| R2         | Se0/0/1      | R3  | Se0/0/1   | Serial           |

### 1.1 Таблиця адресації

| Пристрій   | Інтерфейс | IP-адреса    | Маска           | Шлюз        |
| ---------- | --------- | ------------ | --------------- | ----------- |
| R1         | Fa0/0     | 192.168.1.1  | 255.255.255.0   | N/A         |
| R1         | Fa0/1     | 10.0.0.1     | 255.255.255.252 | N/A         |
| R1         | Se0/0/0   | 10.0.0.9     | 255.255.255.252 | N/A         |
| R2         | Fa0/0     | 192.168.2.1  | 255.255.255.0   | N/A         |
| R2         | Fa0/1     | 10.0.0.2     | 255.255.255.252 | N/A         |
| R2         | Se0/0/1   | 10.0.0.6     | 255.255.255.252 | N/A         |
| R3         | Fa0/0     | 192.168.3.1  | 255.255.255.0   | N/A         |
| R3         | Se0/0/0   | 10.0.0.10    | 255.255.255.252 | N/A         |
| R3         | Se0/0/1   | 10.0.0.5     | 255.255.255.252 | N/A         |
| PC1        | NIC       | 192.168.1.10 | 255.255.255.0   | 192.168.1.1 |
| PC2        | NIC       | 192.168.2.10 | 255.255.255.0   | 192.168.2.1 |
| Web-server | NIC       | 192.168.3.10 | 255.255.255.0   | 192.168.3.1 |

---

### 1.2 Налаштування маршрутизатора R1

```cisco
Router>enable
Router#configure terminal
Router(config)#hostname R1

R1(config)#interface FastEthernet0/0
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#interface FastEthernet0/1
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#interface Serial0/0/0
R1(config-if)#ip address 10.0.0.9 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit
```

---

### 1.3 Налаштування маршрутизатора R2

```cisco
Router>enable
Router#configure terminal
Router(config)#hostname R2

R2(config)#interface FastEthernet0/0
R2(config-if)#ip address 192.168.2.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#interface FastEthernet0/1
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#interface Serial0/0/1
R2(config-if)#ip address 10.0.0.6 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit
```

---

### 1.4 Налаштування маршрутизатора R3

> ⚠️ R3 — DCE на обох серійних портах, тому додаємо `clock rate 64000`

```cisco
Router>enable
Router#configure terminal
Router(config)#hostname R3

R3(config)#interface FastEthernet0/0
R3(config-if)#ip address 192.168.3.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#interface Serial0/0/0
R3(config-if)#ip address 10.0.0.10 255.255.255.252
R3(config-if)#clock rate 64000
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#interface Serial0/0/1
R3(config-if)#ip address 10.0.0.5 255.255.255.252
R3(config-if)#clock rate 64000
R3(config-if)#no shutdown
R3(config-if)#exit
```

---

### 1.5 Налаштування PC та Web-server (графічний інтерфейс)

> Для PC та Web-server команди CLI не потрібні — налаштування через GUI Packet Tracer.

**Кроки:**

1. Двічі клікни на пристрій (PC1, PC2 або Web-server)
2. Перейди на вкладку **Desktop**
3. Клікни **IP Configuration**
4. Вибери **Static** і заповни поля відповідно до таблиці адресації

#### PC1

```
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

#### PC2

```
IP Address:      192.168.2.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.2.1
```

#### Web-server

```
IP Address:      192.168.3.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.3.1
```

#### Увімкнення HTTP на Web-server

1. Двічі клікни на **Web-server**
2. Вкладка **Services**
3. Клікни **HTTP** у лівому меню
4. Переконайся що **HTTP: On** та **HTTPS: On**

> ❗ Без цього кроку браузер на PC1/PC2 не зможе завантажити сторінку в Завданні 7.

---

## Завдання 2–3 — Перевірка налаштування

```cisco
! Перевірка конфігурації (на кожному роутері)
R1#show running-config
R2#show running-config
R3#show running-config

! Перевірка стану інтерфейсів — всі мають бути up/up
R1#show ip interface brief
R2#show ip interface brief
R3#show ip interface brief
```

**Очікуваний результат `show ip interface brief`:**

```
Interface         IP-Address      OK? Method Status   Protocol
FastEthernet0/0   192.168.1.1     YES manual up       up
FastEthernet0/1   10.0.0.1        YES manual up       up
Serial0/0/0       10.0.0.9        YES manual up       up
```

> Якщо якийсь інтерфейс показує `down/down` — перевір кабель або повтори `no shutdown`.

---

## Завдання 4 — Налаштування RIP

> ⚠️ Мережа `192.168.3.0/24` (Web-server) **не включається** до RIP на жодному роутері!

### R1

```cisco
R1>enable
R1#configure terminal
R1(config)#router rip
R1(config-router)#version 2
R1(config-router)#network 192.168.1.0
R1(config-router)#network 10.0.0.0
R1(config-router)#no auto-summary
R1(config-router)#exit
```

### R2

```cisco
R2>enable
R2#configure terminal
R2(config)#router rip
R2(config-router)#version 2
R2(config-router)#network 192.168.2.0
R2(config-router)#network 10.0.0.0
R2(config-router)#no auto-summary
R2(config-router)#exit
```

### R3 — мережа 192.168.3.0 НЕ додається!

```cisco
R3>enable
R3#configure terminal
R3(config)#router rip
R3(config-router)#version 2
R3(config-router)#network 10.0.0.0
R3(config-router)#no auto-summary
R3(config-router)#exit
```

### Перевірка таблиць маршрутизації

```cisco
R1#show ip route
R2#show ip route
R3#show ip route
```

**Які мережі мають бути в таблицях після RIP:**

> Нижче вказані мережі, які кожен роутер **отримує від сусідів** через RIP (не власні `C - connected`).

| Маршрутизатор | Отримує через RIP від сусідів      |
| ------------- | ---------------------------------- |
| R1            | `192.168.2.0/24`, `10.0.0.4/30`    |
| R2            | `192.168.1.0/24`, `10.0.0.8/30`    |
| R3            | `192.168.1.0/24`, `192.168.2.0/24` |

> `192.168.3.0/24` НЕ повинна бути в таблицях R1 і R2 — це перевіряється!

---

## Завдання 5 — Маршрут за замовчуванням на R3

```cisco
R3#configure terminal
R3(config)#ip route 0.0.0.0 0.0.0.0 FastEthernet0/0
R3(config)#exit
```

### Перевірка

```cisco
R3#show ip route
```

**Очікуваний результат — в кінці таблиці з'явиться:**

```
Gateway of last resort is 0.0.0.0 to network 0.0.0.0

S*   0.0.0.0/0 is directly connected, FastEthernet0/0
```

> `S*` означає статичний маршрут-кандидат за замовчуванням (static default).

---

## Завдання 6 — Розповсюдження маршруту через RIP

```cisco
R3(config)#router rip
R3(config-router)#default-information originate
R3(config-router)#exit
```

### Перевірка на R1 та R2

```cisco
R1#show ip route
R2#show ip route
```

**Очікуваний результат — в таблицях R1 і R2 з'явиться:**

```
R*   0.0.0.0/0 [120/1] via 10.0.0.10, ...
```

> `R*` означає що маршрут за замовчуванням отриманий через RIP.

---

## Завдання 7 — Фінальна перевірка

### 7.1 Перевірка через браузер

1. Двічі клікни на **PC1** або **PC2**
2. Вкладка **Desktop → Web Browser**
3. Введи в адресний рядок: `192.168.3.10`
4. Натисни **Go**

✅ Якщо сторінка завантажилась — конфігурація правильна.

### 7.2 Додаткова перевірка через Command Prompt (Desktop → Command Prompt)

```bash
# Перевірка IP налаштувань
ipconfig

# Пінг до свого шлюзу
ping 192.168.1.1

# Пінг до Web-server
ping 192.168.3.10
```

### 7.3 Збереження конфігурації

```cisco
R1#copy running-config startup-config
R2#copy running-config startup-config
R3#copy running-config startup-config
```

---

_Лабораторна робота №6 — Організація комп'ютерних мереж_
