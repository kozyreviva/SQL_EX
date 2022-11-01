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

## 15
Найдите размеры жестких дисков, совпадающих у двух и более PC. Вывести: HD

```sql
Select hd
From PC
Group by hd
Having count(hd) >= 2
```

## 16
Найдите пары моделей PC, имеющих одинаковые скорость и RAM. В результате каждая пара указывается только один раз, т.е. (i,j), но не (j,i), Порядок вывода: модель с большим номером, модель с меньшим номером, скорость и RAM.

```sql
Select distinct pc1.model, pc2.model, pc1.speed, pc1.RAM
From PC as pc1, 
     PC as pc2
Where pc1.speed = pc2.speed
  And pc1.RAM = pc2.RAM
  And pc1.model > pc2.model
```

## 17
Найдите модели ПК-блокнотов, скорость которых меньше скорости каждого из ПК.
Вывести: type, model, speed

```sql
Select distinct p.type, l.model, l.speed
From Product as p,
     Laptop as l
Where l.speed < all(select speed from PC)
  And p.type = 'laptop'
```

## 18
Найдите производителей самых дешевых цветных принтеров. Вывести: maker, price

```sql
SELECT distinct p.maker, pr.price
FROM product as p
Join printer as pr ON p.model =pr.model
Where pr.color = 'y'
  AND pr.price = (Select MIN(price) 
                  FROM printer
                  WHERE printer.color ='y')
```

## 19
Для каждого производителя, имеющего модели в таблице Laptop, найдите средний размер экрана выпускаемых им ПК-блокнотов.
Вывести: maker, средний размер экрана.

```sql
Select p.maker, AVG(l.screen)
From Product as p,Laptop as l
Where p.model = l.model
Group by p.maker
```

## 20
Найдите производителей, выпускающих по меньшей мере три различных модели ПК. Вывести: Maker, число моделей ПК.

```sql
Select maker, count(model)
From Product
Where type = 'pc'
Group by maker
Having count(model) >= 3
```

## 21
Найдите максимальную цену ПК, выпускаемых каждым производителем, у которого есть модели в таблице PC.
Вывести: maker, максимальная цена.

```sql
Select p.maker, MAX(pc.price)
From Product as p,
     PC as pc
Where pc.model =p.model
Group by p.maker
```

## 22
Для каждого значения скорости ПК, превышающего 600 МГц, определите среднюю цену ПК с такой же скоростью. Вывести: speed, средняя цена.

```sql
Select speed, AVG(price)
From PC
Where speed > 600
Group by speed
```

## 23
Найдите производителей, которые производили бы как ПК
со скоростью не менее 750 МГц, так и ПК-блокноты со скоростью не менее 750 МГц.
Вывести: Maker

```sql
SELECT DISTINCT maker 
 FROM Product 
 WHERE maker IN ( 
         SELECT DISTINCT maker  
         FROM Product AS pr 
         JOIN PC AS pc 
         ON pr.model = pc.model 
         WHERE pc.speed >= 750 
 ) 
 AND maker IN ( 
         SELECT DISTINCT maker  
         FROM Product AS pr  
         JOIN Laptop AS l 
         ON pr.model = l.model 
         WHERE l.speed >= 750 
 )
```

## 24
Перечислите номера моделей любых типов, имеющих самую высокую цену по всей имеющейся в базе данных продукции.

```sql
With Max as 
     (Select model, price from Laptop
      Union
      Select model, price from PC
      Union 
      Select model, price from Printer)

Select model
From Max
Where  price = (Select max(price) 
                From Max)
```

## 25

Найдите производителей принтеров, которые производят ПК с наименьшим объемом RAM и с самым быстрым процессором среди всех ПК, имеющих наименьший объем RAM. Вывести: Maker

```sql
SELECT DISTINCT maker
FROM product
WHERE model IN (SELECT model
                FROM pc
                WHERE ram = (SELECT MIN(ram)
                             FROM pc
                            )
                  AND speed = (SELECT MAX(speed)
                               FROM pc
                               WHERE ram = (SELECT MIN(ram)
                                            FROM pc
                                            )
                               )
                )
 AND maker in (SELECT maker
              FROM Product
              WHERE type = 'printer'
             )
```
