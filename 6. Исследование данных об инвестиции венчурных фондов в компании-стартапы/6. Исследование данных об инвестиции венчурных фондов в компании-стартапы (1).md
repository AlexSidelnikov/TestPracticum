# Анализ данных о фондах и инвестициях
# Описание проекта
Нужно проанализировать данные и написать запросы к базе, которая хранит информацию о венчурных фондах и инвестициях в компании-стартапы. Эта база данных основана на датасете Startup Investments, опубликованном на популярной платформе для соревнований по исследованию данных Kaggle.

![Image (4)](https://pictures.s3.yandex.net/resources/1_Baza_dannykh_1673427255.png)

1.
Посчитайте, сколько компаний закрылось.

```
SELECT count(status) 
FROM company 
WHERE status = 'closed'; 
```

2.

Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .

```
SELECT funding_total
FROM company 
WHERE category_code = 'news'
AND country_code = 'USA'
ORDER BY funding_total DESC
```

3.
Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

```
SELECT sum(price_amount )
FROM acquisition
WHERE term_code = 'cash'
AND (EXTRACT(YEAR FROM CAST(acquired_at  AS timestamp)) = '2011'
OR EXTRACT(YEAR FROM CAST(acquired_at  AS timestamp)) = '2012'
OR EXTRACT(YEAR FROM CAST(acquired_at  AS timestamp)) = '2013');
```

4.
Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.

```
SELECT first_name,
    last_name,
    twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%'
```


5.
Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.

```
SELECT *
FROM people
WHERE twitter_username LIKE '%money%' AND last_name LIKE 'K%'
```

6.
Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

```
SELECT SUM(funding_total), country_code 
FROM company
GROUP BY country_code
ORDER BY SUM(funding_total) DESC
```

7.
Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

```
SELECT funded_at,
    MIN(raised_amount),
    MAX(raised_amount)
FROM funding_round
GROUP BY funded_at
HAVING MIN(raised_amount) <> 0
    AND MIN(raised_amount) <> MAX(raised_amount);
```


8.
Создайте поле с категориями:
Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
Отобразите все поля таблицы fund и новое поле с категориями.

```
SELECT *,
CASE

    WHEN invested_companies > 100  THEN 'high_activity'
    WHEN invested_companies >= 20 AND invested_companies < 100  THEN 'middle_activity' 
    WHEN invested_companies < 20  THEN 'low_activity' 
END
FROM fund;
```

9.
Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.

```
SELECT 
       CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies >=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity , ROUND(AVG(investment_rounds))
FROM fund
GROUP BY activity
ORDER BY AVG(investment_rounds) ASC;
```

10.
Выгрузите таблицу с десятью самыми активными инвестирующими странами. Активность страны определите по среднему количеству компаний, в которые инвестируют фонды этой страны.
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды, основанные с 2010 по 2012 год включительно.
Исключите из таблицы страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Отсортируйте таблицу по среднему количеству компаний от большего к меньшему, а затем по коду страны в лексикографическом порядке.
Для фильтрации диапазона по годам используйте оператор BETWEEN.

```
SELECT country_code,
    MIN(invested_companies),
    MAX(invested_companies),
    AVG(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM founded_at) BETWEEN 2010 AND 2012
GROUP BY country_code
HAVING MIN(invested_companies) <> 0
ORDER BY AVG(invested_companies) DESC
LIMIT 10
```

11.
Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

```
SELECT p.first_name,
    p.last_name,
    ed.instituition
FROM  people AS P
LEFT JOIN education AS ed ON p.id = ed.person_id 
```

12.
Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

```
SELECT company.name , COUNT(DISTINCT ed.instituition) as con
FROM company
INNER JOIN  people AS p ON company.id  = p.company_id  
INNER JOIN  education AS ed ON p.id = ed.person_id  
GROUP BY company.name
ORDER BY con DESC 
LIMIT 5
```

13.

Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

```
WITH c AS (SELECT name
           FROM company
           LEFT JOIN funding_round AS fr ON company.id  = fr.company_id
           WHERE status = 'closed'
           AND fr.is_last_round = 1
           AND fr.is_first_round=1)

SELECT DISTINCT(c.name)
FROM c
GROUP BY c.name
```


14.

Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

```
WITH c AS (WITH fr AS (SELECT company_id, funded_at, is_first_round, is_last_round
           FROM funding_round
           WHERE is_last_round = 1
           AND is_first_round=1)
           SELECT DISTINCT name
           FROM company
           INNER JOIN fr  ON company.id  = fr.company_id
           WHERE status = 'closed')
SELECT DISTINCT people.id
FROM people
INNER JOIN company  ON people.company_id  = company.id
INNER JOIN c  ON company.name  = c.name
GROUP BY people.id;
```

15.

Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

```
SELECT  DISTINCT p.id, instituition
FROM people AS p
INNER JOIN education AS e ON p.id = e.person_id
INNER JOIN company AS c ON p.company_id = c.id
WHERE c.name IN (SELECT DISTINCT name
    FROM company AS c
    INNER JOIN funding_round AS fr ON fr.company_id = c.id
    WHERE is_last_round = 1 AND is_first_round = 1 AND status = 'closed')
```

16.
Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.

```
SELECT  DISTINCT p.id, COUNT(instituition)
FROM people AS p
INNER JOIN education AS e ON p.id = e.person_id
INNER JOIN company AS c ON p.company_id = c.id
WHERE c.name IN (SELECT DISTINCT name
    FROM company AS c
    INNER JOIN funding_round AS fr ON fr.company_id = c.id
    WHERE is_last_round = 1 AND is_first_round = 1 AND status = 'closed')
GROUP BY p.id;
```

17.
Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

```
SELECT SUM(count)/COUNT(id) AS vot_tak
FROM (SELECT  DISTINCT p.id, COUNT(instituition)
FROM people AS p
INNER JOIN education AS e ON p.id = e.person_id
INNER JOIN company AS c ON p.company_id = c.id
WHERE c.name IN (SELECT DISTINCT name
    FROM company AS c
    INNER JOIN funding_round AS fr ON fr.company_id = c.id
    WHERE is_last_round = 1 AND is_first_round = 1 AND status = 'closed')
GROUP BY p.id) AS average;
```


18.
Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Facebook*.
*(сервис, запрещённый на территории РФ)

```
SELECT SUM(count)/COUNT(id) AS vot_tak
FROM (SELECT  DISTINCT p.id, COUNT(instituition)
    FROM people AS p
    INNER JOIN education AS e ON p.id = e.person_id
    INNER JOIN company AS c ON p.company_id = c.id
    WHERE c.name = 'Facebook'
    GROUP BY p.id) AS average;
```

19.

Составьте таблицу из полей:
name_of_fund — название фонда;
name_of_company — название компании;
amount — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

```
WITH
f AS (SELECT name
     FROM fund),
c AS (SELECT name
     FROM company),
fr AS (SELECT raised_amount
     FROM funding_round)
SELECT f.name AS name_of_fund,
       c.name AS name_of_company,
       raised_amount AS amount
FROM investment AS i
LEFT JOIN company AS c ON c.id = i.company_id
LEFT JOIN fund AS f ON f.id = i.fund_id
INNER JOIN funding_round AS fr ON fr.id = i.funding_round_id
WHERE c.milestones > 6 AND EXTRACT(YEAR FROM CAST(funded_at AS date)) IN (2012, 2013);
```

20.
Выгрузите таблицу, в которой будут такие поля:
название компании-покупателя;
сумма сделки;
название компании, которую купили;
сумма инвестиций, вложенных в купленную компанию;
доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.

```
SELECT c.name AS acquiring_company,
       a.price_amount,
       c1.name AS acquired_company,
       c1.funding_total, 
       ROUND(a.price_amount/c1.funding_total) AS cost_ratio
FROM acquisition AS a
LEFT JOIN company AS c ON a.acquiring_company_id = c.id
LEFT JOIN company AS c1 ON a.acquired_company_id = c1.id
WHERE price_amount > 0 AND c1.funding_total > 0
ORDER BY a.price_amount DESC, c1.name
LIMIT 10;
```


21.
Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.

```
SELECT c.name, EXTRACT(MONTH FROM CAST(funded_at AS date)) AS month
FROM company AS c
INNER JOIN funding_round AS fr ON fr.company_id = c.id
WHERE c.category_code = 'social' 
    AND EXTRACT(YEAR FROM CAST(fr.funded_at AS date)) IN (2010, 2011, 2012, 2013)
    AND fr.raised_amount > 0;
```

22.
Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
номер месяца, в котором проходили раунды;
количество уникальных названий фондов из США, которые инвестировали в этом месяце;
количество компаний, купленных за этот месяц;
общая сумма сделок по покупкам в этом месяце.

```
WITH
mf AS (SELECT EXTRACT(month from CAST(fr.funded_at as date)) as month,
        COUNT(DISTINCT f.name) as count_fund
        FROM funding_round AS fr
        INNER JOIN investment as i ON fr.id=i.funding_round_id
        INNER JOIN fund as f ON i.fund_id=f.id
        WHERE EXTRACT(year from CAST(fr.funded_at as date)) BETWEEN 2010 AND 2013
        AND f.country_code='USA'
        GROUP BY month),

a AS (SELECT EXTRACT(MONTH FROM CAST(acquired_at as date)) as month,
       COUNT(acquired_company_id) AS count_acquired_company,
       SUM(price_amount) AS amount
       FROM acquisition
       WHERE EXTRACT(YEAR FROM CAST(acquired_at as date)) BETWEEN 2010 AND 2013
       GROUP BY month)
SELECT mf.month,
       count_fund,
       count_acquired_company,
       amount
       
FROM mf
LEFT JOIN a ON mf.month = a.month;
```

23.
Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.

```
WITH
y_2011 AS 
    (SELECT country_code, AVG(funding_total) AS founded_at_2011
    FROM company
    WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) IN (2011)
    GROUP BY country_code),
y_2012 AS 
    (SELECT country_code, AVG(funding_total) AS founded_at_2012
    FROM company
    WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) IN (2012)
    GROUP BY country_code),
y_2013 AS 
    (SELECT country_code, AVG(funding_total) AS founded_at_2013
    FROM company
    WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) IN (2013)
    GROUP BY country_code)

SELECT y_2011.country_code,
       founded_at_2011,
       founded_at_2012,
       founded_at_2013
FROM y_2011
INNER JOIN y_2012 ON y_2012.country_code = y_2011.country_code
INNER JOIN y_2013 ON y_2013.country_code = y_2011.country_code
ORDER BY founded_at_2011 DESC;
```
