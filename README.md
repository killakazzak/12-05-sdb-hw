# Домашнее задание к занятию "`Индексы`" - `Тен Денис`

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение Задание 1

```sql
SELECT
    ROUND((SUM(index_length) / SUM(data_length)) * 100, 1) AS index_to_table_ratio_percentage
FROM
    information_schema.tables
WHERE
    table_schema = 'sakila';
```

![image](https://github.com/killakazzak/12-05-sdb-hw/assets/32342205/4d286035-bac7-4bec-8005-b4875f6c0d64)


### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение Задание 2

```text
Анализируя выполнение запроса, можно выделить следующие узкие места:

1. **Table scan on \<temporary\>** - сканирование временной таблицы без использования индексов. Временная таблица используется для дедупликации данных.

2. **Window aggregate with buffering** - оконная агрегация с буферизацией. В данном случае, выполняется оконная функция `sum(payment.amount) OVER (PARTITION BY c.customer_id, f.title)`, которая требует буферизации данных.

3. **Sort: c.customer_id, f.title** - сортировка результатов по столбцам `customer_id` и `title`. Это может быть узким местом при больших объемах данных.

4. **Nested loop inner join** - вложенные циклы для выполнения внутреннего соединения таблиц. В данном случае, используется несколько вложенных циклов для объединения таблиц `payment`, `rental`, `customer`, `inventory` и `film`. 

5. **Inner hash join (no condition)** - хэш-соединение без условия. Данное соединение выполняется без условия соединения между таблицами `payment` и `film`.

6. **Covering index scan on f using idx_title** - сканирование покрывающего индекса на таблице `film` для выполнения фильтрации по столбцу `title`. Это может быть узким местом при большом количестве данных в таблице `film`.

7. **Covering index lookup on r using rental_date** - выполнение поиска по покрывающему индексу на таблице `rental` по столбцу `rental_date`. Это может быть узким местом при большом количестве данных в таблице `rental`.

8. **Single-row index lookup on c using PRIMARY** - выполнение поиска по индексу на таблице `customer` для получения одной строки данных.

9. **Single-row covering index lookup on i using PRIMARY** - выполнение поиска по покрывающему индексу на таблице `inventory` для получения одной строки данных.

Эти узкие места могут быть оптимизированы путем использования индексов, улучшения структуры запроса и оптимизации использования оконных функций.
```


Для оптимальной производительности добавляем следующие индексы:

```sql
CREATE INDEX idx_payment_date ON payment(payment_date);
CREATE INDEX idx_rental_date ON rental(rental_date);
CREATE INDEX idx_customer_id ON customer(customer_id);
CREATE INDEX idx_inventory_id ON inventory(inventory_id);
CREATE INDEX idx_film_id ON film(film_id);
```
Исправленный запрос с добавленными индексами:
```sql
SELECT CONCAT(c.last_name, ' ', c.first_name) AS full_name, SUM(p.amount) AS total_amount
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE DATE(p.payment_date) = '2005-07-30'
GROUP BY full_name;
```


## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

### Решение Задание 3*

```text

В PostgreSQL существует несколько типов индексов, которые отсутствуют в MySQL. Ниже перечислены некоторые из них:

1. GIN (Generalized Inverted Index): GIN-индекс используется для индексации коллекций значений, таких как массивы, JSON-объекты и полнотекстовый поиск.

2. SP-GiST (Space-Partitioned Generalized Search Tree): SP-GiST-индекс предназначен для индексации данных, которые могут быть разделены на области, такие как геометрические объекты или полнотекстовые документы.

3. BRIN (Block Range Index): BRIN-индекс используется для индексации больших таблиц, разделенных на блоки данных. Он предоставляет компактное представление данных и обеспечивает эффективный доступ к ним.

4. Hash (Хэш-индекс): Хэш-индекс используется для быстрого поиска точных значений. Он хорошо подходит для равенственных операций (=) и не поддерживает сортировку.

5. Partial (Частичный индекс): Частичный индекс позволяет создавать индексы только для определенных строк, удовлетворяющих заданному условию. Это позволяет уменьшить размер индексов и улучшить производительность запросов.

6. Expression (Индекс по выражению): Индекс по выражению позволяет создавать индексы на основе вычисляемых выражений, а не только на конкретных столбцах. Это полезно для индексации выражений или функций.

7. PostGIS (Географический индекс): Географический индекс используется для индексации географических данных, таких как точки, линии и полигоны. Он обеспечивает эффективный поиск и анализ пространственных данных.

В MySQL не все эти типы индексов доступны, и некоторые из них могут быть реализованы с помощью дополнительных расширений или плагинов.

```
