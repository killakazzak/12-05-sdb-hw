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

