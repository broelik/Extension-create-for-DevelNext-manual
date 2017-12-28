# Предисловие
В прошлой части, мы разобрались как создать расширение для JPHP, создать класс из **BaseObject**, а также как возращать значение в php. Эта же часть полностью посвящена классу **BaseWrapper**.
# BaseWrapper
**BaseWrapper** создан для удобства разработчиков. С его помощью, легче осуществлять работу с другими классами Java, а также осуществлять преобразование и обратное преобразование классов. Возможно сейчас вам ничего не понятно, но на практике будет намного легче.
## Кодим
Для начала создадим класс **CurrencyWrapper**, который будет наследовать класс **php.runtime.lang.BaseWrapper** и для примера, в качестве аргумента передадим ему класс **java.util.Currency**.
```java
@Name("Currency")
@Namespace(TestExtension.NS)
@Final
public class CurrencyWrapper extends BaseWrapper<Currency>
{
```
Мы видим, что первые две аннотации, не чем не отличаются от тех, которые мы разбирали в прошлой части. Однако за ними следует другая аннотация, а именно - **Final**.
**Final**, делает наш класс невозможным для расширения другими классами. Это нужно, так как Java класс **Currency** сам является финальным.
Затем мы опредялем класс **CurrencyWrapper** который наследует класс **BaseWrapper**. В качестве аргумента мы передали **Currency** и теперь при вызове метода **getWrappedObject** мы получим экземпляр класс **Currency**. Естественно, если вы хотите чтобы ваш класс имел конструктор, то вы должны инициализировать класс над которым делаете обёртку, в конструкторе вашего класса. Сделать это можно присвоив "защищённой" переменной **__wrappedObject** экземпляр вашего класса, например таким образом:
```java
@Signature
public void __construct(String currencyCode)
{
	__wrappedObject = Currency.getInstance(currencyCode);
}
```
Также если у класса несколько конструкторов, которые принимают одинаковое количество аргументов, то решить эту проблему, можно при помощи инициализации экземпляра класса при вызове статистического метода:
```java
@Signature
public static Memory of(Environment env, WrapLocale locale)
{
	return Memory.wrap(env, Currency.getInstance(locale.getLocale()));
}
```
Теперь экземпляр класса можно создать при помощи конструктора, который принимает строку в качестве аргумента и статистического метода, который в свою очередь принимает **Locale**.
## Наполняем класс методами
У нас уже есть экземпляр класса. Но надо же с ним как-то взаимодействовать. Немного ранее я говорил про метод **getWrappedObject**. Этот метод возращает экземпляр класса, над которым мы делаем обёртку.
Давайте создадим несколько методов для нашего класса:
```java
@Signature
public Memory getSymbol(WrapLocale locale)
{
	return StringMemory.valueOf(getWrappedObject().getSymbol(locale.getLocale()));
}
@Getter
public Memory getCurrencyCode()
{
	return StringMemory.valueOf(getWrappedObject().getCurrencyCode());
}
@Getter
public Memory getDefaultFractionDigits()
{
	return LongMemory.valueOf(getWrappedObject().getDefaultFractionDigits());
}
@Signature
public Memory getSymbol()
{
	return StringMemory.valueOf(getWrappedObject().getSymbol());
}
```
Как и в случае с **BaseObject**, наши методы возращают Memory, но здесь вместо переменной класса, которую мы сами создавали, мы используем **getWrappedObject** для вызова сигнатурных методов, а также вместо **@Signature**, в некоторых методах мы используем аннотацию **@Getter**.
### @Getter и @Setter
Возможно, вы уже встречали что-то подобное, в C#. Эти методы вызываются в тот момент, когда вы обращаетесь к переменной класса, название которой содержится в названии метода. То есть **getCount**, будет вызван когда вы пытаетесь получить значение переменной класса **count**, а соответственно **setCount**, когда пытаетесь присвоить значение **count**.

Наш класс почти готов к использованию, осталось его зарегистрировать в методе **onRegister**, главного класса нашего расширения:
```
registerWrapperClass(scope, Currency.class, CurrencyWrapper.class);
```
Затем, его можно протестировать:
```php
Runtime::addJar("test-ext.jar");
$object = \test\Currency::of(Locale::RUSSIA());
//$object = new \test\Currency("USD");
var_dump($object->currencyCode);
var_dump($object->defaultFractionDigits);
var_dump($object->getSymbol());
var_dump($object->getSymbol(Locale::ENGLISH()));
```
Результат будет примено следующим:
```
string(3) "RUB"
int(2)
string(4) "руб."
string(3) "RUB"
```
# Автокомплит и документация
На данный момент, все импорты нам нужно прописывать в ручную, ну и пока мы подключаем класс, через код, естественно среда не сможет дать нам никаких подсказок. Решить эту проблему очень просто!
В src директории нашего проекта создаём папку:
```
JPHP-INF/sdk/php/
```
В этой папке будут хранится php файлы, описывающии методы наших классов.
На данный момент у нас есть два класса:
* test\TestObject
* test\Currency
Соответственно создаём директорию test в JPHP-INF/sdk/php/, а в ней файлы TestObject.php и Currency.php. То есть php описание наших классов будет хранится по пути:
```
JPHP-INF/sdk/php/{namespace}/{ClassName}.php
```
Содержимое этих файлов, среда будет использовать для помощи при вводе кода. А содержимое должно быть примерно таким:
```php
namespace test;

class TestObject
{
    function __construct(){}
    function append($str){}
    function isEmpty(){}
    function length(){}
}
```
Также, желательно использовать комментарии:
```php
namespace test;

use php\util\Locale;

class Currency
{
    public $symbol;
    public $currencyCode;
    
    function __construct($currencyCode){}
    /**
     * @param Locale $locale
     * return Currency
     */
    static function of(Locale $locale){}
    function getSymbol(Locale $locale = null){}
}
```
На данный момент, мы естественно ничего не увидем в среде, так как это расширение не подключено к DevelNext.
# Заключение
Для подключения к среде, нужно собрать расширение в файл **dnbundle**. В следующей части мы этим и займёмся. По вопросам - https://vk.com/dn_extension или https://vk.com/broelik.
