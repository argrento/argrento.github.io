---
layout: article
title: STM32F100xx – руководство пользователя
modify_date: 2011-06-23
tags: stm32 manual old
image_path: /assets/images/2011-06-23-stm32f100xx-handbook
permalink: /stm32f100xx-handbook/
hidden: true
---

Статья из моего старого блога: [https://kirik444.wordpress.com/stm32f100xx-handbook/](https://kirik444.wordpress.com/stm32f100xx-handbook/)
{:.info}

**Информация устарела,  завершать перевод я не планирую.**
{:.error}


1. [Соглашения, принятые в документации](/2011/06/24/conventions.html)
   1. [Список сокращений при описании битов регистров](/2011/06/24/conventions#abbreviations)
   2. [Глоссарий](/2011/06/24/conventions#glossary)
   3. [Доступная периферия](/2011/06/24/conventions#periphery)
2. [Модуль подсчета CRC](https://kirik444.wordpress.com/2011/07/01/crc-unit)
   1. [Введение в CRC](https://kirik444.wordpress.com/2011/07/01/crc-unit#introduction)
   2. [Основные свойства CRC](https://kirik444.wordpress.com/2011/07/01/crc-unit#features)
   3. [Функциональное описание CRC](https://kirik444.wordpress.com/2011/07/01/crc-unit#features)
   4. [Регистры CRC](https://kirik444.wordpress.com/2011/07/01/crc-unit)
      1. [Регистр данных (CRC_DR)](https://kirik444.wordpress.com/2011/07/01/crc-unit#CRC_DR)
      2. [Независмый регистр данных (CRC_IDR)](https://kirik444.wordpress.com/2011/07/01/crc-unit#CRC_IDR)
      3. [Регистр управления (CRC_CR)](https://kirik444.wordpress.com/2011/07/01/crc-unit#CRC_CR)
   5. [Программирование модуля CRC](https://kirik444.wordpress.com/2011/07/01/crc-unit#crc_prg)
3. [Управление питанием (PWR)](https://kirik444.wordpress.com/2011/07/04/power/)
    1. [Источники питания](https://kirik444.wordpress.com/2011/07/04/power#pwr_supl)
        1. [Независимые источники питания для АЦП и ЦАП](https://kirik444.wordpress.com/2011/07/04/power#ind_pwr)
        2. [Резервные источники питания](https://kirik444.wordpress.com/2011/07/04/power#pwr_reserved)
        3. [Регулятор напряжения](https://kirik444.wordpress.com/2011/07/04/power#pwr_regulator)
    2. [Контроль источников питания](https://kirik444.wordpress.com/2011/07/04/power#pwr_control)
        1. [Сброс при старте (POR - power on reset)/сброс при отключении (PDR - power down reset)](https://kirik444.wordpress.com/2011/07/04/power#POR_PDR)
        2. [Программируемый детектор напряжения (PVD)](https://kirik444.wordpress.com/2011/07/04/power#PVD)
    3. [Режимы с низким энергопотреблением](https://kirik444.wordpress.com/2011/07/04/power#low_power_modes)
        1. [Уменьшение системных частот](https://kirik444.wordpress.com/2011/07/04/power#slow_clocks)
        2. [Отключение периферийного тактирования](https://kirik444.wordpress.com/2011/07/04/power#gating)
        3. [Спящий режим](https://kirik444.wordpress.com/2011/07/04/power#sleep)
        4. [Режим останова](https://kirik444.wordpress.com/2011/07/04/power#stop)
        5. [Режим ожидания](https://kirik444.wordpress.com/2011/07/04/power#standby)
        6. [Автовыход из режима энергосбережения](https://kirik444.wordpress.com/2011/07/04/power#auto_wakeup)
    4. [Регистры управления питанием](https://kirik444.wordpress.com/2011/07/04/power#registers)
        1. [Регистр управления питанием (PWR_CR)](https://kirik444.wordpress.com/2011/07/04/power#PWR_CR)
        2. [Регистр управления/статусный регистр (PWR_CSR)](https://kirik444.wordpress.com/2011/07/04/power#PWR_CSR)
    5. [Программирование модуля PWR](https://kirik444.wordpress.com/2011/07/04/power#programming)
4. [Регистры для резервного копирования (BKP)](https://kirik444.wordpress.com/2011/09/11/bkp)
    1. [Введение в BKP](https://kirik444.wordpress.com/2011/09/11/bkp#introduction)
    2. [Основные особенности BKP](https://kirik444.wordpress.com/2011/09/11/bkp#features)
    3. [Функциональное описание BKP](https://kirik444.wordpress.com/2011/09/11/bkp#description)
        1. [Детектирование сброса](https://kirik444.wordpress.com/2011/09/11/bkp#tamper)
        2. [Калибровка RTC](https://kirik444.wordpress.com/2011/09/11/bkp#rtc)
    3. [Программирование BKP](https://kirik444.wordpress.com/2011/09/11/bkp#programming)
