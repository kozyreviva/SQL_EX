## 26

Найдите среднюю цену ПК и ПК-блокнотов, выпущенных производителем A (латинская буква). Вывести: одна общая средняя цена.

```sql
WITH Price as (SELECT price
          FROM PC 
          WHERE model in (SELECT model
                          FROM Product
                          WHERE type = 'pc'
                            AND maker = 'A')                        
          UNION all
          SELECT price
          FROM Laptop
          WHERE model in (SELECT model
                          FROM Product
                          WHERE type = 'laptop'
                            AND maker = 'A')
          )

SELECT AVG(price)
FROM Price
```


## 27

Найдите средний размер диска ПК каждого из тех производителей, которые выпускают и принтеры. Вывести: maker, средний размер HD.

```sql
Select p.maker, avg(pc.hd)
From PC as pc
Join Product as p on pc.model = p.model
Where maker in (select maker
                From Product
                Where type ='printer')
Group by p.maker
```

## 28

Используя таблицу Product, определить количество производителей, выпускающих по одной модели.

```sql
Select count(maker)
From Product
Where maker in(Select maker
               From Product
               Group by maker
               Having count(model) =1)
```

## 29

В предположении, что приход и расход денег на каждом пункте приема фиксируется не чаще одного раза в день [т.е. первичный ключ (пункт, дата)], написать запрос с выходными данными (пункт, дата, приход, расход). Использовать таблицы Income_o и Outcome_o.

```sql
Select i.point, i.date, i.inc, o.out
From Income_o as i
Left join Outcome_o as o on i.point = o.point and i.date = o.date

Union

Select o.point, o.date, i.inc, o.out
From Income_o as i
right join Outcome_o as o on i.point = o.point and i.date = o.date
```

## 30

В предположении, что приход и расход денег на каждом пункте приема фиксируется произвольное число раз (первичным ключом в таблицах является столбец code), требуется получить таблицу, в которой каждому пункту за каждую дату выполнения операций будет соответствовать одна строка.
Вывод: point, date, суммарный расход пункта за день (out), суммарный приход пункта за день (inc). Отсутствующие значения считать неопределенными (NULL).

```sql
With i as (
       select point, date, sum(inc) as inc
       from Income
       group by point, date), 
 o as (
       select point, date, sum(out) as out
       from Outcome
       group by point, date)

Select coalesce(i.point,o.point),
       Coalesce(i.date,o.date),
       o.out, 
       i.inc
From i full join o on i.point= o.point and i.date = o.date
```

## 31

Для классов кораблей, калибр орудий которых не менее 16 дюймов, укажите класс и страну.

```sql
Select class, country
From Classes
Where bore >= 16
```

## 32

Одной из характеристик корабля является половина куба калибра его главных орудий (mw). С точностью до 2 десятичных знаков определите среднее значение mw для кораблей каждой страны, у которой есть корабли в базе данных.

```sql
Select country, cast(AVG(power(bore,3)/2) as decimal(6,2)) as weight
From (Select country, bore, name
      From Ships as s
      Join Classes as c on s.class=c.class
      
      Union 

      Select country, bore, ship 
      From Outcomes 
      Join Classes on class = ship
      Where ship not in (select name from Ships)) as sp
Group by country
```

## 33

Укажите корабли, потопленные в сражениях в Северной Атлантике (North Atlantic). Вывод: ship.

```sql
Select ship
From Outcomes
Where result = 'sunk'
  And battle = 'North Atlantic'
```

## 34

По Вашингтонскому международному договору от начала 1922 г. запрещалось строить линейные корабли водоизмещением более 35 тыс.тонн. Укажите корабли, нарушившие этот договор (учитывать только корабли c известным годом спуска на воду). Вывести названия кораблей

```sql
SELECT name
FROM Ships
WHERE class IN (SELECT class
                FROM Classes
                WHERE type = 'bb'
                  AND displacement > 35000)
  AND launched >= 1922
```

## 35

В таблице Product найти модели, которые состоят только из цифр или только из латинских букв (A-Z, без учета регистра).
Вывод: номер модели, тип модели.

```sql
SELECT model, type
FROM product
WHERE model NOT LIKE '%[^0-9]%' 
   OR model NOT LIKE '%[^A-Za-z]%'
```

## 36

Перечислите названия головных кораблей, имеющихся в базе данных (учесть корабли в Outcomes).

```sql
SELECT name
FROM ships 
WHERE name = class
UNION
SELECT ship
FROM outcomes
INNER JOIN classes ON outcomes.ship = classes.class
```

## 37

Найдите классы, в которые входит только один корабль из базы данных (учесть также корабли в Outcomes).

```sql
WITH s AS (SELECT ship, classes.class
           FROM outcomes
           JOIN classes ON outcomes.ship = classes.class
           UNION
           SELECT name, class
           FROM ships)

SELECT class 
FROM s   
WHERE class IN (SELECT class
                FROM s
                GROUP BY class
                HAVING COUNT(*)=1)
```

## 38

Найдите страны, имевшие когда-либо классы обычных боевых кораблей ('bb') и имевшие когда-либо классы крейсеров ('bc').

```sql
SELECT country 
FROM classes
WHERE type = 'bb'
INTERSECT
SELECT country 
FROM classes
WHERE type = 'bc'
```
