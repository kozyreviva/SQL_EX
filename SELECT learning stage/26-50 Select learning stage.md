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

## 39

Найдите корабли, `сохранившиеся для будущих сражений`; т.е. выведенные из строя в одной битве (damaged), они участвовали в другой, произошедшей позже.

```sql
WITH b AS (SELECT * 
           FROM outcomes AS o
           JOIN battles AS bt ON bt.name = o.battle
           WHERE EXISTS (SELECT ship
                         FROM outcomes
                         JOIN battles AS bt1 ON bt1.name = outcomes.battle
                         WHERE o.ship = outcomes.ship
                           AND result = 'damaged'
                           AND bt.date > bt1.date))
SELECT DISTINCT ship
FROM b
```

## 40

Найти производителей, которые выпускают более одной модели, при этом все выпускаемые производителем модели являются продуктами одного типа.
Вывести: maker, type

```sql
SELECT maker, type
FROM product
WHERE maker IN (SELECT maker
                FROM product
                GROUP BY maker
                HAVING COUNT(DISTINCT type) = 1)
GROUP BY maker,type
HAVING COUNT(model) > 1
```

## 41

Для каждого производителя, у которого присутствуют модели хотя бы в одной из таблиц PC, Laptop или Printer,
определить максимальную цену на его продукцию.
Вывод: имя производителя, если среди цен на продукцию данного производителя присутствует NULL, то выводить для этого производителя NULL,
иначе максимальную цену.

```sql
WITH un AS(SELECT model, price 
           FROM pc
           UNION
           SELECT model, price
           FROM laptop
           UNION
           SELECT model, price
           FROM printer)

SELECT maker, 
CASE 
WHEN MAX(CASE WHEN price IS NULL THEN 1 ELSE 0 END) = 0
THEN MAX(price) 
END  price        
FROM product
RIGHT JOIN un ON product.model=un.model 
GROUP BY maker
```
## 42

Найдите названия кораблей, потопленных в сражениях, и название сражения, в котором они были потоплены.

```sql
SELECT ship, battle
FROM outcomes
WHERE result = 'sunk'
```

## 43

Укажите сражения, которые произошли в годы, не совпадающие ни с одним из годов спуска кораблей на воду.

```sql
SELECT DISTINCT name
FROM battles
WHERE year(date) NOT IN (SELECT launched
                         FROM ships
                         WHERE launched IS NOT NULL)
```
## 44

Найдите названия всех кораблей в базе данных, начинающихся с буквы R.

```sql
WITH s AS (SELECT name
           FROM ships
           UNION
           SELECT ship
           FROM outcomes)

SELECT name
FROM s
WHERE name LIKE 'R%'
```

## 45

Найдите названия всех кораблей в базе данных, состоящие из трех и более слов (например, King George V).
Считать, что слова в названиях разделяются единичными пробелами, и нет концевых пробелов.

```sql
WITH s AS (SELECT name
           FROM ships
           UNION
           SELECT ship
           FROM outcomes)

SELECT name
FROM s
WHERE name LIKE '% % %'
```

## 46

Для каждого корабля, участвовавшего в сражении при Гвадалканале (Guadalcanal), вывести название, водоизмещение и число орудий.

```sql
SELECT name, displacement, numGuns
FROM outcomes
JOIN (classes JOIN ships ON classes.class = ships.class) ON ship = name
WHERE battle = 'Guadalcanal'

UNION

SELECT ship, displacement, numGuns
FROM outcomes
LEFT JOIN classes ON outcomes.ship = classes.class
WHERE battle = 'Guadalcanal'
  AND ship NOT IN (SELECT name FROM ships)
```

## 47

пределить страны, которые потеряли в сражениях все свои корабли.

```sql
WITH sh AS (SELECT c.country, s.name
            FROM classes AS c
            JOIN ships AS s ON c.class = s.class
            UNION
            SELECT c.country, o.ship
            FROM classes AS c 
            JOIN outcomes AS o ON c.class = o.ship),
      a AS (SELECT country, name,
            CASE 
                WHEN result = 'sunk'
                THEN 1
                ELSE 0
            END AS sunk
            FROM sh 
            LEFT JOIN outcomes AS o ON o.ship = sh.name)


SELECT country
FROM a
GROUP BY country
HAVING COUNT(DISTINCT name) = SUM(sunk)
```

## 48

Найдите классы кораблей, в которых хотя бы один корабль был потоплен в сражении.

```sql
SELECT c.class
FROM classes AS c
JOIN ships AS s ON s.class = c.class
JOIN outcomes AS o ON s.name = o.ship
WHERE result ='sunk'

UNION

SELECT c.class
FROM classes AS c
JOIN outcomes AS o ON c.class = o.ship
WHERE result = 'sunk'
```

## 49

Найдите названия кораблей с орудиями калибра 16 дюймов (учесть корабли из таблицы Outcomes).

```sql
SELECT name
FROM ships 
JOIN classes ON ships.class= classes.class
WHERE bore = 16

UNION

SELECT ship
FROM outcomes
JOIN classes ON classes.class=outcomes.ship
WHERE bore = 16
```

## 50

Найдите сражения, в которых участвовали корабли класса Kongo из таблицы Ships.

```sql
SELECT DISTINCT battle
FROM outcomes 
JOIN ships ON ship = name
WHERE class = 'Kongo'
```
