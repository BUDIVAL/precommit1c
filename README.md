## Набор утилит для автоматической разборки/сборки внешних обработок и отчетов, при помещении(commit) в git.

### Что к чему
----
* pyv8unpack.py - python скрипт, получающий список помещаемых файлов при коммите, фильтрующий по расширению только внешние обработки/отчеты и запускающий внешнюю обработку для распаковки этих файлов. 
* [V8Reader.epf](http://infostart.ru/public/106310/) - внешняя обработка 1С, которая с помощью [v8unpack](http://svn2.assembla.com/svn/V8Unpack/track/) разбирает внешние обработки, определяет нормальные наименования для каталогов форм, файлов модулей объектов и т.д. и раскладывает их в нормальную структуру папок. 
* ibService - сервисная база данных на 1С, для запуска V8Reader.epf
* pre-commit - собственно командный файл вызываемый git перед каждым помещением. Выполняет роль простой запускалки скрипта pyv8unpack.py 

### Установка

1. Зависимости: 
    * python 3.3
    * установленная платформа 1С предприятия. 
    * git
    * в случаии запуска из под wine, необходим и msscriptcontrol.

2. По умолчанию считается, что пути к python.exe и git.exe находятса в переменной path, иначе необходимо указать явный путь в файлах pre-commit(для python) и pyv8unpack.py(для git)

3. Путь к платформе находит автоматически, в случаии стандатной установки 1С. Если необходимо указать явно путь к платформе, необходимо: Указать переменную окружения PATH1C c путем к каталогу, где установленна 1С
```
set PATH1C = d:\program\
```
или создать файл ini рядом с файлом скрипта pyv8unpack.py или в домашней папке в корне, с именем precommit1c.ini и содеражнием:
```
[DEFAULT]
onecplatfrorms = c:\program\1cv8\8.3.5.823\bin\1cv8.exe
```

4. Путь хранения исходных текстово разобранных обработок поумолчанию используется как **src** (для обеспечения совместимости со старыми версиями обработки), однако его можно переназначить в ini файле
```
[DEFAULT]
source = plugin_source
```

5. Наконец содержимое каталога необходимо скопировать в каталог .git/hooks/ вашего проекта. 
> *Примечание:* каталог .git по умолчанию скрыт.  

```
.git\
    hooks\
        pre-commit
        V8Reader.epf
        ibService 
        pyv8unpack.py
```

##Запуск 

После установки достаточно для проверки сделать commit для любого файла epf и в вашем репозитарии автоматически должна создаться папка *src* повторяющая полностью структуру проекта, те файлы которые были измененны или же добавленны распакуются в папки с аналогичным наименованием. 

##Ограничения

Одинковыми именами файлы с разным расширением epf и erf называть не надо, т.к. каталоги с исходниками создаются только по наименованию без учета расширения и возможен конфликт. 
Дополнительно необходима настройка git для возможности использования кирилических наименований внешних обработок ```git config --local core.quotepath false```
##Что внутри

как это работает: pyv8unpack.py повторяет полностью иерархию папок относительно корня репозитария только в папке SRC (от слова source) или ту которую вы определили в конфигурационном файлу , каждая для каждой измененной внешней обработки создается своя папка и туда с помощью v8unpack распаковывается помещаемая обработка, с помощью v8reader определяютса наименования макетов, форм, модуля обработки и переименовываются, переименования сохраняютса в служебном файле renames.txt , те файлы, которые невозмонжно определить или же носят чисто служебный характер, переносятса в каталог *und*
