---
layout: article
title: Аппроксимация таблично заданной функции методом кубического сплайна
modify_date: 2011-12-28
tags: old numerical_analysis algorithms
image_path: /assets/images/2009-10-11-cube-approximation
---

Наконец-то я закончил эту программу. Как и предыдущая работа, она написана с использованием потоков и динамичеких массивов в консольном режиме. Входные данныей программы -- таблица значений некоторой неизвестной функции. После отработки программы на экране появятся уравнения кубических сплайнов. Пример построения -- в низу страницы.

Теория по теме: [Страница 1]({{page.image_path}}/00-page-1.jpg), [Страница 2]({{page.image_path}}/01-page-2.jpg)

Это сканы из методички по численным методам. В алгоритме расчета сплайнов присутствует трехдиагональная матрица. Программа нахождения ее корней – смотри предыдущую запись.

Итак, код.
~~~
/*
 * Copyright (c) 2009 Argrento
 */

#include <iostream>
#include <iomanip>
#include <math.h>

using std::cin;
using std::cout;
using std::endl;


// Функция решения трехдиагональной матрицы (solving tridiagonal matrix)
void solveMatrix(
  int,   // Размерность (dimension)
  float *, // Входная матрица (input matrix)
  float *, // Вектор ответов (answer vector)
  bool   // Надо ли выводить промежуточные вычисления (show intermediate calculations results, or not)
);

int main(int argc, char * argv[])
{
  // Input number of points
  cout << "Vvedite kolichestvo tochek >>> ";
  int n = 10;
  cin >> n;

  // Выделяем память под массив таблицы значений (memory allocation)
  float * x_table = (float *) calloc(n, sizeof(float));
  float * y_table = (float *) calloc(n, sizeof(float));

  // Ввод таблицы значений (input array of points)
  for (int i = 0; i < n; i++) {
    cout << "Input x" << i + 1 << " >>> ";
    cin >> *(x_table + i);

    cout << "Input y" << i + 1 << " >>> ";
    cin >> *(y_table + i);
  }

  // Вывод таблицы значений (output array of points)
  cout << endl << ">>> INPUT TABLE <<<" << endl << endl;
  for (int i = 0; i < n; i++) {
    cout << "x" << i + 1 << "= " << *(x_table + i) << " ";
    cout << "y" << i + 1 << "= " << *(y_table + i) << endl;
  }

  // Для удобства :) (for convenience)
  n -= 2;

  // Выделение памяти под коэффициенты С и всю матрицу (memory allocation for base matrix and answer vecor)
  static float * c    = (float *) calloc(n + 2, sizeof(float));
  static float * matrix = (float *) calloc((n) * (n + 1), sizeof(float));

  //Расчет шага (step size calculation)
  float h = (*(x_table + 1) - *x_table);
  cout << endl << endl;


  // Заполнение матрицы (This will fill the base matrix...)
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < n + 1; j++) {
      if (i == j) {
        *(matrix + i * (n + 1) + j) = 4;
      } else if ((i == j + 1) || (i == j - 1)) {
        *(matrix + i * (n + 1) + j) = 1;
      } else {
        *(matrix + i * (n + 1) + j) = 0;
      }
      cout << *(matrix + i * (n + 1) + j) << " ";
    }
    cout << endl;
  }

  // Заполнение столбца ответов в расширенной матрице (...and the answer victor)
  for (int i = 1; i < n + 1; i++) {
    *(matrix + (i - 1) * (n + 1) + n) = 3 / h / h
      * (*(y_table + i + 1) - 2 * *(y_table + i) + *(y_table + i - 1));
  }

  for (int i = 0; i < n; i++) {
    for (int j = 0; j < n + 1; j++) {
      cout << *(matrix + i * (n + 1) + j) << " ";
    }
    cout << endl;
  }

  solveMatrix(n, matrix, c, false);

  //Решение этой матрицы - нахождение коэффициентов С
  // (solving base matrix)

  // Отсев погрешностей (destruction of errors)
  for (int i = 0; i < n; i++) {
    if (fabs(*(c + i)) <= 0.00001) *(c + i) = 0;
  }
  // Выделение памяти под массивы остальных коэффицентов (memory allocation)
  n += 2;
  float * a = (float *) calloc(n, sizeof(float));
  float * b = (float *) calloc(n, sizeof(float));
  float * d = (float *) calloc(n, sizeof(float));

  // Добавление в массив С первого и последнего элемента со значением 0 (adding to the answer vector 2 elements)
  int p = n - 1;
  while (p != 0) {
    p--;
    *(c + p + 1) = *(c + p);
  }
  *c = 0;
  *(c + n - 1) = 0;

  // Вычисление остальных коэффициентов сплайнов (spline factors calculation)
  for (int i = 1; i < n - 1; i++) {
    *(a + i) = *(y_table + i);
    *(b + i) = (*(y_table + i + 1) - *(y_table + i)) / h
      - h / 3 * (*(c + i + 1) + 2 * *(c + i));
    *(d + i) = (*(c + i + 1) - *(c + i)) / 3 / h;
  }
  // Вывод сплайнов (splines output)
  for (int i = 1; i < n - 1; i++) {
    cout << "S"
       << i
       << "(x) = "
       << *(a + i)
       << " + "
       << *(b + i)
       << "*(x - "
       << *(x_table + i)
       << ") + "
       << *(c + i)
       << "*(x - "
       << *(x_table + i)
       << ")**2 +"
       << *(d + i)
       << "*(x - "
       << *(x_table + i)
       << ")**3" << endl;
  }
} // main

// Код функции решения трехдиагональной матрицы (solving function code)
void solveMatrix(int n, float * base, float * ans, bool view)
{
  for (int i = 0; i < n * n + n; i++) {
    cout << "*(base+" << i << ")=" << *(base + i) << endl;
  }

  float * alpha;
  float * betta;

  alpha = (float *) calloc(n, sizeof(float));
  betta = (float *) calloc(n, sizeof(float));

  *alpha = -*(base + 1) / *base;
  *betta = *(base + n) / *base;

  if (view) {
    cout << endl << "alpha 1 = " << *alpha << endl;
  }

  if (view) {
    cout << "betta 1 = " << *betta << endl;
  }

  for (int i = 1; i < n; i++) {
    *(alpha + i) = -*(base + i * (n + 1) + i + 1)
      / (*(base + i * (n + 1) + i - 1) * *(alpha + i - 1) + *(base + i * (n + 1) + i));

    if (view) {
      cout << "alpha" << i << " = -" << *(base + i * (n + 1) + i + 1) << "/("
         << -*(base + i * (n + 1) + i + 1) << "*" << *(alpha + i - 1) << "+" << *(base + i * (n + 1) + i)
         << ")=" << *(alpha + i) << endl;
    }

    *(betta + i) = (*(base + i * (n + 1) + n) - *(base + i * (n + 1) + i - 1) * *(betta + i - 1))
      / (*(base + i * (n + 1) + i - 1) * *(alpha + i - 1) + *(base + i * (n + 1) + i));

    if (view) {
      cout << "betta" << i << " = (" << *(base + i * (n + 1) + n) << "-" << *(base + i * (n + 1) + i - 1)
         << "*" << *(betta + i - 1);
    }

    if (view) {
      cout << ")/(" << *(base + i * (n + 1) + i - 1) << "*" << *(alpha + i - 1) << "+"
         << *(base + i * (n + 1) + i) << ")=" << *(betta + i) << endl;
    }
  }

  *(ans + n) = (*(base + n * (n + 1)) - *(base + n * (n + 1) - 2) * *(betta + n))
    / (*(base + n * (n + 1) - 2) * *(alpha + n) + *(base + n * (n + 1) - 1));

  for (int i = n - 1; i >= 0; i--) {
    *(ans + i) = *(alpha + i) * *(ans + i + 1) + *(betta + i);
  }

  if (view) {
    for (int i = 0; i < n; i++) {
      cout << "Koren' " << i + 1 << " = " << *(ans + i) << endl;
    }
  }
} // solveMatrix

~~~
{: .language-cpp}

Пример аппроксимации функции y=sin(x) (Approximation example)

![]({{page.image_path}}/03-sinx.png)
