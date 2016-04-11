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
	ruleDef(paser,"Exp",seq(ichar("a"),ichar("B")));
```

Эта строка описывает создание парсера который последовательно вызывает парсер символа "а" без учета регистра, а затем парсер символа "В" без учета регистра.

Тест для этого парсера выглядит так - 
```
function testParser_seq() export
	
	parser = new Structure;
	RuleDef(parser,"Start",seq(ichar("a"),ichar("B")));
	result = runParser("Ab","Start",parser);
	resultArray = new array;
	resultArray.Add(new Structure("value,position","A",new Structure("position,line,column",2,1,2)));
	resultArray.Add(new Structure("value,position","b",new Structure("position,line,column",3,1,3)));
	wait = makeSuccess(resultArray, (new Structure("position,line,column",3,1,3)));
	assert.What(result).Equal(wait);
endfunction
```
Как видно - парсер seq должен возвращать массив, где содержится результат работы парсеров

Вот один из возможных вариантов функции applyParser_seq
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
Тут просто - начало цикла который проходит по коллекции парсеров которые должны вернуть успех при разборе

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

Заметьте - каждый параметр передаваемый в функцию seq проходит проверку в функции wrap. Эта проверка нужна только для того что бы обеспечить единообразие в хранении парсеров для функции applyParser, больше никакой полезной нагрузки он не несет.

Тест отрабатывает успешно, можем переходить к реализации следующего оператора


#Alt

Оператор Alt выполняет функцию выбора. Он возвращает первый успешно отработавший парсер.
Традиционно - тест который описывает граматику 
```
	testAlt = "a" | "B"
```
т.е. символ либо а либо b без учета регистра. Строка подаваемая на вход - "bA" и должен успешно быть разобран первый символ.
```
function testParser_alt() export
	parser = new Structure;
	RuleDef(parser,"testAlt",alt(ichar("a"),ichar("B")));
	value = "bA";
	result = runParser(value,"testAlt",parser);
	resultArray = new Structure("value,position","b",new Structure("position,line,column",2,1,2));
	wait = makeSuccess(resultArray, (new Structure("position,line,column",2,1,2)));
	assert.What(result).Equal(wait);
endfunction
```

Код оператора ниже

```
function applyParser_alt(where,fparser,parser) export
	result = new Array;
	for each wrkParser in fparser.parsers do
		wrkData  = ApplyParser(where,wrkParser,parser);
		if isSuccess(wrkData) then
			return wrkData;
		endif;
	enddo;
	return makeFailure(where);
endfunction
```

Как видно - ничего волшебного не происходит - оператор бежит по списку подчиненных парсеров, и возвращает результат работы первого парсера который успешно сработал.
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

Оператор ? необходим для ситуации когда появление парсера - опционально. Например грамматика

```
	testQuest = "а" "B"?
```
успешно обработает и строку "аВ" и "а"

Традиционно - тест. Точнее 2 теста - один для сутации когда находящийся под оператором ? парсер сработал, второй - когда нет.

```
function testParser_one_or_zerro_0() export
	parser = new Structure;
	RuleDef(parser,"start",zerroOrOne(ichar("a"),-1));
	value = "abA";
	result = runParser(value,"start",parser);
	pos= new Structure("position,line,column",2,1,2);
	wait = makeSuccess(new Structure("value,position","a",pos),(pos));
	assert.What(result).Equal(wait);
endfunction

function testParser_one_or_zerro_1() export
	parser = new Structure;
	RuleDef(parser,"start",zerroOrOne(ichar("a"),-1));
	value = "2abA";
	result = runParser(value,"start",parser);
	pos= new Structure("position,line,column",1,1,1);
	wait = makeSuccess(-1,(pos));
	assert.What(result).Equal(wait);
endfunction
```

Обратите внимание на вызов zerroOrOne - вторым паарметром передается значение которое будет возвращено если персер - не сработал.
Код парсера 

```
function applyParser_oneOrZerro(where,fparser,parser) export
	fParserRef = fparser.parserRef;
	wrkData  = ApplyParser(where,fParserRef,parser);
	if isSuccess(wrkData) then
		return wrkData;
	endif;
	return makeSuccess(fparser.default,getRest(wrkData));
endfunction

```

Как вы уже поняли на этих трех примерах - писать операторы не сложно. Оставщиеся операторы - остаются для самостоятельной разработки, или (если спешите) можете посмотреть в code\CollectionParser.epf

