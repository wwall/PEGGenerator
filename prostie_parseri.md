# Простые парсеры

Прежде чем перейти к продолжению - определимся что же в итоге мы хотим получить? Кроме декларированной в самом начале цели генерации текста программы, нм сейчас потребуется некое внутренне представление. Его объявим следующим образом (для грамматики со сложением)

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


Осталось определиться с интерфейсом функций и глобальными переменными. Для того что бы проще было с этим определиться - напишем тест
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

Что есть парсер в текущей постановке? Это просто структура, хранящая в себе параметры для парсера. Эта структура будет обрабатываться в соотвествующей функции (applyParser_cChar), которая возвращает либо признак успешного разбора и значение рабора, либо просто признак неудачи. В качестве параметра у этой функции выступает только текущая позиция в строке и  описание парсера.

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

Теперь нам осталось только определить - откуда берется новая позиция. Так как cChar - терминальный парсер, то самое очевидное - заставить функцию getChar возвращать кроме прочитанного значения еще и новую позицию. Саму строку разбора будем храниться в глобальной переменной textString (требование не обязательное, просто для того что бы не передавать строку как параметр во все парсеры, т.е. для упрощения интерфейса ) 
Например так-

```
function getChar(where)
    position = where.position;
    value = mid(textString,position,1);
    return new Structure("value,position",value,position+1);
endfunction

function makeSuccses(value,where)
    return new Structure("value,position,type",value,where,1);
endfunction

function makeFailure(where)
    return new Structure("position,type",where,0);
endfunction

function applyParser_cChar(where,parser)
    value = getChar(where);
    if value.value = parser.value then
        return makeSucces(value.value,value.position)
    endif;
    return makeFailure(where.position);
endfunction

```

Но как вызвать эти функции для разбора строки? Очень просто - достаточно написать простую функцию - applyParser

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
Реализацию остальных парсеров можно посмотреть в коде обработки peg1.epf (тут ссылка)

Переходим к реализации чуть более сложных парсеров.

