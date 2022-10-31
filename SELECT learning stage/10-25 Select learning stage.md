## 10
Найдите модели принтеров, имеющих самую высокую цену. Вывести: model, price

```sql
Select model, price
From Printer 
Where price in (Select MAX(price)
                From Printer)
```

## 11
Найдите среднюю скорость ПК.

```sql
Select AVG(speed)
From PC
```

## 12
Найдите среднюю скорость ПК-блокнотов, цена которых превышает 1000 дол.

```sql
Select AVG(speed)
From Laptop
Where price > 1000
```

## 13
Найдите среднюю скорость ПК, выпущенных производителем A.

```sql
Select AVG(pc.speed)
From PC as pc
Join Product as p ON pc.model=p.model
Where p.maker ='A'
```

## 14
Найдите класс, имя и страну для кораблей из таблицы Ships, имеющих не менее 10 орудий.

```sql
Select s.class, s.name, c.country
From Ships as s
Join Classes as c ON s.class=c.class
Where c.numGuns >= 10
```
