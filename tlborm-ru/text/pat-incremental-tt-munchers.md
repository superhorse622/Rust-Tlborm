% Последовательный потребитель TT

```rust
macro_rules! mixed_rules {
    () => {};
    (trace $name:ident; $($tail:tt)*) => {
        {
            println!(concat!(stringify!($name), " = {:?}"), $name);
            mixed_rules!($($tail)*);
        }
    };
    (trace $name:ident = $init:expr; $($tail:tt)*) => {
        {
            let $name = $init;
            println!(concat!(stringify!($name), " = {:?}"), $name);
            mixed_rules!($($tail)*);
        }
    };
}
# 
# fn main() {
#     let a = 42;
#     let b = "Ho-dee-oh-di-oh-di-oh!";
#     let c = (false, 2, 'c');
#     mixed_rules!(
#         trace a;
#         trace b;
#         trace c;
#         trace b = "They took her where they put the crazies.";
#         trace b;
#     );
# }
```

Этот паттерн - наверное, *самая мощная* доступная техника разбора макросов,
позволяющая разбирать грамматику любой сложности.

"Потребитель TT" - это рекурсивный макрос, который работает, последовательно
обрабатывая шаг за шагом то, что ему подали на вход. На каждом шаге он ищет
совпадение с образцом и удаляет (потребляет) сочетание токенов из головы входа,
создает какой-то промежуточный результат, затем вызывает сам себя с оставшейся
частью входа в качестве аргумента.

Причина по которой "TT" указано в имени  - необработанная часть входа *всегда*
захватывается как `$($tail:tt)*`. Происходит это, потому что повторение `tt` -
это единственный способ *без потерь* захватить часть входа макроса.

Единственными жесткими ограничениями, которые накладываются на потребитель TT,
являются те же, что и на всю систему макросов в целом:

* Вы можете искать совпадение только с литералами или грамматическими
конструкциями, которые могут захватываться `macro_rules!`.

* Вы не можете искать совпадение с синтаксически неправильной конструкцией.

В то же время важно помнить о лимите рекурсии макросов. `macro_rules!` *никак*
не устраняет и не оптимизирует хвостовую рекурсию. При написании потребителя TT
рекомендуется предпринимать все усилия, чтобы удержаться в лимите рекурсии.
Можно сделать это, добавив дополнительные правила для учета вариантов входа (что
уменьшит рекурсию в промежуточном слое), или пойти на компромисс и изменить
входную последовательность для использования стандартных повторений.