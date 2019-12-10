---
layout: article
title: Решение СЛАУ с трехдиагональной матрицей методом прогонки
modify_date: 2011-12-28
tags: old numerical_analysis algorithms
image_path: /assets/images/2009-09-11-tridiagonal-matrix-algorithm
---

О том, xто такое метод прогонки, можно прочитать [здесь](https://ru.wikipedia.org/wiki/%D0%A7%D0%B8%D1%81%D0%BB%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5_%D0%BC%D0%B5%D1%82%D0%BE%D0%B4%D1%8B).

С реализацией я особо не заморачивался, поэтому консольный режим. Написана в среде MS Visual Studio 2005.

Зависимости:

* `stdafx.h`
* `conio.h`
* `iostream.h`
* `iomanip.h`

Полный код представлен ниже.

~~~

/*
 * Copyright (c) 2009 Argrento
 */

#include "stdafx.h"
#include "conio.h"

//Директивы для работы с потоками ввода-вывода
#include <iostream>
#include <iomanip>

//Используем cout, cin и endl из пространства имен std
using std::cout;
using std::cin;
using std::endl;

//Самая-самая главная процедура 8-)
int _tmain(int argc, _TCHAR* argv[]) {
  cout<<"Kolichestvo peremennih >>> ";
  int n;

  //Вводим в переменную данные с потока
  cin >> n;

  //Выделение памяти под основной массив
  float *base = (float *)calloc(n * (n + 1), sizeof(float));

  //Эти два вложенных цикла используются для ввода элементов массива
  for (int i=0; i<n; i++) {
    for (int j=0; j<n; j++) {
      cout << "Vvedite element "<<j + 1<<" stroki "<<i + 1<<" >>> ";
      cin >> *(base + i * (n + 1) + j);
    }
    cout << "Vvedite otvet k stroke "<<i + 1<<" >>> ";
    cin >> *(base + i*(n + 1) + n);
  }

  //Следующие строчки мне нужны были для контроля индексации
  cout << endl << endl;
  for (int i=0; i<n*n+n; i++) {
    cout << "*(base + " << i << ") = " << *(base + i) << endl;
  }


  //Выделение массивов для вспомогательных коэффициентов
  float *alpha;
  float *betta;
  alpha = (float *)calloc(n, sizeof(float));
  betta = (float *)calloc(n, sizeof(float));

  //Расчет и вывод первых коэффициентов
  *alpha = - *(base + 1) / *base;
  *betta = *(base + n) / *base;
  cout << endl << "alpha 1 = " << *alpha << endl;
  cout << "betta 1 = " << *betta << endl;

  //В этом АЦЦКОМ цикле производится расчет всех остальных коэффициентов. Также имеется вывод. Он сдесь опять же для отлова ошибок... ну и для красоты :-)

  for (int i = 1; i < n; i++) {
    *(alpha + i) = -*(base + i*(n + 1) + i + 1) /
                     (*(base + i * (n + 1) + i - 1) * *(alpha + i - 1) +
                      *(base + i * (n + 1) + i));

    *(betta + i) = (*(base + i * (n + 1) + n) -
                    *(base + i * (n + 1) + i - 1) * *(betta + i - 1)) /
                   (*(base + i * (n + 1) + i) +
                    *(base + i * (n + 1) + i - 1) * *(alpha + i - 1));

    cout << "alpha"
         << i
         << " = -"
         << *(base + i * (n + 1) + i + 1)
         << " / ("
         << - *(base + i * (n + 1) + i + 1)
         << " * "
         << *(alpha + i - 1 )
         << " + "
         << *(base + i*(n + 1) + i)
         << " ) = "
         << *(alpha + i)
         << endl;

    cout << "betta"
         << i
         << " = ("
         << *(base + i * (n + 1) + n)
         << " - "
         << *(base + i * (n + 1) + i - 1)
         << " * "
         << *(betta + i - 1)
         << ") / ("
         << *(base + i * (n + 1) + i - 1)
         << " * "
         << *(alpha + i - 1)
         << " + "
         << *(base + i * (n + 1) + i)
         << ") = "
         << *(betta + i)
         << endl;
  }

  //Выделение память под массив ответов
  float *ans = (float*)calloc(n, sizeof(float));

  //Вычисление первого последнего корня уравнения (Xn)
  *(ans + n) = (*(base + n * (n + 1)) - *(base + n * (n + 1) - 2) * *(betta + n)) /
               (*(base + n * (n + 1) - 2) * *(alpha + n) + *(base + n * (n + 1) - 1));

  //В этом цикле определяются предыдущие корни уравнения (X(n-1)..X1)
  for (int i = n - 1; i >= 0; i--) {
    *(ans + i) = *(alpha + i)**(ans + i + 1) + *(betta + i);
  }

  //А здесь эти корни выводятся на экран
  for(int i=0; i<n; i++) {
    cout << "Koren' " << i + 1 << " = " << *(ans + i) << endl;
  }

  //Задержка экрана
  getch();

  //Конец :-)
  return 0;
}
~~~
{: .language-cpp}

Как и всегда, все очень просто и понятно.
