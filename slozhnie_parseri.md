# Сложные парсеры
В этой главе мы реализуем следующие парсеры 
* seq - парсер собирающий несколько парсеров в один
* alt - парсер альтернатив
* zerroOrMore - парсер разбирающий ноль или больше вхождения парсеров
* zerroOrOne - парсер разбирающий ноль или одно вхождение парсеров
* oneOrMore - парсер разбирающий один или больше вхождения парсеров
* fn - парсер, выполняющий код в случае если зависимый парсер успешно отработал

#Seq

Оператор seq это функция обрабатывающая результаты подчинненых парсеров
Пример вызова 
```
	ruleDef(paser,"Exp",seq("digit",cchar("+"),"digit"));
```

Эта строка описывает создание парсера который последовательно вызывает парсер с именем digit, парсер символа "+" и снова парсер "digit".

Рассмотрим подробнее определение оператора seq
```
	function applyParser_seq(where,fparser,ParserCollection)
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
Подгатавливаем служебные переменные - 
* wrkValue это состояние для работы парсера, 
* result - массив результатов. 

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


Осталось сделать опеределение для вызова. И тут возникает сложность между способом хранения парсеров (массив) и способом передачи. Разделим функцию seq на 2 функции с разныем набором формальных параметров.

```


function wrap(x)
    return ?(typeof(x)=type("String"),ref(x),x);
endfunction

procedure Add2Array(x,array)
    if x <> undefined then
        Array.Add(wrap(x));
    Endif
endprocedure

function toArray(a1=undefined,a2=undefined,a3=undefined,a4=undefined,a5=undefined,a6=undefined,a7=undefined,a8=undefined,a9=undefined,a0=undefined)
    Array = new Array;
    Add2Array(a1,Array);
    Add2Array(a2,Array);
    Add2Array(a3,Array);
    Add2Array(a4,Array);
    Add2Array(a5,Array);
    Add2Array(a6,Array);
    Add2Array(a7,Array);
    Add2Array(a8,Array);
    Add2Array(a9,Array);
    Add2Array(a0,Array);
    return Array;
endfunction

function seq(a1,a2,a3=undefined,a4=undefined,a5=undefined,a6=undefined,a7=undefined,a8=undefined,a9=undefined,a0=undefined)
    return new Structure("type,parsers","seq",toArray(a1,a2,a3,a4,a5,a6,a7,a8,a9,a0));
endfunction


function _seq(Array)
    return new Structure("type,parsers","seq",Array);
endfunction


```

Как видно из кода - каждый параметр передаваемый в функцию seq проходит проверку в функции wrap. Эта проверка нужна толь для того что бы обеспечить единообразие в хранении парсеров для функции applyParser.  

Тест отрабатывает успешно, можем переходить к реализации следующего оператора


#Alt

Оператор Alt выполняет функцию выбора. Он возвращает первый успешно отработавший парсер.
Традиционно - тест 
```
function testParser_seq() export
    parser = new Structure;
    RuleDef(parser,"testAlt",alt(ichar("a"),ichar("B")));
    value = "bA";
    result = runParser(value,"testAlt",parser);
    wait = makeSuccess(new Structure("value,position","b",new Structure("position,line,column",2,1,2)), (new Structure("position,line,column",2,1,2)));
    Ожидаем.Что(result).Равно(wait);
endfunction
```

Грамматика которую разбирает этот тест

```
	testAlt = "а"|"B"
```


Код оператора ниже

```
function applyParser_alt(where,fparser,parser) export
    for each wrkParser in fparser.parsers do
        wrkData  = ApplyParser(where,wrkParser,parser);
        if isSuccess(wrkData) then
            return wrkData;
        endif;
    enddo;
    return makeFailure(where);
endfunction

```

Как видно - ничего волшебного не происходит - оператор бежит по списку подчиненных парсеров, и возвращает результат работы парсера который успешно сработал
Остается только описать конструктор для этого оператора. Он простой - 

```
function alt(a1,a2,a3=undefined,a4=undefined,a5=undefined,a6=undefined,a7=undefined,a8=undefined,a9=undefined,a0=undefined)
    return new Structure("type,parsers","alt",toArray(a1,a2,a3,a4,a5,a6,a7,a8,a9,a0));
endfunction

function _alt(Array)
    return new Structure("type,parsers","alt",Array);
endfunction

```

Запускаем тесты и убеждаемся что все работает. 


# Парсер zerroOrMore *


Парсер ?

Парсер +


Парсер fn

**Парсеры просмотра**

Парсер предпросмотра с инверсией

Парсер предпросмотра
