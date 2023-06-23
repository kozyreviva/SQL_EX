## 51

Найдите названия кораблей, имеющих наибольшее число орудий среди всех имеющихся кораблей такого же водоизмещения (учесть корабли из таблицы Outcomes).

```sql
WITH sh1 AS (SELECT name, displacement, numGuns
             FROM ships
             JOIN classes ON classes.class = ships.class

             UNION

             SELECT ship, displacement, numGuns
             FROM outcomes
             JOIN classes ON ship = class)

SELECT name
FROM sh1
WHERE numGuns >= ALL (SELECT numGuns
                      FROM sh1 AS sh2
                      WHERE sh1.displacement = sh2.displacement)
```

## 52

Определить названия всех кораблей из таблицы Ships, которые могут быть линейным японским кораблем,
имеющим число главных орудий не менее девяти, калибр орудий менее 19 дюймов и водоизмещение не более 65 тыс.тонн

```sql
SELECT name
FROM ships
JOIN classes ON classes.class = ships.class
WHERE (country = 'Japan' OR country IS NULL)
  AND (type = 'bb' OR type IS NULL)
  AND (bore < 19 OR bore IS NULL)
  AND (numGuns >= 9 OR numGuns IS NULL)
  AND (displacement <=65000 OR displacement IS NULL)
```
