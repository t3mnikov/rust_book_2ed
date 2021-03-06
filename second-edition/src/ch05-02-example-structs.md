## Пример использования структур

Для понимания где же лучше всего использовать структуры, мы напишим программу,
которая будет рассчитывать площадь прямоугольника. Мы начнём с создания переменных,
а потом изменение за именение напишем код, который будет использовать структуры.

Создадим проект программы Cargo *rectangles*, которая будет получать на вход длину
и ширину прямоугольника в пикселах и будет рассчитывать площадь прямоугольника.
Пример кода 5-8 проиллюстрирует это:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let length1 = 50;
    let width1 = 30;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(length1, width1)
    );
}

fn area(length: u32, width: u32) -> u32 {
    length * width
}
```

<span class="caption">Пример 5-8: Расчёт площади прямоугольника</span>

Теперь, проверим её работу `cargo run`:

```text
The area of the rectangle is 1500 square pixels.
```

### Рефакторинг программы с помощью кортежей

Хотя пример 5-8 работает и расчитывает площадь прямоугольника, мы можем улучшить
программу. Переменные длина и ширина связаны логически, т.к. они описывают параметры
прямоугольника.

Задача этого метода описана в названии:

```rust,ignore
fn area(length: u32, width: u32) -> u32 {
```

Функция `area` расчитывает площадь одного прямоугольника, но в функцию вводятся
два параметра. Эти параметры связаны, но это никак не оказывает влияние на код программы.
Было бы лучше, если бы код был более очивидным и управляемым, чтобы переменные ширина
и длина были сгруппированы вместе. Мы уже знаем один способ группировки переменны -
кортежи. Следующий пример показывает, как это можно реализовать:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let rect1 = (50, 30);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">Пример 5-8: Определени длины и ширины в кортеже</span>

Программа стала более понятной и структурированной. Но всёже есть проблема -
элемены кортежа - это безымянные индексы.

Это хорошо, что не важно в каком порядке в кортеже переменные, но если будет нужно
напечатать или сообщить какие-любо величину - можно сделать ошибку, перепутав переменные
местами.

### Изменения в коде с помощью структуры. Добавления в код читабельности и определённости

Мы будем использовать структуры для улучнешния читабельности и упраления кодом.
Пример 5-10:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.length * rectangle.width
}
```

или так:

```rust
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = &(Rectangle { length: 50, width: 30 });

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.length * rectangle.width
}
```

<span class="caption">Пример 5-10: Определение структуры `Rectangle`</span>

Мы определяем и именуем структуру `Rectangle`. Внутри фигурных скобок `{}` мы
определяем поля `length` и `width`, который имеют тип данны `u32`. Далее, в
методе `main` мы создаём экземпляр `Rectangle`, который имеет длину 50 и ширину 30.

Функция `area` также как и при работе с кортежем, использует один именованный параметр,
значение которого мы заимствуем из метода `main`. Как мы уже знаем в коде происходит
заимствование, а не изменение владения. Поэтому после работы в методе `area` переменную
`rect1` можно далее использовать в методе `main`. Заимствование осуществляется
благодаря использованию сомвола `&` при передаче параметра и в описании параметра
функции.

Функция `area` имеет доступ к полям экземпляра. Теперь определение функции точно
объясняет, что она делает - высисляет площадь прямоугольника.

### Добавим функциональности (признаки и поведение)

Было бы удобно иметь возможность печатать состояние экземпляра прямоугольника во
время отладки программы.
Пример 5-11:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {}", rect1);
}
```

<span class="caption">Пример 5-11: Попытка напечатать содержание состояние экземпляра
`Rectangle`</span>

Когда мы попытаемся выполнить этот код, то получим ошибку компиляции:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

С помощью макроса `println!` может делать различные виды форматирования. По умолчанию,
`{}` - это контейнер каких либо данных для форматирования, изветного как `Display`.
Простые встроенные типы данных реализуют `Display`. Но структуры не реализуют.
Это надо делать явно в коде.

Обратите внимание на информационное сообщение в коде:

```text
note: `Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```

Попробуем им воспользоваться. Изменим код соответствующим образом:

```rust,ignore
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {:?}", rect1);
}
```

Мы использовали вывод поведение `Debug`, которое позволяет нам напечатать содержание
структуры по умолчанию.

Выполнив это код также получим сообщение об ошибке компиляции:

```text
error: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

Но, как и в прошлый раз компилятор спешит к нам на помощь и даёт ценные и важные
указания:

```text
note: `Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```

Rust даёт возможность напечатать отладочную информацию, но для этого нам надо
явно описать это поведение. Для это необходимо аннотировать определение структуры
следующим текстом `#[derive(Debug)]`.

Пример 5-12:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {:?}", rect1);
}
```

<span class="caption">Listing 5-12: Добавление аннтации для использования поведения
`Debug`</span>

Теперь ошибок компиляции не будет. Ура!

```text
rect1 is Rectangle { length: 50, width: 30 }
```

Это, конечно, не самый лучший способ представления информации, но для отладки подойдёт.
Для более сложных структур было бы лучшим решением сделать вывод инфомации более удобным.
Для этого используем аннотацию `{:#?}` вместо `{:?}`.

```rust
#[derive(Debug)]
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {:#?}", rect1);
}
```
Результат:

```text
rect1 is Rectangle {
    length: 50,
    width: 30
}
```

Rust предоставляет специальные объекты - признаки для реализаци поведения в структурах.
Мы раскажем как работать с этими объектами - поведениями в главе 10.

Функция `area` весьма специфична. Она делает расчет площади прямоугольника.
Было бы лучеше, если бы поведение было связано с структурой, т.к. функция может работать
только с этой ней. В следующей главе мы расскажем, как улучшить код и реализовать
эту связь. Мы расскажем, как сделать из обособленной функции метод структуры.
