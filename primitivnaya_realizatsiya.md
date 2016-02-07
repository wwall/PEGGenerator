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
Как видно из кода - функция просто плучает символ по указаной позиции и анализирует его. Если это не символ '+' тогда возвращает undefined, иначе - увеличивает указатель позиции на 1 и возвращает прочитанный символ. 
Реализция readNumber чуть сложнее
```
function readNumber(textString,curPos)
    valueNumber = "";
    savePos = curPos;
    while true do
        value = mid(textString,curPos,1);
        if value < "0" or value > "9" then
            break;
        endif;
        valueNumber = valueNumber + 1;
        curPos = curPos + 1;
    enddo;
    if valueNumber = "" then
        curPos = savePos;
        return undefined;
    endif;
    return number(readValue);
endfunction
```

