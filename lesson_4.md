## Агрегация позволяет произвести расчет значений полей одной из существующих функций. Агрегация может быть по всей таблице или по группа (используя group by).

## Виды агрегаций

1. count() - кол-во не null строк
2. max() - максимальное значение
3. min() - минимальное значение
4. avg() - среднее значение
5. sum() - сумма значений

```sql
-- шаблон
select 
	max(<column_name>),
	min(<column_name>),
	avg(<column_name>),
	sum(<column_name>)
from <table_name>
```

Важно указать, что при агрегации в select могут быть только те поля, которые либо присутствуют в группировке, либо обработаны агрегационной функцией

```sql
-- шаблон
select 
	max(<column_name>),
	<column1_name> -- ОШИБКА!!!
from <table_name>
```

Задание

```sql
-- Найдите максимальную зарплату сотрудников из таблицы HR.EMPLOYEES

select 
	max(salary) as max_salary
from HR.EMPLOYEES
```

## Подзапросы

Подзапросы не являются неотъемлемой частью агрегации и могут использоваться без нее, однако они помогают реализовывать сложную логику в паре с агрегацией.

Подзапрос может использоваться для формирования источника данных в запросе.

```sql
-- шаблон

select 
	<column_name>
from (
	select *
	from <table1_name>
) t1
```

Обратите внимание что в таких случаях указывать алиас у источника данных необходимо 

Так же результат работы подзапроса может использоваться в условии

```sql
-- шаблон

select 
	<column_name>
from <table_name>
where <column1_name> = (
	select 
		max(<column_name>)
	from <table1_name>
)
```

Задание

```sql
-- Найдите имена и фамилии сотрудников с максимальной зарплатой HR.EMPLOYEES

select 
    FIRST_NAME,
    LAST_NAME
from hr.employees 
where salary = (select max(salary) from hr.employees )
```

## Группировка

Группировка позволяет сформировать отдельные группы из строк и считать агрегацию в них. Группы формируются по различным значениям указанного поля или полей.

Давайте разберем шаблон

```sql
select 
<column1_name>,
	max(<column_name>)
from <table_name>
group by <column1_name>
```

В данном запросе группы формируются по уникальным значениям column1_name и в них идет рассчет максимального значения column_name.

Задание

```sql
-- Найдите имена и фамилии сотрудников с максимальной зарплатой
-- в каждом департаменте HR.EMPLOYEES

select 
    t1.FIRST_NAME,
    t2.LAST_NAME
from hr.employees t1
inner join (
    select 
        department_id,
        max(salary) as max_salary
    from hr.employees 
    group by department_id
) t2
on t1.department_id = t2.department_id
and t1.salary = t2.max_salary
```

## Having

Оператор having позволяет указать условия на сагрегированных данных 

```sql
select 
<column1_name>,
	max(<column_name>)
from <table_name>
group by <column1_name>
having max(<column_name>) >10
```

Задание

```sql
/*
Выведите название отделов с кол-вом сотрудников больше 10
*/

select 
    t1.department_name
from hr.departments t1
inner join (
    select 
        department_id
    from hr.employees
    group by department_id
    having count(*) > 10
) t2
on t1.department_id = t2.department_id

/*
Выведите название отделов с кол-вом сотрудников больше среднего
*/

select 
    t1.department_name
from  hr.departments t1
inner join (
    select 
        department_id
    from hr.employees
    group by department_id
    having count(*) > (
        select 
            avg(cnt)
        from (
            select 
                count(*) as cnt
            from hr.employees
            group by department_id
        ) t1
    )
) t2
on t1.department_id = t2.department_id
```

## count

Функция count имеет некоторые особенности, она позволяет считать кол-во ненулевых строк у всех полей, у одного поля или уникальные значения поля.

```sql
-- шаблон подсчета ненулевых строк по всем полям
select 
	<column1_name>,
	count(*)
from <table_name>
group by <column1_name>

-- шаблон подсчета ненулевых строк по одному полю
select 
	<column1_name>,
	count(<column_name>)
from <table_name>
group by <column1_name>

-- шаблон подсчета уникальных строк по указанному полю
select 
<column1_name>,
	count(distinct <column_name>)
from <table_name>
group by <column1_name>

```

Задание

```sql
/*
Посчитайте кол-во людей в каждом департаменте
*/

select
	department_id,
	count(*) as cnt
from hr.employees
group by 	department_id;
```

## Оператор Case

Оператор case позволяет сформировать значение в зависимости от условия. Это единственный вариант реализации условного оператора в SQL. 

Важно заметить, что оператор case может как формировать значение для поля, так и быть вложенным в case или в условие.

```sql
/*
Рассчет значения поля
*/

select 
<column1_name>,
	case
		when <column2_name> is null
			then 0
		else <column2_name>
	end 
from <table_name>;

/*
Рассчет значения для агрегаци
*/

select 
	sum(
	case
		when <column2_name> is null
			then 0
		else <column2_name>
	end 
)
from <table_name>
```

Задание

```sql
/*
Сформировать поле SALARY_GROUP которое принимает 
	- значение 1, если зп сотрудника больше 10000 
	- значение 0, если зп сотрудника меньше 10000
*/

select
	case
		when salary > 10000 then 1
		when salary < 10000 then 0
	end as SALARY_GROUP
from hr.employees;

/*
Посчитать кол-во записей в этих группах
*/

select
	sum(
		case
			when salary > 10000 then 1
			when salary < 10000 then 0
		end
	) as SALARY_GROUP_1,
	sum(
		case
			when salary > 10000 then 0
			when salary < 10000 then 1
		end
	) as SALARY_GROUP_2
from hr.employees;

/*
Необходимо посчитать кол-во денег, которые должен уплатить каждый клиент
*/

select 
    t1.CUST_FIRST_NAME,
    t1.CUST_LAST_NAME,
    t2.customer_sum
from oe.customers t1
left join (
    select 
        t1.customer_id,
        sum(t2.total_summ) as customer_sum
    from oe.orders t1
    left join (
        select
            order_id,
            sum(UNIT_PRICE*QUANTITY) as total_summ
        from oe.order_items
        group by order_id
    ) t2
    on t1.order_id = t2.order_id
    group by t1.customer_id 
) t2
on t1.customer_id = t2.customer_id

```