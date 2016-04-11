# Необходимая теория

## Примитивная реализация

В данном контексте слово "примитивная" используется для того что бы показать возможность автоматической генерации и анализа парсеров. 
Все будем делать руками, для того что бы понять как дальше будет работать автоматическая генерация. 

Начнем с того что напишем простой тест


```
function testPlus()
    result = parse("1+2");
    assert.What(result).Equal(3)
endfunction
```

и приступим к реализации функции parse. Схематично эта функция выглядит так

```
function parse(textString)
    curPos = 1;
    
    firstValue = readNumber(textString,curPos);
    if firstValue = undefined then
        return undefiend;
    endif;
    plus = readPlus(textString,curPos);
    if plus = undefined then
        return undefiend;
    endif;
    secondValue = readNumber(textString,curPos)
    if secondValue = undefiend then
        return undefined;
    endif;
    return firstValue+secondValue;
endfunction
```

Уже сейчас понятно - нам потребуются функции readNumber и readPlus, которые будут возвращать прочитанное значение (если прочитать его получилось) или Неопределено (если читаемый символ не соответствует ожиданиям). Простая реализация для readPlus может выглядеть примерно так

```
function readPlus(textString,curPos)
    value = mid(tesxString,curPos,1);
    if value <> "+" then 
        return Undefined;
    endif;
    curPos = curpos + 1;
    return value;
endif;
```

Функция readPlus просто получает символ по указаной позиции и анализирует его. 
Если это не символ '+' тогда возвращает undefined, иначе - увеличивает указатель позиции на 1 и возвращает прочитанный символ. 
Обработка ошибок *умышленно* не добавлена на этом этапе, что бы не загромождать код ненужными пока частностями. Чуть позже мы обязательно и ее добавим.


Реализация readNumber чуть сложнее
```
function readNumber(textString,curPos)
    valueNumber = "";
    while true do
        value = mid(textString,curPos,1);
        if value < "0" or value > "9" then
            break;
        endif;
        valueNumber = valueNumber + 1;
        curPos = curPos + 1;
    enddo;
    if valueNumber = "" then
        return undefined;
    endif;
    return number(readValue);
endfunction
```
В цикле аккумулируется строковое значение, до тех пор пока прочитанный символ - цифра. 
Возвращаемое значение зависит от прочитанного набор - если он пуст - то возвращаем undefiend, иначе - прочитанное число преобразованное к типу число. Ошибки преобразования быть не может, так как накопленное значение - всегда последовательность цифр, а они приводятся к числу встроеной функцией число


В принципе - парсер рабочий. Но функция readNumber - немного корява. Давайте введем еще одну функцию, аналог readPlus - readDigit, которая будет отвечать за чтение одного символа и его анализ 

```
function readDigit(textString,curPos)
    value = mid(textString,curPos,1);
    if value < "0" or value > "9" then
        return undefined;
    endif;
    curPos = curPos + 1;
    return value;
endfunction
    
function readNumber(textString,curPos)
    valueNumber = "";
    while true do
        value = readDigit(textString,curPos);
        if value = undefined then
            break;
        endif;
        valueNumber = valueNumber + value;
    enddo;
    if valueNumber = "" then
        return undefined;
    endif;
    return number(readValue);
endfunction
```

Уже сейчас заметно что наши функции начинают вырабатывать собственный интерфейс - возвращать Неопределено в случае ошибки, или прочитанное значение. 
Так же потихоньку вырисовывается и шаблон по которому пишутся функции. Одно "но" омрачает - пока функции readPlus и readDigit все же сильно различаются - все дело в проверке. 
Но если немного подумать, но исходную грамматику можно переписать так - 

```
    Exp = digit '+' digit 
    digit = digitChar+ 
    digitChar = '0' | '1' |'2' |'3' |'4' |'5' |'6' |'7' |'8' |'9'   
```

И привести наш код примерно к такому виду (реализация функции intoMap - примитивная, и не будет показана)

```
function matchChar(textString,curPos,value)
    analized = mid(textString,curPos,1);
    if typeof(value) = type("map") then
        result = value[analized] <> undefined;
    else
        result = value = analized;
    endif;
    if not result then
        return undefined;
    endif;
    curPos = curPos + 1;
    retunr analized;
endfunction

function readPlus(textString,curPos)
    return matchChar(textString,curPos,"+");
endfunction

function readDigit(textString,curPos)
    return = matchChar(textString,curPos,intoMap('0','1','2','3','4','5','6','7','8','9'));
endfunction
    
function readNumber(textString,curPos)
    valueNumber = "";
    while true do
        value = readDigit(textString,curPos);
        if value = undefined then
            break;
        endif;
        valueNumber = valueNumber + 1;
    enddo;
    if valueNumber = "" then
        return undefined;
    endif;
    return number(readValue);
endfunction
```

Для чего мы это сделали? Все просто - мы просто пытаемся поделить наши парсеры на два типа - те что работают со входным потоком (readDigit, readPlus ) и те, 
что работают с результатами парсеров. В теории прасеры первого типа называются терминальными, второго - нетерминальными. 

Многие скажут что код стал запутаннее. Согласен с этим, и даже больше -  источником запутывания является функция readNumber - она  выбивается из логики работы (в ней нет определения парсера, есть только реализация). исправим это. Давайте предположим что у нас есть функция OneOrMore которая принимает в себя парсер, и возвращает массив результата работы вложенных парсеров, либо undefined - если парсер не сработал. 


```
function oneOrMore(textString,curPos,parser)
    result = new Array;
    while true do 
        parserResult = eval(parser);
        if parserResult = undefined then
            break;
        endif;
        result.add(parserResult);
    enddo;
    if result.count() = 0 then
        return undefined;
    endif;
    result result;
endfunction

function readNumber(textString,curPos)
    valueNumber = oneOrMore(textString,curPos,"readDigit(textString,curPos)");
    if valueNumber = undefined then
        return undefined;
    endif;
    for each x in ValueNumber do
        result = result + x;
    enddo;
    return number(readValue);
endfunction
```

К сожалению, а может и к счастью, 1С не поддерживает функции как fco, поэтому нам пришлось прибегнуть к использованию функции Вычислить(). 
Но так или иначе - своего мы добились - разделение на парсеры произведено. В следующем разделе мы разработаем набор функций и структур для формирования парсеров, позволяющий единообразно задавать как терминальные парсеры, так и нетерминальные, определим интерфейс вызова парсера и необходимые служебные переменные и функции. 


## Простые парсеры

Прежде чем перейти к продолжению - определимся что же в итоге мы хотим получить? Кроме декларированной в самом начале цели генерации текста программы, нам сейчас потребуется некое внутренне представление. Его объявим следующим образом (для грамматики со сложением)

```
    ruleDef(parser,"Exp",fn(seq("digit",cChar("+"),"digit"),"$$ = $1 + $3"));
    ruleDef(parser, "digit", fn(oneOreMore(range("0","9")),"result = """"; for each x in $1 do result = result + x.value; enddo; $$ = number(result)"))
```

Фактически это и есть наша грамматика, просто записана немножко по другому. Понимая к чему стремимся, осталось определить только используемые функции. 

    //терминальные парсеры
    cChar - парсер для 1 символа. Сравнение идет с учетом регистра символа
    iChar - парсер для 1 символа. Сравнение идет без учета регистра символа
    cString - парсер для строки. Сравнение идет с учетом регистра символов
    iStrin - парсер для строки. Сравнение идет без учета регистра символов
    range  - парсер для интервала, задается минимальное и максимальное значение элементов множества.
    code - парсер реализует сравнение символа и кода символа
    eof - парсер проверяет конец потока
    any - парсер безусловного чтения 1 символа
    // парсеры предпросмотра
    negativeLookahead - парсер предпросмотра с инверсией
    positiveLookahead  - парсер предпросмотра
    // нетерминальные парсеры
    seq - парсер собирающий несколько парсеров в один
    alt - парсер альтернатив
    zerroOrMore - парсер *
    zerroOrOne - парсер ?
    oneOrMore - парсер +
    fn - парсер, выполняющий код в случае если зависимый парсер успешно отработал


Также потребуется  зафиксировать интерфейс функций и используемые глобальные переменными. Для того что бы проще было с этим определиться - напишем тест

```
function test_parseChar()
    parser = new Stucture;
    ruleDef(parser,"charPlus",cChar("+"));
    result = parse("+","charPlus",parser);
    assert.what(result.value).equal("+");
    assert.what(result.position).equal(2);
    assert.what(result.type).equal(1);
    
endfunction
```

Функция ruleDef - это просто определение нового ключа в структуре. Код простой 

```
function ruleDef(parserCollection,name,parser)
    parserCollection.insert(name,parser);
endfunction
```

Далее все возможные парсеры будут храниться именно в коллекции parserCollection.
Простая реализация для определения парсера разбора одного символа может выглядеть так

```
function cChar(value)
    return new Structure("type,value","cChar",value);
endfunction
```

Что есть парсер в текущей постановке? Это просто структура, хранящая в себе параметры для парсера и тип парсера. Эта структура будет обрабатываться в соотвествующей функции (applyParser_cChar), которая возвращает либо признак успешного разбора и значение рабора, либо просто признак неудачи. В качестве параметра у этой функции выступает только текущая позиция в строке и  описание парсера.

В случае успешного разбора - возвращается структура с прочитанным значением, новой позицией и признаком успеха. Если парсер не смог отработать - будет возвращаться структура со старым значением позиции и признаком неуспеха. 

То есть  для парсера обрабатывающего сравнение символа возможная такая реализация. 

```
function applyParser_cChar(where,parser,parserCollection)
    value = getChar(where);
    if value = parser.value then
        return Признак успеха и значение
    endif;
    return Признак неудачи;
endfunction
```

Признак успеха и неудачи можно обернуть в структуру, на пример, так

```
function makeSuccses(value,where)
    return new Structure("value,rest,type",value,where,1);
endfunction

function makeFailure(where)
    return new Structure("rest,type",where,0);
endfunction

function applyParser_cChar(where,parser)
    value = getChar(where);
    if value = parser.value then
        return makeSucces(value,новая позиция)
    endif;
    return makeFailure(where);
endfunction

```

Теперь нам осталось только определить - откуда берется новая позиция. Так как cChar - терминальный парсер, то самое очевидное - заставить функцию getChar возвращать кроме прочитанного значения еще и новое состояние. Саму строку разбора будем храниться в глобальной переменной textString (требование не обязательное, просто для того что бы не передавать строку как параметр во все парсеры, т.е. для упрощения интерфейса ) 
Например так-

```
function getPos(value)
    if value.property("position") then
        return value.position;
    endif;
    raise "No position field";
endfunction

function getChar(where)
    position = where.position;
    value = mid(textString,position,1);
    return new Structure("value,position",value,position+1);
endfunction

function makeSuccses(value,where)
    return new Structure("value,rest,type",value,where,1);
endfunction

function makeFailure(where)
    return new Structure("rest,type",where,0);
endfunction

function getRest()
    if value.property("rest") then
        return value.rest;
    endif;
    raise "No Rest in state";
endfunction

function getValue(value) export
    if value.property("value") then
        return value.value;
    endif;
    raise "No Value in state";
endfunction



function applyParser_cChar(where,parser)
    rChar = getChar(where);
    if fparser.char = rChar.value then
        return makeSuccess(rChar,getPos(rChar));
    else
        return makeFailure(where);
    endif
endfunction

```
Более эффективно было бы все обернуть в шаблон состояние, но учитывая что работа с укаателями в 1С невозможна, то эффективной реализации не получиться. Но в качестве тренировки - я бы рекомендовал сделать вариант работы генератора и с использованием этого подхода.


Продолжаем. Возникает вопрос -  как вызвать эти функции для разбора строки? Очень просто - достаточно написать простую функцию - applyParser

```
function applyParser(where,name,parserCollection)
    workParser = unedfined;
    if typeof(name) = type("String") then
        if not parserCollection.property(name) then
            raise "Undefined parser with name '"+name+"'";
        endif;
    else
        workParser = name;
    endif;

    if workParser.type = "cChar" then
        return applyParser_cChar(where,parser);
    endif;

endfunction

function parse(inputString,parserName,parserCollection)
    textString = inputString;
    return applyParser(new Structure("position",1),parserName,parserCollection);
endfunction
```

Вот собственно и все - наш тест отработал.
Реализацию остальных терминальных парсеров можно посмотреть в коде обработки [code\SimpleParser.epf]

Переходим к реализации чуть более сложных парсеров.

## Сложные парсеры
В этой главе мы реализуем следующие парсеры 
* seq - парсер собирающий несколько парсеров в один
* alt - парсер альтернатив
* zerroOrOne - парсер разбирающий ноль или одно вхождение парсеров


Парсеры 
* zerroOrMore - парсер разбирающий ноль или больше вхождения парсеров
* oneOrMore - парсер разбирающий один или больше вхождения парсеров
* fn - парсер, выполняющий код в случае если зависимый парсер успешно отработал

остаются для самостоятельной разработки
###Seq

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


###Alt

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


### Парсер zerroOrOne (?)

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

### Где мы сейчас? 
Вспомним первончальную постановку задачи 
Нам дана граматика 
```
    Exp = digit '+' digit {$$ = $1 + $2}
    digit = digitChar+ {
	res = ""; 
	for each x in $1 do 
		res = res + x; 
	enddo; 
	
    $$ = number(res);}
    digitChar = [0-9] {$$ = $1.value}
```

Если сейчас мы перепишем ее в таком виде  

```
function testParser_Сложение() export
	parser = new Structure;
	RuleDef(parser,"Exp",fn(seq("digit",cChar("+"),"digit"),"$$ = $1 + $3"));
	RuleDef(parser,"digit",fn(seq(oneOrMore("digitChar")),"res = """"; 
	|for each x in $1 do 
	|	res = res + x; 
	|enddo; 
	|$$ = number(res);"));
	
	RuleDef(parser,"digitChar",fn(range("0","9"),"$$ = $$.value;"));
	
	result = runParser("1+2","Exp",parser);
	wait = 3;
	assert.What(result.value).Equal(wait);
	
endfunction
```
то задача будет решена. Но пользоваться такой записью - очень неудобно. В связи с чем переходим ко вторй части - реализация языка описания парсеров и его тестирвоание.