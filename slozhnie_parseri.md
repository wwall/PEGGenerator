# Сложные парсеры
В этой главе мы реализуем следующие парсеры 
    seq - парсер собирающий несколько парсеров в один
    alt - парсер альтернатив
    zerroOrMore - парсер разбирающий ноль или больше вхождения парсеров
    zerroOrOne - парсер разбирающий ноль или одно вхождение парсеров
    oneOrMore - парсер разбирающий один или больше вхождения парсеров
    fn - парсер, выполняющий код в случае если зависимый парсер успешно отработал


Seq
Оператор seq это функция обрабатывающая результаты подчинненых парсеров
Пример вызова 
```
	ruleDef(paser,"Exp",seq("digit",cchar("+"),"digit"));
```

Эта строка описывает создание парсера который последовательно вызывает парсер с именем digit, парсер символа "+" и снова парсер "digit".

Рассмотрим подробнее определение оператора seq
```
	function seq(where,fparser,ParserCollection)
		wrkValue = where;
		result = new Array;
		for each parser in fparser.parser do
			value = applyParser(wrkWalue,parser,parserCollection);
			if isFail(value) then
				return makeFail(where);
			endif;
			result.add(makeSucces(getValue(value),getRes(value)));
			wrkValue = getRest(value);
		enddo;
		return makeSucces(result,getRest(wrkValue));
	endfunction
```

О чем говорит этот код? Разберем построчно - 

```
	wrkValue = where;
	result = new Array;
```
Подгатавливаем служебные переменные - wrkValue это состояние для работы парсера, result - массив результатов. 

```
	for each parser in fparser.parser do	
```
Тут просто - начало цикла который проходит по коллекции 

```
	value = applyParser(wrkWalue,parser,parserCollection);
```
Получаем результа применения парсера к текущему состоянию потока. 
```
	if isFail(value) then
		return makeFail(where);
	endif;
```
Если применить парсер не удалось, то возвращаем неудачу для всей цепочки парсеров. 
```
	result.add(makeSucces(getValue(value),getRes(value)));
	wrkValue = getRest(value);
```
А если применение парсера прошло успешно - сохраняем в массиве результат и меняем состояние потока.
Когда все парсеры успешно отработали - вернем результат успешного применения оператора seq



Парсер alt

Парсер fn

Парсер seq

Парсер *

Парсер ?

Парсер +

**Парсеры просмотра**

Парсер предпросмотра с инверсией

Парсер предпросмотра
