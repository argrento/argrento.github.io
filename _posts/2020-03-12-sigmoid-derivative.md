---
layout: article
title: Производная сигмоиды
tags: math neural_networks study
modify_date: 2020-03-12
image_path: /assets/images/2019-12-06-found-sw-dp-with-id-0x2ba01477
mathjax: true
mathjax_autoNumber: true
---

В этой статье мы будем искать производную сигмоиды -- гладкой монотонной возрастающей нелинейной функции. Более подробно можно прочитать здесь -- [https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D0%B3%D0%BC%D0%BE%D0%B8%D0%B4%D0%B0]().

Наиболее часто под сигмоидой понимают логистическую функцию

$$\sigma(x) = \frac{1}{1 + e^{-x}}$$

График которой представлен на рисунке ниже.

<figure>
  <img src="https://upload.wikimedia.org/wikipedia/commons/a/ac/Logistic-curve.png" alt="Рис 1. График сигмоиды"/>
  <figcaption>Рис 1. График сигмоиды. <i>Материал из Wikimedia Commons</i>.</figcaption>
</figure>

---

Итак, нам надо найти производную сигмоиды (1):

$$\sigma'(x) = \frac{d}{dx} \color{red}{\sigma(x)} = \frac{d}{dx} \color{red}{\frac{1}{1 + e^{-x}}}$$

Перепишем (2) с учётом отрицательных степеней:

$$\sigma'(x) = \frac{d}{dx} \color{red}{(1 + e^{-x})^{-1}}$$

Вспомним [теорему о дифференцировании обратной функции](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%BE%D0%B8%D0%B7%D0%B2%D0%BE%D0%B4%D0%BD%D0%B0%D1%8F_%D0%BE%D0%B1%D1%80%D0%B0%D1%82%D0%BD%D0%BE%D0%B9_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B8):

$$ \left[ \frac{1}{u(x)} \right]' =  \left[ u(x)^{-1} \right]' = - \left[ \frac{u'(x)}{u(x)^2} \right] = -u(x)^{-2} \cdot u'(x)$$

И применим эту теорему к (3).

$$\frac{d}{dx} \color{red}{(1 + e^{-x})^{-1}} = \color{red}{-(1 + e^{-x})^{-2} \cdot \frac{d}{dx} (1 + e^{-x})}$$

Учтём то, что дифференцирование -- линейная операция:

$$[a \cdot u(x) + b \cdot v(x)]' = a \cdot u'(x) + b \cdot v'(x)$$

Применяя это свойство к (5) получим следующее:

$$-(1 + e^{-x})^{-2} \cdot \color{red}{\frac{d}{dx} (1 + e^{-x})} = -(1 + e^{-x})^{-2} \cdot \left( \color{red}{\frac{d}{dx} [ 1 ] + \frac{d}{dx} e^{-x}} \right ) $$

Учитывая тот факт, что производная константы равна нулю, а производная экспоненты может быть найдена по формуле $\left( e^{u(x)} \right)' = e^{u(x)} \cdot u'(x)$, упростим второй множитель правой части уравнения (7)

$$\color{red}{\frac{d}{dx} [ 1 ] + \frac{d}{dx} e^{-x}}  =   e^{-x} \cdot \frac{d}{dx} (-x) = -e^{-x}$$

Подставим (8) в (5):

$$ \sigma'(x) = -(1 + e^{-x})^{-2} \cdot -e^{-x} = \frac{e^{-x}}{\left( 1 + e^{-x} \right)^2 } $$

В свою очередь, это уравнение можно переписать в виде:

$$ \boxed{\sigma'(x) =  \sigma(x) \cdot (1 - \sigma(x) ) } $$
