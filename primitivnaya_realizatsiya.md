# Примитивная реализация

Сразу оговорюсь - "примитивная" не значит неэффективна. В данном контексте слово "примитивная" используется для того что бы показать невозмоность автоматической генерации и анализа парсеров. Все будем делать руками. Начнем с того что напишем простой тест


```
function testPlus()
    result = parse("1+1");
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

Как видно из текста - нам потребуются функции readNumber и readPlus, которые будут возвращать прочитаное значение (если прочитать его получилось) или Неопределено (если читаемый символ не соответствует ожиданиям). Простая реализация для readPlus может выглядеть примерно так

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
Как видно из кода - функция просто плучает символ по указаной позиции и анализирует его. Если это не символ '+' тогда возвращает undefined, иначе - увеличивает указатель позиции на 1 и возвращает прочитанный символ. Обработка ошибок умышленно не добавлена на этом этапе, что бы не загромождать код ненужными пока частностями. Чуть позже мы обязательно и ее добавим.
Реализция readNumber чуть сложнее
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
Как видно из кода - в цикле аккумулируется строковое значение, до тех пор пока прочитанный символ - цифра. Возвращаемое значение завивит от прочитаного набор - если он пуст - то возвращаем undefiend, иначе - прочитаное число преобразованное к типу число.


В принципе - парсер рабочий. Но функция readNumber - немного корява. Давайте введем еще одну функцию, аналог readPlus - readDigit, которая будет отвечать чза чтение одного символа и его анализ 


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
        valueNumber = valueNumber + 1;
    enddo;
    if valueNumber = "" then
        return undefined;
    endif;
    return number(readValue);
endfunction
```

Как становится видно - наши функции начинают вырабатывать собственный интерфейс - возвращать неопределено в случае ошибки, или прочитанное значение. Так же потихньку выресовывается и шаблон по которому пишутся функции.Оно но омрачает - пока функции readPlus и readDigit все же сильно разлчаются - все дело в проверке. Но если немного подумать, но исходную грамматику можно переписать так - 

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
Как видим - мы пытаемся поделить наши парсеры на два типа - те что работают со входным потоком (readDigit, readPlus ) и те, что работают с резалтатми парсеров. В теории прасеры первого типа называются терминальными, второго - нетерминальными.  
Код стал запутанее. И источником запутывания является функция readNumber - он немного выбивается из логики работы (в ней нет определния парсера, есть только реализация). исправим это. Давайте предположим что у нас есть функция OneOrMore которая принимает в себя парсер, и взвращает массив результата работы вложеных парсеров, либо undefined - если первый парсер не сработал. 


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

К сожалению, а может и к счастью, 1С не поддерживает функции как FCO, поэтому нам пришлось прибегнуть к использованию функции Вычислить(). но так или иначе - своего мы добились - разделение на парсеры произведено. В следующем разделе мы разработаем набор функций и структур для формирвания парсеров, позволяющий единообразно задавать как терминальные парсеры, так и нетерминальные, определим интерфейс вызова парсера и необходимые служебные переменные и функции. 




