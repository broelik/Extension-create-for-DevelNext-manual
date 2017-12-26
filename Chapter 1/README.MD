# Предисловие
Наверное многие из вас сталкивались с проблемой не хватки какого-то функционала в DevelNext. Решить это можно несколькими способами. Первый способ для ленивых - ждать пока необходимый функционал реализуют разработчики DevelNext или же  сторонние разработчики. Второй способ - использовать стороннее ПО, но это чревато проблемами, да и это самый что ни на есть костыль! Ну и третий способ - написать собственное расширение, реализующее необходимый вам функционал. 
# Подготовка
Список необходимого:
* Java 8
* Eclipse
* Установленный JavaFX
* Основы Java

Первое что мы должны сделать - создать проект. 
**File -> New -> Java Project**

Далее вводим название нашего проекта (например "TestExtension"), выбираем Java 8 и нажимаем кнопку "Next >". 
![](/Chapter%201/1.jpg)

На этой странице нам требуется добавить необходимую библиотеку jphp-runtime.
![](/Chapter%201/2.jpg)

Найти её можно в %Папка с установленной DevelNext%/lib. Обычно DevelNext установлен в %programfiles%/DevelNext.
![](/Chapter%201/3.jpg)

Жмём Finish и ждём окончания загрузки.

# Первый код
Создаём пакет **org.develnext.jphp.ext.{extensionlowercasename}** и добавляем в него класс **{ExtensionName}Extension**. Этот класс должен наследовать **php.runtime.ext.support.Extension** и обязательно включать в себя методы **getStatus** и **onRegister**:
```java
package org.develnext.jphp.ext.testextension;
...
public class TestExtension extends Extension
{
	@Override
	public Status getStatus()
	{
		return Status.EXPERIMENTAL;
	}
	@Override
	public void onRegister(CompileScope scope)
	{
		// TODO register classes here
	}
}
```

Разберём код подробней. Сначала идёт название нашего пакета. Затем следует объевления главного класса нашего расширения который наследуется от Extension. Далее идут методы **getStatus**, который возвращает статус разработки расширения и **onRegister**, который компилятор вызовет при подключении расширения к проекту.
Пока наше расширение не добавляет ни одного класса, но это немного позже. Нам нужно сделать так, чтобы компилятор понимал, какой класс использовать для получения необходимой информации о расширении. Для этого мы создадим в корневой папке нашего проекта, файл **META-INF/services/php.runtime.ext.support.Extension** и пропишем в нём полный путь до нашего класса, в моём случае это - **org.develnext.jphp.ext.testextension.TestExtension**.
## Создание классов
На данный момент, наше расширение уже можно использовать, но в нём нет ни одного класса! Нужно это исправить.
Создадим пакет **org.develnext.jphp.ext.{extensionlowercasename}.classes**. В этом пакете будут находится все классы, которые мы можем использовать непосредственно в DevelNext. В этом же пакете создадим класс с именем, например **TestObject**. Он будет наследовать **php.runtime.lang.BaseObject** и должен включать в себя конструктор **(Environment env, ClassEntity clazz)**:
```java
package org.develnext.jphp.ext.testextension.classes;
...
public class TestObject extends BaseObject
{
	public TestObject(Environment env, ClassEntity clazz)
	{
		super(env, clazz);
	}
}
```
На данный момент этот класс DevelNext никак не сможет идентифицировать. Это проблему можно решить добавив аннотации **@Name** и **@Namespace**, но перед этим мы для удобства добами константу NS, с namespace нашего расширения, в главный класс нашего расширения:
```java
public class TestExtension extends Extension
{
	public static final String NS = "test";
```
Теперь мы можем добавить имя и namespace нашему классу:
```java
@Name("TestObject")
@Namespace(TestExtension.NS)
public class TestObject extends BaseObject
{
```
Теперь путь до нашего класса в PHP выглядит так:
**test\TestObject**
## Наполняем класс методами
На данный момент, наш класс не имеет ни одного метода и даже конструктора.
Для создания, так называемых, сигнатурных методов нужно использовать аннотация **@Signature**:
```java
	private StringBuilder builder;
	@Signature
	public void __construct()
	{
		builder = new StringBuilder();
	}
```
Для примера, я объявил переменную класса builder и присваиваю ей значение в конструкторе.
Передать аргументы в функцию просто:
```java
	@Signature
	public void append(String s)
	{
		builder.append(s);
	}
```
А что если, нам нужно вернуть какое-то значение? В этом нам поможет класс **Memory** и унаследованные от него **LongMemory**, **StringMemory** и т.д:
```java
	@Signature
	public void append(String s)
	{
		builder.append(s);
	}
	@Signature
	public Memory isEmpty()
	{
		return builder.length() == 0 ? Memory.TRUE : Memory.FALSE;
	}
	@Signature
	public Memory length()
	{
		return LongMemory.valueOf(builder.length());
	}
```
Осталось зарегистировать класс. Для этого переходим в главый класс нашего расширения и в методе **onRegister** прописываем следующее:
```
registerClass(scope, TestObject.class);
```
Теперь наконец-то можно протестировать наше расширение. 
