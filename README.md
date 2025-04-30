# DZ12-05
# Домашнее задание к занятию «Индексы» Тимохин Вячеслав Витальевич

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

![1_Снимок экрана от 2025-04-27 21-09-52](https://github.com/user-attachments/assets/3c727b30-1167-4c5f-aa98-0eda2245a4ca)


### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

![2_Снимок экрана от 2025-04-30 21-53-50](https://github.com/user-attachments/assets/bc296fff-dc60-43de-901d-b1204abed646)
1 Полное сканирование таблицы payment, чтобы отфильтровать строки с payment_date, равной '2005-07-30'. Это может быть медленным, если таблица крупная.
2 Отсутствие индекса на столбец payment_date, что делает операцию фильтрации медленной.
3 Полное сканирование таблицы rental, чтобы присоединить ее к таблице payment. Это может быть медленно, если таблиа крупная.
4 Отсутствие индекса на столбец rental_date, что делает операию присоединения медленной.
5 Операции хеш-соединения, которые могут быть медленными для крупных наборов данных.
6 Последовательные сканирования таблиц inventory и film, чтобы присоединить их к таблице rental, это может быть медленным, если таблицы крупные.

Чтобы оптимизировать запрос, мы можем рассмотреть создание индексов на столбцах payment_date и rental_date, а также оптимизацию порядка соединения и использование более эффективных алгоритмов соединения.


Создать индекс на столбец payment_date в таблице payment:
CREATE INDEX idx_payment_date ON payment (payment_date);

Создать индекс на столбец rental_date в таблице rental:
CREATE INDEX idx_rental_date ON rental (rental_date);

Создать индекс на столбец customer_id в таблице rental:
CREATE INDEX idx_rental_customer_id ON rental (customer_id);

Создать индекс на столбец inventory_id в таблице rental:
CREATE INDEX idx_rental_inventory_id ON rental (inventory_id);

Создать индекс на столбец film_id в таблице inventory:
CREATE INDEX idx_inventory_film_id ON inventory (film_id);

Оптимизированный запрос:
-----
SELECT DISTINCT
CONCAT(c.last_name, ' ', c.first_name),
SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM
payment p
INNER JOIN rental r ON p.rental_id = r.rental_id
INNER JOIN customer c ON r.customer_id = c.customer_id
INNER JOIN inventory i ON r.inventory_id = i.inventory_id
INNER JOIN film f ON i.film_id = f.film_id
WHERE
p.payment_date = '2005-07-30'
-----


## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*
