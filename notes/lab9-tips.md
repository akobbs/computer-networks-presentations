# Підказки до лабораторної роботи №9

## Завдання 1. Побудова топології в Packet Tracer

### Крок 1. Додайте обладнання

У нижній панелі Packet Tracer оберіть категорію **Routers** і додайте на робочу область **три маршрутизатори моделі 1841**.  
Назвіть їх **R1**, **R2**, **R3**.

Оберіть категорію **End Devices** → **PC-PT** і додайте **три комп'ютери**: **PC1**, **PC2**, **PC3**.

> ⚠️ Маршрутизатор 1841 має два інтерфейси FastEthernet (Fa0/0 і Fa0/1) та слоти для модулів.  
> Для серійних з'єднань (Se0/0/0, Se0/0/1) потрібно додати модуль **WIC-2T**:  
> клікніть на маршрутизатор → вкладка **Physical** → вимкніть живлення → перетягніть модуль **WIC-2T** у вільний слот → увімкніть живлення.

---

### Крок 2. З'єднайте пристрої кабелями

Використовуйте інструмент **Connections** (блискавка в нижній панелі).

| З'єднання | Тип кабелю                       | Інтерфейси              |
| --------- | -------------------------------- | ----------------------- |
| PC1 ↔ R1  | Copper Straight-Through (прямий) | PC1 Fa0 → R1 Fa0/0      |
| PC2 ↔ R2  | Copper Straight-Through (прямий) | PC2 Fa0 → R2 Fa0/0      |
| PC3 ↔ R3  | Copper Straight-Through (прямий) | PC3 Fa0 → R3 Fa0/0      |
| R1 ↔ R2   | Copper Cross-Over (перехресний)  | R1 Fa0/1 → R2 Fa0/1     |
| R1 ↔ R3   | Serial DCE/DTE                   | R1 Se0/0/0 → R3 Se0/0/0 |
| R2 ↔ R3   | Serial DCE/DTE                   | R2 Se0/0/1 → R3 Se0/0/1 |

> 💡 **Серійний кабель (Serial DCE/DTE):** у Packet Tracer оберіть **Serial DCE** — кінець із позначкою «clock» (DCE) підключайте до маршрутизатора, який задаватиме тактову частоту.  
> За схемою лабораторної роботи **DCE-кінець** підключається до **R3** на обох серійних з'єднаннях.  
> Перевірити можна командою `show controllers Se0/0/0` — у рядку буде вказано `DCE` або `DTE`.

---

## Завдання 1.1. Налаштування IP-адрес

### На маршрутизаторах (R1, R2, R3)

Перейдіть у режим конфігурації та налаштуйте кожен інтерфейс за таблицею адресації.

**Загальна послідовність команд:**

```
Router> enable
Router# configure terminal
Router(config)# hostname R1

R1(config)# interface fa0/0
R1(config-if)# ip address <IP-адреса> <маска>
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface fa0/1
R1(config-if)# ip address <IP-адреса> <маска>
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface se0/0/0
R1(config-if)# ip address <IP-адреса> <маска>
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Конкретні команди для R1:**

```
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface fa0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface se0/0/0
R1(config-if)# ip address 10.0.0.9 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Конкретні команди для R2:**

```
R2(config)# interface fa0/0
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# interface fa0/1
R2(config-if)# ip address 10.0.0.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# interface se0/0/1
R2(config-if)# ip address 10.0.0.6 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
```

**Конкретні команди для R3:**

```
R3(config)# interface fa0/0
R3(config-if)# ip address 192.168.3.1 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface se0/0/0
R3(config-if)# ip address 10.0.0.10 255.255.255.252
R3(config-if)# clock rate 64000
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface se0/0/1
R3(config-if)# ip address 10.0.0.5 255.255.255.252
R3(config-if)# clock rate 64000
R3(config-if)# no shutdown
R3(config-if)# exit
```

> ⚠️ **Команда `clock rate 64000`** вводиться лише на DCE-кінці серійного з'єднання (у нашому випадку — на R3).  
> Вона задає тактову частоту для синхронізації серійного каналу.  
> Якщо ввести її на DTE-кінці — команда не матиме ефекту або викличе помилку.

---

### На комп'ютерах (PC1, PC2, PC3)

Клікніть на комп'ютер → вкладка **Desktop** → **IP Configuration** → оберіть **Static** і введіть дані вручну.

**PC1:**
| Поле | Значення |
|---|---|
| IP Address | 192.168.1.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.1.1 |

**PC2:**
| Поле | Значення |
|---|---|
| IP Address | 192.168.2.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.2.1 |

**PC3:**
| Поле | Значення |
|---|---|
| IP Address | 192.168.3.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.3.1 |

---

### Перевірка після налаштування

Після введення всіх адрес виконайте на кожному маршрутизаторі:

```
R1# show ip interface brief
```

У стовпцях **Status** і **Protocol** мають бути значення `up` / `up` для всіх налаштованих інтерфейсів.  
Якщо бачите `administratively down` — інтерфейс не активовано командою `no shutdown`.
