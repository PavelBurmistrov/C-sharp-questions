### Что такое CLR? Какие функции она выполняет?
- *CLR (Common Language Runtime)* - общеязыковая среда исполнения для IL (=CIL/MSIL) кода, в который компилируются программы написанные на .NET-совместимых ЯП: *C#, C++, VB, F#,  Iron Python, Iron Ruby, IL* и тд. 

*CLR* компилирует код приложения на языке *IL* во время его исполнения *+* предоставляет доступ к *.NET FCL (Framework Class Library)*.

Объекты, написанные на разных языках, могут взаимодействовать друг с другом. Например, разработчик может определить класс, а затем на другом языке создать производный от него класс или вызвать метод из исходного класса. 
- *Основные функции *CLR*:* 
 1. Загрузка сборок    
 2. Управление памятью 
 3. Интеграция языков   
 4. Безопасность (верификация) 
 5. Обработка исключений
 6. Повышение производительности

### Загрузка CLR. Если мы запускаем 2 или более программы на .NET - сколько экземпляров CLR у нас будет?
- *Загрузка CLR:*

1. **> К**омпилятор анализирует и проверяет исходный код. Генерирует управляемый модуль [PE32/PE32+ заголовок, CLR заголовок, метаданные, IL код] 
2. **> М**одули объединяются в сборку (для C# это CSC.exe) 
3. **> В** зависимости от указанной целевой платформы C# генерирует заголовок — PE32 или PE32+, а также включает в него требуемую процессорную архитектуру (или признак независимости от архитектуры) 
4. **> W**indows анализирует заголовок EXE-файла для определения того, какое именно адресное пространство необходимо для его работы — 32- или 64-разрядное 
5. **> W**inda загружает в адр. пространство соотв. версию библиотеки MSCorEE.dll 
6. **> О**сновной поток вызывает в MSCorEE.dll метод, который инициализирует CLR, загружает сборку EXE, и вызывает ее метод Main, в котором содержится точка входа 
7. **> З**агрузка CLR завершена.


- *Если мы запускаем 2 или более программы на .NET - сколько экземпляров CLR у нас будет?*

Столько экземпляров сколько и приложений. JIT-компилятор хранит машинные команды в динамической памяти. Это значит, что скомпилированный код уничтожается по завершении работы приложения. Для повторного вызова приложения или для параллельного запуска его второго экземпляра (в другом процессе операционной системы) JIT-компилятору придется заново скомпилировать IL-код в машинные команды. 

### JIT компилятор - что это такое и для чего он нужен?
Для выполнения какого-либо метода его IL-код должен быть преобразован в
машинные команды. Этим занимается *JIT-компилятор (Just-In-Time)*.
Когда метод Main первый раз обращается к вызываемому методу в коде, вызывается
функция *JITCompiler*. Она компилит IL код в машинный команды. 
<image src="https://user-images.githubusercontent.com/49167504/199069040-b2a58e3f-be01-403b-a321-6e365c5532ec.png" width="500">
 
### Быстродействие IL->EXE - это медленее чем обычный EXE? Или есть нюансы? Когда наблюдается снижение производительности, а когда нет?
 - *Быстродействие IL->EXE - это медленее чем обычный EXE?*
 
  В тот момент, когда JIT-компилятор компилирует IL-код в машинный код во время выполнения, он знает о среде выполнения больше, чем может знать неуправляемый компилятор.
И вот, почему IL->EXE может быть быстрее обычного EXE:
1. JIT модет знать о процессоре. Неуправляемый код использует обычно просто общий набор команд, и это не повышает эффективность.
2. Если компьютер оснащен всего одним процессором, то JIT не будет генерировать машинные команды для указанного фрагмента.
```
 if (numberOfCPUs > 1) {
 ...
}
 ```
 3. Можно использовать утилиту Ngen.exe. Эта утилита компилирует весь IL-код сборки в машинный код и сохраняет его в файле на диске. При загрузке CLR проверяет сама наличие этой утилиты, если она есть, то юзает её.
 - *Когда наблюдается снижение производительности, а когда нет?*
 
 Снижение только при первом вызове метода. JIT-компилятор оптимизирует машинный код. И опять же: создание оптимизированного кода занимает больше времени, но при выполнении он гораздо производительнее, чем неоптимизированный.
 
### Что такое PDB файлы? Для чего они нужны?
 
Компилятор строит файл *PDB (Program Database)* только при **отладке**. Файл PDB помогает отладчику находить локальные переменные и связывать команды IL с исходным кодом. JIT сохраняет инфу о том, какой машинный код был сгенерирован для каждой команды IL.
 
### Безопасный и небезопасный код. Почему считается, что приложения на .NET являются безопасными?
 - *Безопасный и небезопасный код*
 
 По умолчанию компилятор C# компании Microsoft генерирует *безопасный код* - код, безопасность которого подтверждается в процессе верификации. Но можно писать и *небезопасный код* (unsafe{..}), и напрямую работать с адресами памяти. Небезопасный код - это риск.
 - *Почему приложения на .NET - безопасные?*
 
 В процессе компиляции IL в машинные инструкции CLR *верификация*— анализ кода IL и проверка безопасности всех операций. Например, верификация убеждается в том, что каждый метод вызывается с правильным количеством параметров, что все передаваемые параметры имеют правильный тип и тд. Вся информация о методах и типах, используемая в процессе верификации, хранится в метаданных. Верификация управляемого кода гарантирует, что код не будет некорректно обращаться к памяти и не сможет повредить выполнению кода другого приложения.
 
### CLS/CTS - важно понимать, что это такое и уметь ответить
 
 *CLS (Common Language Specification)* - определяет минимальный набор возможностей, которые должны поддерживаться всеми *CLS*-совместимыми языками. *CLS* определяет правила, которым должны соответствовать типы и методы с внешней видимостью, для того чтобы они могли использоваться в любом *CLS*-совместимом языке программирования.
 
 *CTS (Common Type System)* - описывает способ определения и поведение типов. *CTS* определяет набор правил, которыму должны следовать все языки при работе с типами: содерж. поле, метод, свойство, событие; задает правила видимости типов; задает правила наследования; срок жизни объектов и тд. Синтаксис разный, но поведение одно. Ещё одно правило *CTS* - все типы являются производными от типа System.Object, т.е у каждого тиа есть мин. набор аспектов поведения.
 
<image src="https://user-images.githubusercontent.com/49167504/199208075-68b9103b-467e-4ac3-925f-0bbb7fdf6159.png" width="460"> <img src="https://user-images.githubusercontent.com/49167504/199209093-2f11b665-e10e-4de1-9f64-26bab0dfafd1.png" width="540">

### Обычные сборки и сборки со строгим именем. Что это такое? В чем отличия? Как идентифицируются сборки со строгим именем?
 
 Среда CLR поддерживает два вида сборок: **обычные** и со **строгими именами** *(strongly named assemblies)*. Они имеют одинаковую структуру: PE32/PE32+, CLR заголовок, метаданные, таблица манифеста, IL. 
 
 Отличаются тем, что "строгие" сборки подписаны при помощи пары ключей, уникально идентифицирующей издателя сборки. Эта пара ключей позволяет уникально идентифицировать сборку, обеспечивать ее безопасность, управлять ее версиями, а также развертывать в любом месте пользовательского жесткого диска или даже в Интернете.
 | Тип сборки                | Закрытое развертывание | Глобальное развертывание |
 |---------------------------|------------------------|--------------------------|
 | Сборка с нестрогим именем |           Да           |          Нет             |
 | Сборка со строгим именем  |           Да           |          Да              |
 
 У сборки со **строгим именем** четыре атрибута, уникально ее идентифицирующих: имя файла (без расширения), номер версии, идентификатор регионального стандарта и открытый ключ. Компания, желающая снабдить свои сборки уникальной меткой, должна получить пару ключей — открытый и закрытый, после чего открытый ключ можно будет связать со сборкой. 
 
 Первый этап создания такой сборки — получение ключа при помощи утилиты Strong Name (SN.exe). Создается файл, содержащий открытый и закрытый ключи в двоичном формате.
 При компиляции сборки с параметром /keyfile:[файл с парой ключей].snk, компилятор C# подписывает сборку закрытым ключом и встраивает открытый ключ в манифест сборки.
 <img src="https://user-images.githubusercontent.com/49167504/199287551-ce68c411-4ad1-4154-ba70-1618fac87222.png" width="550">
 
 Подписание файла закрытым ключом и внедрение подписи и открытого ключа в сборку позволяет CLR убедиться в том, что сборка не была модифицирована или повреждена. При установке сборки в GAC система хеширует содержимое файла с манифестом и сравнивает полученное значение с цифровой подписью RSA, встроенной в PE-файл (после извлечения подписи с помощью открытого ключа). Идентичность значений означает, что содержимое файла не было модифицировано. 
 
### Что такое GAC? Как он работает?

*GAC - (global assembly cache)* - глобальный кэш сборок, место, где располагаются совместно используемые сборки. Обычно *GAC* находится в каталоге %SystemRoot%\Microsoft.NET\Assembly. *GAC* имеет иерархическое строение и содержит множество вложенных каталогов, имена которых генерируются по определенному алгоритму. Ни в коем случае не следует копировать файлы сборок в *GAC* вручную — вместо этого надо использовать инструменты. 
 
### Какие действия выполняет оператор new в C#
 *new Выполняет:* 
 1. Считает кол-во байтов для хранения !экземплярных! полей типа и базового + байты для индекса блока синхронизации и указателя на объект-тип
 2. Выделяет память в куче
 3. Инициализация блока синхронизации(вроде -1) и указателя на объект-тип
 4. Вызывает конструктор с параметрами или по умолчанию..
 5. Возвращает указатель на место в памяти, где лежит созданный объект
 
### Приведение типов: явное, is, as. Отличия
 **is** возвращает TRUE либо FALSE 
 
 **as** возвращает ссылку на объект или NULL (Оба не генерируют исключения)
 
 Пример:
 
``` 
 Employee e = new Manager();
 
 if (Employee is Manager) // тут is приводит тип и проверяет на совместимость
 
 { Manager m = (Manager) e } // тут еще раз приводим тип, и тоже проверяем является ли e ссылкой на Manager
 
 Manager m = e as Manager; // если всё ок, то as возвращает указатель на этот объект, в противном случае NULL
 
 if (e != null) {...} // проверим на е налл и можем юзать
 ```
 
### Как работают статические, экзэмплярные и виртуальные методы в CLR (объяснение на основе кучи + object type + objects)
 
 При вызове **статического** метода: CLR находит объект-тип к которому отностится вызвший метод тип -> находит точку входа в метод -> обрабатывается JIT-компилятором (если в первый раз) и передает управление машинному коду.
 
 При вызове **экземплярного** метода: CLR находит объект-тип соответствующий типу вызвавшей этот метод переменной или идет в базовый объект-тип и там ищет метод... В каждом объекте-типе есть поле, содержащее ссылку на его базовый тип, конечная точка - это System.Type? - он сам на себя указывает.  
 
 CLR следует по адресу вызывающего объекта -> проверяет указатель на объект-тип -> находит в таблице методов объекта-типа запись вызова метода и (при первом вызове JIT-компиляция нужна) вызвает полученный машинный код. Потом вызывает по иерархии классов метод, который был переопределен, тот просто возвращает управление.
 
### Что такое стек? Как он работает? 
  
 **Стек** *(LIFO)* хранит значимые типы и ссылки на объекты в куче - 1МБ фиксированный размер, возможео переполнение стека. Указатель стека при выделении в нем памяти смещается в сторону уменьшения адресов. 
 
#### Что такое куча? Как она работает?
 
В **куче** хранятся ссылочные типы данных - объекты, объект-типы, упакованные значимые типы. Это динамическая область памяти, объекты оттуда удаляются сборщиком мусора. При закрытии программы память освобождается.
 
### Сколько может быть стеков и куч в выполняемой программе? Почему именно так?
 
 Стеков столько, сколько потоков.
 
 Куча одна и как раз поэтому созданы инструменты синхронизации, ибо потоки имеют общий доступ к объектам в куче. 
 
### Примитивные + значимые типы, примеры, где они хранятся
 
 *Примитивные* типы данных - те, о которых знает компилятор C# (Int16, Int32, Int64, UInt16, UInt32, Uint64, char, Byte, SByte, Single, Double, Decimal)
 
 Еще значимых: структуры, перечисления(enum)
 
 Значимые типы хранятся в стеке. Стек создается размером 1 МБ
 
 При присвоении значимые типы копируются. Память освобождается сразу при возвращении управления методом.
 
### Ссылочные типы, примеры, где они хранятся
 
 Ссылочные типы хранятся в куче (ссылка хранится в стеке, сам объект в куче) - это классы, интерфейсы, string, все ссылочные наследованы от типа object.
 
 При присвоении копируется ссылка на объект в куче. Память очищается сборщиком мусоро, когда ссылка перестаёт указывать на объект в куче, кроме слабых ссылок, например Блока синхронизации.
 
### Что такое упаковка/распаковка типов
``` 
 Struct A {}
 
 A a;
 
 object b = a;  // Упаковка
 
 A c = (A)b;    // Распаковка
 ```
 **Упаковка** - преобразование значимого типа в ссылочный. **Распаковка** - извлечение адреса полей (+ копирование).
 
 *При упаковке:* выделяется память в куче для полей объекта + указатель на объект-тип и индекс блока синхронизации. Копируются поля объекта в кучу и возвращается указатель на эти поля.
 
 *При распаковке:* получаем адрес на поля в куче и копируем в переменную. 
 
### Хэш коды. Почему важно переопределять Equal + GetHashCode в классах? В каких случаях эти методы надо переопределять?
 
 Два равных объекта должны иметь одинаковые хеш-коды. Например, хеш-таблици, Dictionary требуют выполнения этого условия. 
 
 Переопределяя Equals надо переопределить и GetHashCode, и чтобы алгоритм для вычисления равенства соответствовал алгоритму вычисления хеш-кода.
 
### Что такое статические классы? Особенности их работы
 
 **Статические классы** - CLR определеят как абстрактный запечатанный (abstract + sealed class). 
 
 Нельзя создавать экземпляры статического класса. Не применимо наследование. Статические поля, и методы хранятся в объекте-типе.
 
 
### Производительность виртуальных и невиртуальных методов - что быстрее и почему?
 
 Быстрее *невиртуальный* метод. Для вызова *виртуального*, экземплярного в IL коде компилируется команда *callvirt*, которая еще и проверяет вызванную переменную на NULL. *call* работает с виртуальными, экземплярными, статическими методами - работает быстрее, т.к. без проверки. 
 
### Конструкторы значимых типов и статических типов - особенности и отличия
 
 Конструкторы значимых типов надо вызывать явно.
 
### Обязательные и необязательнве параметры методов
 
 Необязательные параметры методов - те, которые инициализированы уже. Указываются справа от обязательных. *void Method(int a, int b = 10){}*
 
### Как работает params для параметров методов?
 
 Массив параметров, можно передавать переменное количество параметров. Если использовать тип object, то можно передавать значения разных типов. 
 
 Пример: *void Method(char c, params int[] numbers){}*
 
### Передача параметров по ссылке и по значению в метод - в чем отличия?
 
 При передаче параметров по значению само значение копируется в метод, если оно изменятся в методе, то оригинал остается прежним. 
 
 При передче параметров по ссылке, передается ссылка на переменную - работаем с оригиналом. Если мы передаем в качестве параметра переменную типа class e, то у нас просто копируется адрес на объект в куче -> если в методе сделаем e = new(); то будем работать с другим объектом. Та переменная останется неизменной(оригинал). 
 
### Что такое ref
 
 Передача параметра по ссылке: значимых и ссылочных. Указатель на переменную. *void Method(ref int a, ref Employee e)*
 
### Что такое out?
 
 То же что и *ref*, только мы должны инициализировать внутри метода обязательно переменную. *void Method(out int a, out Employee e)*
 
### Вызов *виртуальных методов* - как это работает внутри CLR?
 
 CLR следует по адресу вызывающего объекта -> проверяет указатель на объект-тип -> находит в таблице методов объекта-типа запись вызова метода и (при первом вызове JIT-компиляция нужна) вызвает полученный машинный код. Потом вызывает по иерархии классов метод, который был переопределен, тот просто возвращает управление.
 
### Что такое **Singleton** паттерн и как его реализовать
 
 Паттерн, который позволяет создать только один экземпляр класса. Конструктор делаем приватным, а в свойстве можно создать объект, там же и установать синхронизатор для потоков, чтоб они не смогли создать несколько экземпляров.
 ```
 public class Singleton
    {
        private static Singleton _instance = null!;
        private static object _objSync = new object();

        private Singleton()
        {
        }

        public static Singleton Instance
        {
            get
            {
                if (_instance == null)
                {
                    lock (_objSync)
                    {
                        if (_instance == null)
                        {
                            _instance = new Singleton();
                        }
                    }
                }
                return _instance;
            }
        }
    }
 ```
### Оператор **lock** синхронизации доступа в .NET, как он работает, связать его с блоком индекса синхронизации
 
 lock представляет из себя *try {} finally {}*
 
 lock: вызывает метод *Monitor.Enter(object _syncObject, ref bool lockTaken)* - он проверяет аргумент *lockTaken* - 
 
 если *lockTaken == True*, т.е. *lock* был взят, то вызываем метод, который генерирует исключение ArgumentException и нас кидает в блок Finally. 
 
 если *lockTaken == False*, то вызывается метод *ReliableEnter(object _syncObject, ref bool lockTaken)* - что-то делает и *lockTaken* присваивает True. 
 
 Выполняем всё, что нам надо . . .
 
 Заходим в блок *Finally {...}* 
 
 Если у нас *lockTaken == True*, то вызывается метод *Monitor.Exit(object, bool&)* - освобождает объект и делает *lockTaken = False*.
 
 Если же *lockTaken == False*, просто выходим с неудачей из блока.
 
 При инициализации CLR выделяет массив блоков синхронизации в куче. **Индекс блока синхронизации** - индекс в массиве этих блоков. При начальной инициализации ИБС = -1.
 
 При вызове *Monitor.Enter()* CLR обнаруживает в массиве свободный блок синхронизации и присваивает ссылку на него объекту(привязка объекта к БС). 
 
 *Monitor.Exit()* проверяет наличие потоков, ожидающих блока синхронизации. Если их нет(залоченный объект больше никому не нужен из потоков), метод возвращает индексу -1 - это означает, что БС свободен и может быть связан с другим объектом. 
 
 Блок синхронизации содержит слабую ссылку на объект, владеющий блоком и ссылку на Monitor.
