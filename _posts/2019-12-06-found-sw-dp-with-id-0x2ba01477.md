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

### Причины на SWD
