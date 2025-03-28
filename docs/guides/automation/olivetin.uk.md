---
title: OliveTin
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6
tags:
  - автоматизація
  - web
  - bash
---

# Як встановити та використовувати OliveTin на Rocky Linux

## Вступ

Ви коли-небудь втомлювалися вводити ті самі команди CLI знову і знову? Ви коли-небудь хотіли, щоб усі інші у вашому домі могли перезапустити сервер Plex без вашого втручання? Хочете просто ввести ім’я на веб-панелі, натиснути кнопку й спостерігати, як чарівним чином з’являється спеціальний контейнер Docker/LXD?

Тоді ви можете перевірити OliveTin. OliveTin — це просто програма, яка дозволяє створити веб-сторінку з файлу конфігурації, і ця веб-сторінка має кнопки. Натискайте кнопки, і OliveTin запустить попередньо встановлені команди bash, які ви налаштували самостійно.

Звичайно, технічно ви можете створити щось подібне самостійно, з нуля, маючи достатній досвід програмування... але цей *шлях* простіше. Після налаштування це виглядає приблизно так (зображення надано [репозиторієм OliveTin](https://github.com/OliveTin/OliveTin)):

![Скріншот OliveTin на робочому столі; він містить кілька квадратів у сітці з мітками та діями для кожної команди, яку можна виконати.](olivetin/screenshotDesktop.png)

!!! Warning "НІКОЛИ не запускайте цю програму на публічному сервері"

    Ця програма, за дизайном і за власним визнанням творця, призначена для використання в локальних мережах, *можливо* в налаштуваннях розробників. Однак наразі він не має системи автентифікації користувачів і (поки розробник не виправить це) *за замовчуванням* працює від імені root.
    
    Тому використовуйте це все у захищеній мережі з брандмауером. *Не* розміщуйте його на будь-якому місці, призначеному для загального використання. Поки що.

## Передумови та припущення

Щоб слідувати цьому посібнику, вам знадобиться:

* Комп’ютер під керуванням Rocky Linux
* Мінімальний комфорт або досвід роботи з командним рядком.
* Кореневий доступ або можливість використовувати `sudo`.
* Вивчити основи YAML. Це не важко; ви дізнаєтеся про це нижче.

## Встановлення OliveTin

OliveTin містить попередньо зібрані RPM. Просто завантажте тут найновіший випуск для вашої архітектури та встановіть його. Якщо ви виконуєте цей посібник на робочій станції з графічним робочим столом, просто завантажте файл і двічі клацніть його у вибраному файловому менеджері.

Якщо ви встановлюєте цю програму на сервері, ви можете завантажити її на свою робочу машину та завантажити через SSH/SCP/SFTP або зробити те, що деякі люди рекомендують не робити, і завантажити її за допомогою `wget`.

Наприклад:

```bash
wget https://github.com/OliveTin/OliveTin/releases/download/2022-04-07/OliveTin_2022-04-07_linux_amd64.rpm
```

Потім інсталюйте програму за допомогою (знову, наприклад):

```bash
sudo rpm -i OliveTin_2022-04-07_linux_amd64.rpm
```

OliveTin може працювати як звичайна служба `systemd`, але поки не вмикайте її. Спочатку вам потрібно налаштувати файл конфігурації.

!!! Note "Примітка"

    Після деякого тестування я визначив, що ці самі інструкції зі встановлення добре працюватимуть у контейнері Rocky Linux LXD. Для всіх, хто любить Docker, доступні готові зображення.

## Налаштування дій OliveTin

OliveTin може робити все, що може робити bash, і багато іншого. Ви можете використовувати його для запуску програм із параметрами CLI, запуску сценаріїв bash, перезапуску служб тощо. Щоб почати, відкрийте файл конфігурації за допомогою текстового редактора за вашим вибором за допомогою root/sudo:

```bash
sudo nano /etc/OliveTin/config.yaml
```

Найпростіший вид дії — це кнопка; ви натискаєте на неї, і команда запускається на головному комп’ютері. Ви можете визначити це у файлі YAML наступним чином:

```yaml
actions:
  - title: Restart Nginx
    shell: systemctl restart nginx
```

Ви також можете додавати власні значки до кожної дії, як у випадку з емодзі Unicode:

```yaml
actions:
  - title: Restart Nginx
    icon: "&#1F504"
    shell: systemctl restart nginx
```

Я не збираюся вдаватися в усі подробиці параметрів налаштування, але ви також можете використовувати введення тексту та спадні меню, щоб додати змінні та параметри до команд, які ви хочете запустити. Якщо ви це зробите, OliveTin запропонує вам ввести перед виконанням команди.

Роблячи це, ви можете запускати будь-яку програму, керувати віддаленими машинами за допомогою SSH, запускати веб-хуки тощо. Перегляньте  [the official documentation](https://docs.olivetin.app/action_examples/intro.html), щоб отримати більше ідей.

Але ось мій власний приклад: у мене є особистий сценарій, який я використовую для створення LXD-контейнерів із попередньо встановленими на них веб-серверами. За допомогою OliveTin я зміг швидко створити графічний інтерфейс для зазначеного сценарію наступним чином:

```yaml
actions:
- title: Build Container
  shell: sh /home/ezequiel/server-scripts/rocky-host/buildcontainer -c {{ containerName }} -d {{ domainName }} {{ softwarePackage }}
  timeout: 60
  arguments:
    - name: containerName
      title: Container Name
      type: ascii_identifier

    - name: domainName
      title: Domain
      type: ascii_identifier

    - name: softwarePackage
      title: Default Software
      choices:
        - title: None
          value:

        - title: Nginx
          value: -s nginx

        - title: Nginx & PHP
          value: -s nginx-php

        - title: mariadb
          value: -s mariadb
```

На передній частині це виглядає так (і так, OliveTin має темний режим, і мені *справді* потрібно змінити цей значок):

![Форма з трьома способами введення тексту та випадаючим меню](olivetin/containeraction.png)

## Увімкнення OliveTin

Після того, як ваш конфігураційний файл створено так, як ви хочете, просто ввімкніть і запустіть OliveTin за допомогою:

```bash
sudo systemctl enable --now OliveTin
```

Кожного разу, коли ви редагуєте файл конфігурації, вам потрібно буде перезапустити службу звичайним способом:

```bash
sudo systemctl restart OliveTin
```

## Висновок

OliveTin — це чудовий спосіб запускати все, від команд bash до деяких досить складних операцій зі сценаріями. Пам’ятайте, що за замовчуванням усе працює як root, якщо ви не використовуєте su/sudo у своїх командах оболонки, щоб змінити користувача для цієї конкретної команди.

Таким чином, ви повинні бути обережними, як ви налаштовуєте все це, особливо якщо ви плануєте надати доступ (наприклад) своїй родині, контролювати домашні сервери та пристрої тощо.

І знову ж таки, не розміщуйте це на публічному сервері, якщо ви не готові самостійно спробувати захистити сторінку.

В іншому випадку веселіться з цим. Це акуратний маленький інструмент.
