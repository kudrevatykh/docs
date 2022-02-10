---
sourcePath: ru/ydb/ydb-docs-core/ru/core/yql/reference/yql-docs-core-2/udf/list/yson.md
sourcePath: ru/ydb/yql/reference/yql-docs-core-2/udf/list/yson.md
---

# Yson
[YSON](https://yt.yandex-team.ru/docs/description/common/yson.html) — разработанный в Яндексе формат данных, похожий на JSON:

* Сходства с JSON:
    * не имеет строгой схемы;
    * помимо простых типов данных поддерживает словари и списки в произвольных комбинациях.
* Некоторые отличия от JSON:
    * Помимо текстового представления имеет и бинарное;
    * В текстовом представлении вместо запятых — точки с запятой, а вместо двоеточий — равно;
* Поддерживается концепция «атрибутов», то есть именованных свойств, которые могут быть присвоены узлу в дереве.

Особенности реализации и функциональность модуля:

* Наравне с YSON данный модуль поддерживает и стандартный JSON, что несколько расширяет область его применения.
* Работает с DOM представлением YSON в памяти, которое в терминах YQL передается между функциями как «ресурс» (см. [описание специальных типов данных](../../types/special.md)). Большинство функций модуля имеют семантику запроса на выполнение указанной операции с ресурсом и возвращают пустой [optional](../../types/optional.md), если операция не удалась из-за несоответствия фактического типа данных ожидаемому.
* Предоставляет несколько основных классов функций (полный список и подробное описание функций см. ниже):
    * `Yson::Parse***` — получение ресурса с DOM-объектом из сериализованных данных, все дальнейшие операции выполняются уже над полученным ресурсом;
    * `Yson::From` — получение ресурса с DOM-объектом из простых типов данных YQL или контейнеров (списков или словарей);
    * `Yson::ConvertTo***` — преобразовать ресурс к [простым типам данных](../../types/primitive.md) или [контейнерам](../../types/containers.md);
    * `Yson::Lookup***` — получение одного элемента списка или словаря с опциональным преобразованием в нужный тип данных;
    * `Yson::YPath***` — получение одного элемента дерева документа по указанному относительному пути с опциональным преобразованием в нужный тип данных;
    * `Yson::Serialize***` — получить из ресурса копию его данных, сериализованную в одном из форматов;
* Для удобства при передаче сериализованного Yson и Json в функции, ожидающие на входе ресурс с DOM-объектом, неявное преобразование через `Yson::Parse` или `Yson::ParseJson` происходит автоматически. Также в SQL синтаксисе оператор точки или квадратных скобок автоматически добавляет вызов `Yson::Lookup`. Для сериализации ресурса по-прежнему нужно вызывать `Yson::ConvertTo***` ([тикет про поддержку синтаксиса CAST](https://st.yandex-team.ru/YQL-2610)) или `Yson::Serialize***`. Таким образом, например, получения элемента "foo" словаря из колонки mycolumn типа Yson в виде строки может выглядеть так: `SELECT Yson::ConvertToString(mycolumn["foo"]) FROM mytable;` или `SELECT Yson::ConvertToString(mycolumn.foo) FROM mytable;`. В варианте с точкой можно экранировать спецсимволы по [общим правилам для индентификаторов](../../syntax/expressions.md#escape).
* Технически модуль реализован на основе снепшота кодовой базы [YT](https://wiki.yandex-team.ru/yt/) в Аркадии.

Функции модуля стоит рассматривать как «кубики», из которых можно собирать разные конструкции, например:

* `Yson::Parse*** -> Yson::Serialize***` — конвертация из одного формата в другой;
* `Yson::Parse*** -> Yson::Lookup -> Yson::Serialize***` — извлечение значения указанного поддерева в исходном дереве YSON;
* `Yson::Parse*** -> Yson::ConvertToList -> ListMap -> Yson::Lookup***` — извлечение элементов по ключу из YSON списка.

См. также примеры комбинирования YSON-функций в [tutorial](https://yql.yandex-team.ru/Tutorial/yt_17_Yson_and_Json).

**Примеры**

``` yql
$node = Json(@@
  {"abc": {"def": 123, "ghi": "привет"}}
@@);
SELECT Yson::SerializeText($node.abc) AS `yson`;
-- {"def"=123;"ghi"="\xD0\xBF\xD1\x80\xD0\xB8\xD0\xB2\xD0\xB5\xD1\x82"}
```

``` yql
$node = Yson(@@
  <a=z;x=y>[
    {abc=123; def=456};
    {abc=234; xyz=789};
  ]
@@);
$attrs = Yson::YPath($node, "/@");

SELECT
  ListMap(Yson::ConvertToList($node), ($x) -> { return Yson::LookupInt64($x, "abc") }) AS abcs,
  Yson::ConvertToStringDict($attrs) AS attrs,
  Yson::SerializePretty(Yson::Lookup($node, "7", Yson::Options(false AS Strict))) AS miss;

/*
- abcs: `[123; 234]`
- attrs: `{"a"="z";"x"="y"}`
- miss: `NULL`
*/
```

## Yson::Parse... {#ysonparse}

``` yql
Yson::Parse(Yson{Flags:AutoMap}) -> Resource<'Yson2.Node'>
Yson::ParseJson(Json{Flags:AutoMap}) -> Resource<'Yson2.Node'>
Yson::ParseJsonDecodeUtf8(Json{Flags:AutoMap}) -> Resource<'Yson2.Node'>

Yson::Parse(String{Flags:AutoMap}) -> Resource<'Yson2.Node'>? -- принимает YSON в любом формате
Yson::ParseJson(String{Flags:AutoMap}) -> Resource<'Yson2.Node'>?
Yson::ParseJsonDecodeUtf8(String{Flags:AutoMap}) -> Resource<'Yson2.Node'>?
```

Результат всех трёх функций является несериализуемым: его можно только передать на вход другой функции из библиотеки Yson, но нельзя сохранить в таблицу или вернуть на клиент в результате операции — попытка так сделать приведет к ошибке типизации. Также запрещено возвращать его за пределы [подзапросов](../../syntax/select.md): если это требуется, то надо вызвать [Yson::Serialize](#ysonserialize), а оптимизатор уберёт лишнюю сериализию и десериализацию, если материализация в конечном счёте не потребуется.

{% note info "Примечание" %}

Функция `Yson::ParseJsonDecodeUtf8` ожидает, что символы, выходящие за пределы ASCII, должны быть дополнительно заэкранированы. Подробности о правилах экранирования можно найти в [коде YT](https://a.yandex-team.ru/arc/trunk/arcadia/yt/yt/core/misc/utf8_decoder.cpp).

{% endnote %}


## Yson::From {#ysonfrom}
``` yql
Yson::From(T) -> Resource<'Yson2.Node'>
```

`Yson::From` является полиморфной функцией, преобразующей в Yson ресурс большинство примитивных типов данных и контейнеров (списки, словари, кортежи, структуры и т.п.), [пример](https://yql.yandex-team.ru/Operations/X5sdMpdg8tLNyOczX6J8qtBTJv7iItZt01ExReYM0o0=). Тип исходного объекта должен быть совместим с Yson. Например, в ключах словарей допустимы только типы `String` или `Utf8`, а вот `String?` или `Utf8?` уже нет.

## Yson::WithAttributes
``` yql
Yson::WithAttributes(Resource<'Yson2.Node'>{Flags:AutoMap}, Resource<'Yson2.Node'>{Flags:AutoMap}) -> Resource<'Yson2.Node'>?
```
Добавляет к узлу Yson (первый аргумент) атрибуты (второй аргумент). Атрибуты должны представлять из себя узел map.

## Yson::Equals

``` yql
Yson::Equals(Resource<'Yson2.Node'>{Flags:AutoMap}, Resource<'Yson2.Node'>{Flags:AutoMap}) -> Bool
```
Проверка деревьев в памяти на равенство, толерантная к исходному формату сериализации и порядку перечисления ключей в словарях.

## Yson::GetHash
``` yql
Yson::GetHash(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Uint64
```
Вычисление 64-битного хэша от дерева объектов.

## Yson::Is...
``` yql
Yson::IsEntity(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsString(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsDouble(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsUint64(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsInt64(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsBool(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsList(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
Yson::IsDict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> bool
```
Проверка, что текущий узел имеет соответствующий тип. Entity это `#`.

## Yson::GetLength
``` yql
Yson::GetLength(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Uint64?
```
Получение числа элементов в списке или словаре.

## Yson::ConvertTo... {#ysonconvertto}
``` yql
Yson::ConvertTo(Resource<'Yson2.Node'>{Flags:AutoMap}, Type<T>) -> T
Yson::ConvertToBool(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Bool?
Yson::ConvertToInt64(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Int64?
Yson::ConvertToUint64(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Uint64?
Yson::ConvertToDouble(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Double?
Yson::ConvertToString(Resource<'Yson2.Node'>{Flags:AutoMap}) -> String?
Yson::ConvertToList(Resource<'Yson2.Node'>{Flags:AutoMap}) -> List<Resource<'Yson2.Node'>>
Yson::ConvertToBoolList(Resource<'Yson2.Node'>{Flags:AutoMap}) -> List<Bool>
Yson::ConvertToInt64List(Resource<'Yson2.Node'>{Flags:AutoMap}) -> List<Int64>
Yson::ConvertToUint64List(Resource<'Yson2.Node'>{Flags:AutoMap}) -> List<Uint64>
Yson::ConvertToDoubleList(Resource<'Yson2.Node'>{Flags:AutoMap}) -> List<Double>
Yson::ConvertToStringList(Resource<'Yson2.Node'>{Flags:AutoMap}) -> List<String>
Yson::ConvertToDict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,Resource<'Yson2.Node'>>
Yson::ConvertToBoolDict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,Bool>
Yson::ConvertToInt64Dict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,Int64>
Yson::ConvertToUint64Dict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,Uint64>
Yson::ConvertToDoubleDict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,Double>
Yson::ConvertToStringDict(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,String>
```

{% note warning "Внимание" %}

Данные функции по умолчанию не делают неявного приведения типов, то есть значение в аргументе должно в точности соответствовать вызываемой функции.

{% endnote %}

`Yson::ConvertTo` является полиморфной функцией, преобразующей Yson ресурс в указанный во втором аргументе тип данных с поддержкой вложенных контейнеров (списки, словари, кортежи, структуры и т.п.), [пример](https://yql.yandex-team.ru/Operations/X5AsHC--PI3cxBmdA89P5XVtX5m6tn9P2l31MAFsqNo=).

## Yson::Contains {#ysoncontains}
``` yql
Yson::Contains(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Bool?
```
Проверяет наличие ключа в словаре. Если тип объекта map, то ищем среди ключей.
Если тип объекта список, то ключ должен быть десятичным числом - индексом в списке.


## Yson::Lookup... {#ysonlookup}
``` yql
Yson::Lookup(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Resource<'Yson2.Node'>?
Yson::LookupBool(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Bool?
Yson::LookupInt64(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Int64?
Yson::LookupUint64(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Uint64?
Yson::LookupDouble(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Double?
Yson::LookupString(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> String?
Yson::LookupDict(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Dict<String,Resource<'Yson2.Node'>>?
Yson::LookupList(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> List<Resource<'Yson2.Node'>>?
```
Перечисленные выше функции представляют собой краткую форму записи для типичного сценария использования: `Yson::YPath` — переход в словарь на один уровень с последующим извлечением значения — `Yson::ConvertTo***`. Второй аргумент для всех перечисленных функций — имя ключа в словаре (в отличие от YPath, без префикса `/`) или индекс в списке (например, `7`). Упрощают запрос и дают небольшой выигрыш в скорости работы.


## Yson::YPath {#ysonypath}
``` yql
Yson::YPath(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Resource<'Yson2.Node'>?
Yson::YPathBool(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Bool?
Yson::YPathInt64(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Int64?
Yson::YPathUint64(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Uint64?
Yson::YPathDouble(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Double?
Yson::YPathString(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> String?
Yson::YPathDict(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> Dict<String,Resource<'Yson2.Node'>>?
Yson::YPathList(Resource<'Yson2.Node'>{Flags:AutoMap}, String) -> List<Resource<'Yson2.Node'>>?
```
Позволяет по входному ресурсу и пути на языке [YPath](https://yt.yandex-team.ru/docs/description/common/ypath.html) получить ресурс, указывающий на соответствующую пути часть исходного ресурса.

## Yson::Attributes {#ysonattributes}
``` yql
Yson::Attributes(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Dict<String,Resource<'Yson2.Node'>>
```
Получение всех атрибутов узла в виде словаря.

## Yson::Serialize... {#ysonserialize}
``` yql
Yson::Serialize(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Yson -- бинарное представление
Yson::SerializeText(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Yson
Yson::SerializePretty(Resource<'Yson2.Node'>{Flags:AutoMap}) -> Yson -- чтобы увидеть именно текстовый результат, можно обернуть его в ToBytes(...)
```

## Yson::SerializeJson {#ysonserializejson}
```yql
Yson::SerializeJson(Resource<'Yson2.Node'>{Flags:AutoMap}, [Resource<'Yson2.Options'>?, SkipMapEntity:Bool?, EncodeUtf8:Bool?]) -> Json?
```

* `SkipMapEntity` отвечает за сериализацию значений в словарях, имеющих значение `#`. На значение атрибутов флаг не влияет. По умолчанию `false`.
* `EncodeUtf8` отвечает за экранирование символов, выходящих за пределы ASCII. Детали о правилах экранирования можно найти в [коде YT](https://a.yandex-team.ru/arc/trunk/arcadia/yt/yt/core/misc/utf8_decoder.cpp). По умолчанию `false`.

Типы данных `Yson` и `Json`, возвращаемые функциями сериализации, представляет собой частный случай строки, про которую известно, что в ней находятся данные в соответствующем формате (Yson/Json).

## Yson::Options {#ysonoptions}
``` yql
Yson::Options([AutoConvert:Bool?, Strict:Bool?]) -> Resource<'Yson2.Options'>
```
Передаётся последним опциональным аргументом (который для краткости не указан) в методы `Parse...`, `ConvertTo...`, `Contains`, `Lookup...` и `YPath...`, которые принимают результат вызова `Yson::Options`. По умолчанию все поля `Yson::Options` выключены (false), а при включении (true) модифицируют поведение следующим образом:

* **AutoConvert** — если переданное в Yson значение не в точности соответствует типу данных результата, то значение будет по возможности сконвертировано. Например, `Yson::ConvertToInt64` в этом режиме будет делать Int64 даже из чисел типа Double.
* **Strict** — по умолчанию все функции из библиотеки Yson возвращают ошибку в случае проблем в ходе выполнения запроса (например, попытка парсинга строки не являющейся Yson/Json, или попытка поиска по ключу в скалярном типе, или запрошено преобразование в несовместимый тип данных, и т.п.), а если отключить строгий режим, то вместо ошибки будет возвращаться `NULL`.

**Пример:**
``` yql
$x = Yson(@@{y = true}@@);
SELECT Yson::LookupBool($x, "z"); --- Ошибка
SELECT Yson::LookupBool($x, "z", Yson::Options()); --- null (по-умолчанию все поля false)
SELECT Yson::LookupBool($x, "z", Yson::Options(false as Strict)); --- null
SELECT Yson::LookupBool($x, "z", Yson::Options(true as Strict)); --- Ошибка

$x = Yson(@@{y = 5.5}@@);
SELECT Yson::LookupInt64($x, "y") --- Ошибка
SELECT Yson::LookupInt64($x, "y", Yson::Options(false as AutoConvert)); --- null
SELECT Yson::LookupInt64($x, "y", Yson::Options(true as AutoConvert)); --- 5
```

Если во всём запросе требуется применять одинаковые значения настроек библиотеки Yson, то удобнее воспользоваться [PRAGMA yson.AutoConvert;](../../syntax/pragma.md#yson.autoconvert) и/или [PRAGMA yson.Strict;](../../syntax/pragma.md#yson.strict). Также эти `PRAGMA` являются единственным способом повлиять на неявные вызовы библиотеки Yson, которые возникают при работе с типами данных Yson/Json.