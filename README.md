# Class of organic molecule
_____
## Autor: Dvoretskov Pavel
____

Данный проект является парсером классов органических молекул из SMILE структур <br>
Стек проекта:
- python version 3.11.0
- Google Colab

## Параметры некоторых настроек
____
Импорт датасета из таблицы гугл докс
```python 
wb =gc.open_by_url('ваша ссылка на таблицу google docx')
ws = wb.worksheet('страница документа с которой будете работать')
```

Функция парсит сайт и выводит SMILE
```python 
def CIRconvert(ids):
    try:
        url = 'http://cactus.nci.nih.gov/chemical/structure/' + quote(ids) + '/smiles'
        ans = urlopen(url).read().decode('utf8')
        return ans
    except:
        return 'Did not work'
```

Создание нового столбца с SMILE соединения
```python 
ds['Smiles'] = ds['наименование столбца из которого извлекается SMILE'].apply(lambda x: CIRconvert(x))
```

### Настройка заполенения словаря:
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
Можно установить типы классификации молекул в рамках SMILE.
Порядок в словаре указывает на в заполнении списка классов конкретного соединения - подробнее в блоке ниже.

Функция *type_of*, в данном конкретном случае, создает список классов к которым пренадлежит молекула, после чего выбирает первую в списке - приоритетную.

```python 
def types_of(mol):
  result = []
  for i in Classes_Org:
    if mol.find(i) != -1 and Classes_Org[i] not in result:
      result.append(Classes_Org[i])
  return result[0]
```
Если нужен полный список, удалите *[0]* в *return result[0]*

Создаем отдельный столбец с типом(типами) органических соединнений в таблице используя функцию из прошлого блока:
```python 
ds['тип соединения'] = ds['Smiles'].apply(lambda x : types_of(x))
```
