## Конструктор

 Конструкторы в JavaScript тоже действуют отличным от большинства языков образом. Любая функция, вызванная с использованием ключевого слова `new`, станет конструктором.

Внутри конструктора (вызываемой функции) `this` будет указывать на новосозданный `Object`. [Прототипом](#object.prototype) этого **нового** экземпляра будет `prototype` функции, которая была вызвана под видом конструктора.

Если вызываемая функция не возвращает явного значения посредством `return`, то она автоматически вернёт `this` — тот самый новый экземпляр.

    function Foo() {
        this.bla = 1;
    }

    Foo.prototype.test = function() {
        console.log(this.bla);
    };

    var test = new Foo();

В этом примере `Foo` вызывается в виде конструктора, следовательно прототип созданного объекта будет привязан к `Foo.prototype`.

В случае, когда функция в явном виде возвращает некое значение используя `return`, в результате выполнения конструктора мы получим именно его, **но только** если возвращаемое значение представляет собой `Object`.

    function Bar() {
        return 2;
    }
    new Bar(); // новый объект

    function Test() {
        this.value = 2;

        return {
            foo: 1
        };
    }
    new Test(); // возвращённый объект

Если же опустить ключевое слово `new`, то функция **не** будет возвращать никаких объектов.

    function Foo() {
        this.bla = 1; // свойство bla устанавливается глобальному объекту
    }
    Foo(); // возвращает undefined

Хотя этот пример и будет работать — в связи с поведением [`this`](#function.this) в JavaScript, значение будет присвоено *глобальному объекту* — навряд ли это предполагалось его автором.

### Фабрики

Если вы хотите предоставить возможность опускать оператор `new` при создании объектов, возвращайте из соответствующего конструктора явное значение посредством `return`.

    function Bar() {
        var value = 1;
        return {
            method: function() {
                return value;
            }
        }
    }
    Bar.prototype = {
        foo: function() {}
    };

    new Bar();
    Bar();

В обоих случаях при вызове `Bar` мы получим один и тот же результат — новый объект со свойством `method`, являющимся [замыканием](#function.closures)).

Ещё следует заметить, что вызов `new Bar()` никак **не** воздействует на прототип возвращаемого объекта. Хоть прототип и назначается всем новосозданным объектам, `Bar` никогда не возвращает этот новый объект (_прим. перев._ — судя по всему, подразумевается, что код `Bar` не может влиять на прототип созданного объекта, и под словами «новый объект» в последнем случае кроется прототип нового объекта, а не сам новый объект).

В предыдущем примере нет никаких функциональных различий между вызовом конструктора с оператором `new` и вызовом без него.

### Создание объектов с использованием фабрик

Нередко встречаются советы **не** использовать оператор `new`, поскольку если вы его забудете, это может привести к ошибкам.

Чтобы создать новый объект, нам предлагают использовать фабрику и создать новый объект *внутри* этой фабрики.

    function Foo() {
        var obj = {};
        obj.value = 'blub';

        var private = 2;
        obj.someMethod = function(value) {
            this.value = value;
        }

        obj.getPrivate = function() {
            return private;
        }
        return obj;
    }

Хотя данный пример и сработает, если вы забыли ключевое слово `new` и, возможно, благодаря ему вам станет легче работать с [приватными переменными](#function.closures), у него есть несколько недостатков:

 1. Он использует больше памяти, поскольку созданные объекты **не** хранят методы в прототипе и соответственно для каждого нового объекта создаётся копия каждого метода.
 2. Чтобы эмулировать наследование, фабрике нужно скопировать все методы из другого объекта или установить прототипом нового объекта старый.
 3. Разрыв цепочки прототипов по мнимой необходимости избавиться от использования ключевого слова `new` идёт вразрез с духом языка.

### Заключение

Хотя забытое ключевое слово `new` и может привести к багам, это точно **не** причина отказываться от использования прототипов. В конце концов, полезнее решить какой из способов лучше совпадает с требованиями приложения: крайне важно выбрать один из стилей создания объектов и после этого **не изменять** ему.

