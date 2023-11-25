# Домашнее задание к занятию "`Индексы`" - `Стрекозов Владимир`

### Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.  
```SQL
SELECT  ((SELECT sum(index_length) from information_schema.tables WHERE table_schema = 'sakila')/(SELECT sum(data_length) from information_schema.tables WHERE table_schema = 'sakila'))*100 as index_size_in_data_size  
       FROM information_schema.tables
       WHERE table_schema = 'sakila' limit 1;
```  
![](https://github.com/Svalker1989/SQL_Indexes/blob/main/Z1.PNG)  

### Задание 2
Выполните explain analyze следующего запроса:  
```SQL
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
перечислите узкие места;  
После выполнения explain analyze бросается в глаза долгое выполнение оконной функции, при этом в ней происходит группировка по столбцам разных таблиц.  
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.  
Удалив из функции столбец f.title и саму таблицу film из запроса, запрос стал отрабатывать быстрее. 
Запрос после оптимизации:  
```SQL
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount), c.customer_id
from payment p
join rental r on p.payment_date = r.rental_date 
join customer c on r.customer_id = c.customer_id 
join inventory i on i.inventory_id = r.inventory_id 
where  date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
group by c.customer_id;
```
Результат выполнения запроса после оптимизации:  
 ![](https://github.com/Svalker1989/SQL_Indexes/blob/main/Z2_1.PNG)  
![](https://github.com/Svalker1989/SQL_Indexes/blob/main/Z2_2.PNG)  
Так жедобавил индекс, я его пробовал и первый раз добавлять, но изменений в скорости выполнения не увидел, пожтому не стал о нем писать.
![](https://github.com/Svalker1989/SQL_Indexes/blob/main/Z2_3.PNG)  
