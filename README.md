# Class of organic molecule
_____
## Autor: Dvoretskov Pavel
____

Данный проект является парсером классов органических молекул из SMILE структур <br>
Стек проекта:
- python version 3.11.0

## Основные функции
____
Основная функция имеет вид:
```python
def imoprt_table(url,sheet):
```
Где 
- url -> Ссылка на гугл таблицу
- sheet -> Наименование листа на котором находятся ваши данные <br>
В предыдущей ячейке имеется переменная *first_row* задайте её при необходимости - первая строка с данными на листе таблицы

---
Дополнительная функция  
```python
def data_mine(file_name,type)
```
Отбирает для дальнейшего анализа определенный класс молекул <br>
Где
- file_name -> словарь созданный циклом:
```python 
result = {}
for i in tables:
  result[i] = imoprt_table(tables[i],'RESULTS.CSV')
```
- type -> тип молекулы для дальнейшего анализа, можно не записать, тогда анализироваться будут все имеющиеся молекулы в независимости от их класса

### Настройка заполнения словаря:
```python 
Classes_Org = {
               'c3' : 'Three_Aroma',
               'c2' : 'Bi_Aroma',
               'c1' : 'Aroma',
               'C3' : 'Three_Cyclo', 
               'C2' : 'Bi_Cyclo',
               'C1' : 'Cyclo', 
               '#': 'Alkyne',
               'c' : 'Aroma',
               '=' : 'Alkene',
               'S': 'S',
               'N' : 'N',
               '(C' : 'ISO',
               'C' : 'Alkane',
               }
```
Можно установить типы классификации молекул в рамках SMILE. <br>
Порядок в словаре указывает на порядок в заполнении списка классов конкретного соединения.

## Содержимое функции import_table
____
Импорт датасета из таблицы гугл докс
```python 
wb =gc.open_by_url(url)
ws = wb.worksheet(sheet)
```

Функция парсит сайт и выводит SMILE молекулы
```python 
def CIRconvert(ids):
    try:
        url = 'http://cactus.nci.nih.gov/chemical/structure/' + quote(ids) + '/smiles'
        ans = urlopen(url).read().decode('utf8')
        return ans
    except:
        return 'unknown'
```

Создание нового столбца с SMILE соединения
```python 
ds['Smiles'] = ds['наименование столбца из которого извлекается SMILE'].apply(lambda x: CIRconvert(x))
```

Функция *type_of*, в данном конкретном случае, создаёт список классов к которым принадлежит молекула, после чего выбирает первую в списке - приоритетную.

```python 
def types_of(mol):
  result = []
  for i in Classes_Org:
    if mol.find(i) != -1 and Classes_Org[i] not in result:
      result.append(Classes_Org[i])
  return result[0]
```
Если нужен полный список, удалите *[0]* в *return result[0]*

Функция *count_c* считывает количество С в SMILE...
```python 
def count_c(c):
    result = c.count('C') + c.count('c')
    return result
```
... в последующем используется в создании нового столбца
```python 
ds['sum_c'] = ds['Smiles'].apply(lambda x : count_c(x))
```

Цикл записывает считывает значения из столбца *'type'* , создает и заполняет столбец с этим классом соединений:
```python 
for i in ds['type'].unique():
    ds[i] = 0
  for i in range(len(ds)):
    obj = ds.iloc[i]
    stry = obj['type']
    obj[stry] = obj['Area Pct']
    ds.iloc[i] = obj
```
