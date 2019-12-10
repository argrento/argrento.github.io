---
layout: article
title: Found SW-DP with ID 0x2BA01477
tags: nordic nrf52832 zephyr
modify_date: 2019-12-06
image_path: /assets/images/2019-12-06-found-sw-dp-with-id-0x2ba01477
---

Был обычный вечер четверга, я разбирался с возможностями [RTOS Zephyr](https://www.zephyrproject.org/),
постигая премудрости `west`, и ничто не предвещало беды. Я пытался понять, почему
не работает стандартный пример `basic/blinky`, как вдруг после одного из многочисленных
обновлений прошивки всё сломалось. И сломалось капитально: контроллер больше не прошивался.

---

### blinky

Один из самых простых примеров -- blinky, исходный код которого ниже.

~~~
/*
 * Copyright (c) 2016 Intel Corporation
 *
 * SPDX-License-Identifier: Apache-2.0
 */

#include <zephyr.h>
#include <device.h>
#include <drivers/gpio.h>

#define LED_PORT    DT_ALIAS_LED0_GPIOS_CONTROLLER
#define LED         DT_ALIAS_LED0_GPIOS_PIN

/* 1000 msec = 1 sec */
#define SLEEP_TIME	1000

void main(void)
{
  u32_t cnt = 0;
  struct device *dev;

  dev = device_get_binding(LED_PORT);
  /* Set LED pin as output */
  gpio_pin_configure(dev, LED, GPIO_DIR_OUT);

  while (1) {
    /* Set pin to HIGH/LOW every 1 second */
    gpio_pin_write(dev, LED, cnt % 2);
    cnt++;
    k_sleep(SLEEP_TIME);
  }
}
~~~
{: .language-c}

Код тривиальный: после настройки необходимого пина в бесконечном цикле его значение меняется с требуемой задержкой.
Для сборки необходимо указать требуемое устройство -- я выбрал [nRF52832-mdk](https://wiki.makerdiary.com/nrf52832-mdk/). У меня её нет,
но схема моего самодельного устройства максимально приближена к схеме nRF52832-mdk (если точнее, то не всего устройства, а только обвязки nRF52832).

Затем я запустил сборку командой `west build -b nrf52832_mdk`. Сборка прошла успешно, и в терминале появилась информация об остатке свободной памяти.

~~~
[116/121] Linking C executable zephyr/zephyr_prebuilt.elf
Memory region         Used Size  Region Size  %age Used
           FLASH:       11680 B       512 KB      2.23%
            SRAM:        3908 B        64 KB      5.96%
        IDT_LIST:          56 B         2 KB      2.73%
[121/121] Linking C executable zephyr/zephyr.elf
~~~

Теперь с чистой совестью можно прошить контроллер этой свежесобранной прошивкой.

`west flash --runner jlink --device NRF52832_XXAA`.

Полный лог процесса прошивки находится [здесь](https://gist.github.com/argrento/b599fc6cad7cb4794542ed5ba37fae2d).
Если не считать предупреждения `WARNING: Runner jlink is not configured for use with nrf52832_mdk, this may not work` в самом начале,
то можно подумать, что проблем нет. Как бы не так!

### Cannot connect to target

Devicetree для nRF52832_mdk содержит в себе упоминания от трёх светодиодах, находящихся на пинах `22`, `23` и `24` порта `0`.
Быстро подключив осциллограф к пину `22`, я убедился что на нём логический ноль. Не было даже намёка на хоть какое-то изменение состояния раз в 1 секуду.

![]({{page.image_path}}/00-led-line.png)

Это было странно, потому что в подобном коде ошибиться практически невозможно. Для теста я решил изменить код так, чтобы попробовать получить желаемый
результат на пине 23. Сборка прошла без проблем, а вот залить прошивку не удалось: вместо чтения регистров и проверки содержимого флеш-памяти [лог](https://gist.github.com/argrento/73a8ead6f14e236fd9e318f19f2c1c52) содержал
многократное повторение следующего текста:

~~~
Connecting to target via SWD
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
Scanning AP map to find all available APs
AP[0]: Stopped AP scan as end of AP map has been reached
Iterating through AP map to find AHB-AP to use
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
Scanning AP map to find all available APs
AP[0]: Stopped AP scan as end of AP map has been reached
Iterating through AP map to find AHB-AP to use
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
Scanning AP map to find all available APs
AP[0]: Stopped AP scan as end of AP map has been reached
Iterating through AP map to find AHB-AP to use
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
Scanning AP map to find all available APs
AP[0]: Stopped AP scan as end of AP map has been reached
Iterating through AP map to find AHB-AP to use
Cannot connect to target.
~~~

Двольно неприятно, но ничего страшного, потому что есть официальная утилита `nrfjprog` от Nordic, в которой есть ключ `recover`. Этот ключ стирает содержимое
всей памяти и отключает защиту чтения.

~~~
argz:blinky|master⚡ ⇒  nrfjprog --recover                              
[1]    16688 segmentation fault (core dumped)  nrfjprog --recover
~~~

Погодите, что? Я подумал, что мне показалось и попробовал ещё раз.

~~~
argz:blinky|master⚡ ⇒  nrfjprog --recover                              
[1]    16695 segmentation fault (core dumped)  nrfjprog --recover
~~~

Итак, официальная утилита `nrfjprog` не может восстановить nRF52832. И причём не просто не может, а падает с SEGFAULT при попытке сделать это. Более подробно можно посмотреть в dmesg:

`segfault at fffffffffffffff9 ip 00007fbf86dff8a3 sp 00007ffdddc75830 error 5 in libjlinkarm_unknown_nrfjprogdll.so[7fbf86dda000+194000]`

### Причины этих неприятностей

Сложившаяся ситуация -- это результат моих ошибок в проектировании печатной платы и
выборе платформы для компиляции проекта. Разбор полётов я сделаю с конца списка.

#### segfault

![]({{page.image_path}}/02-sch.png)

Это результат аппаратной проблемы. На картинке вверху часть моей схемы, содержащая цепь сброса.
После того, как я отпаял конденатор C16, `nrfjprog` падать перестал. 

#### Cannot connect to target

Эта проблема появилась в результате неправильного выбра платформы (nrf52832_mdk).

![]({{page.image_path}}/01-reset.png)

На картинке сверху состояние ноги RESET (P0.21), которая подключена к NRST разъёма программирования. Именно из-за этого контроллер не прошивается с помощью `west flash`.

Для лечения используется утилита JLinkExe.

~~~
argz:blinky|master⚡ ⇒  JLinkExe
SEGGER J-Link Commander V6.54c (Compiled Nov  7 2019 17:05:53)
DLL version V6.54c, compiled Nov  7 2019 17:05:41

Connecting to J-Link via USB...O.K.
Firmware: J-Link V9 compiled Jun  2 2222 22:22:22
Hardware version: V9.40
S/N: 59401308
License(s): GDB, RDI, FlashBP, FlashDL, JFlash, RDDI
VTref=3.282V


Type "connect" to establish a target connection, '?' for help
J-Link>connect
Please specify device / core. <Default>: NRF52832_XXAA
Type '?' for selection dialog
Device>NRF52832_XXAA
Please specify target interface:
  J) JTAG (Default)
  S) SWD
  T) cJTAG
TIF>S
Specify target interface speed [kHz]. <Default>: 4000 kHz
Speed>4000
Device "NRF52832_XXAA" selected.


Connecting to target via SWD
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
Scanning AP map to find all available APs
AP[0]: Stopped AP scan as end of AP map has been reached
Iterating through AP map to find AHB-AP to use
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
Scanning AP map to find all available APs
AP[2]: Stopped AP scan as end of AP map has been reached
AP[0]: AHB-AP (IDR: 0x24770011)
AP[1]: JTAG-AP (IDR: 0x02880000)
Iterating through AP map to find AHB-AP to use
AP[0]: Core found
AP[0]: AHB-AP ROM base: 0xE00FF000
CPUID register: 0x410FC241. Implementer code: 0x41 (ARM)
Found Cortex-M4 r0p1, Little endian.
FPUnit: 6 code (BP) slots and 2 literal slots
CoreSight components:
ROMTbl[0] @ E00FF000
ROMTbl[0][0]: E000E000, CID: B105E00D, PID: 000BB00C SCS-M7
ROMTbl[0][1]: E0001000, CID: B105E00D, PID: 003BB002 DWT
ROMTbl[0][2]: E0002000, CID: B105E00D, PID: 002BB003 FPB
ROMTbl[0][3]: E0000000, CID: B105E00D, PID: 003BB001 ITM
ROMTbl[0][4]: E0040000, CID: B105900D, PID: 000BB9A1 TPIU
ROMTbl[0][5]: E0041000, CID: B105900D, PID: 000BB925 ETM

****** Error: Could not find core in Coresight setup
J-Link>erase
Target connection not established yet but required for command.
Device "NRF52832_XXAA" selected.


Connecting to target via SWD
InitTarget() start
InitTarget() end
Found SW-DP with ID 0x2BA01477
AP map detection skipped. Manually configured AP map found.
AP[0]: AHB-AP (IDR: Not set)
AP[0]: Core found
AP[0]: AHB-AP ROM base: 0xE00FF000
CPUID register: 0x410FC241. Implementer code: 0x41 (ARM)
Found Cortex-M4 r0p1, Little endian.
FPUnit: 6 code (BP) slots and 2 literal slots
CoreSight components:
ROMTbl[0] @ E00FF000
ROMTbl[0][0]: E000E000, CID: B105E00D, PID: 000BB00C SCS-M7
ROMTbl[0][1]: E0001000, CID: B105E00D, PID: 003BB002 DWT
ROMTbl[0][2]: E0002000, CID: B105E00D, PID: 002BB003 FPB
ROMTbl[0][3]: E0000000, CID: B105E00D, PID: 003BB001 ITM
ROMTbl[0][4]: E0040000, CID: B105900D, PID: 000BB9A1 TPIU
ROMTbl[0][5]: E0041000, CID: B105900D, PID: 000BB925 ETM
Cortex-M4 identified.
Erasing device...
Comparing flash   [100%] Done.
Erasing flash     [100%] Done.
J-Link: Flash download: Total time needed: 0.026s (Prepare: 0.015s, Compare: 0.000s, Erase: 0.007s, Program: 0.000s, Verify: 0.000s, Restore: 0.003s)
Erasing done.
J-Link>q
~~~

После запуска `JLinkExe` надо ввести команду `connect`, затем указать устройство (чаще всего оно определится автоматически) `NRF52832_XXAA`, интерфейс подключения `SWD`, частота `4000 kHz`. После подключения достаточно ввести всего 1 команду: `erase`. Вся прошивка будет стёрта, работоспособность восстановится.
