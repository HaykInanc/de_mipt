## Очистка данных

Очистка данных играет важную роль в процессе загрузки данных. В задачи дата инженера входит привести данные к одному виду, оставить только валидные данные и отбросить все лишнее. 

Для очистки данных в SQL есть множество строковых функций.

|         |                                      |                                                |
| ------- | ------------------------------------ | ---------------------------------------------- |
| CONCAT  | CONCAT('A','BC')                     | объединение строк                              |
| INSTR   | INSTR( 'This is a playlist', 'is')   | возвращает позицию, где начинается подстрока   |
| LENGTH  | LENGTH('ABC')                        | возвращает длину строки                        |
| REPLACE | REPLACE('JACK AND JOND','J','BL')    | заменяет подстроку на другую                   |
| SUBSTR  | SUBSTR('Oracle Substring', 1, 6 )    | позволяет получить подстроку                   |




Примеры

```sql
select concat('привет', 'друг') from dual; -- конкатенация
select 'привет'|| ' '|| 'друг' from dual; -- конкатенация через оператор ||

-- позиция подстроки в строке 
select instr('привет мой друг', 'мой') from dual; 
-- при отсутствии подстроки значение 0
select instr('привет мой друг', 'мой_') from dual; 

-- длинна строки
select length('привет мой друг') from dual;
-- при измерении длинны учитываются боковые пробельные символы
select length('привет мой друг           ') from dual;

-- замена подстроки на другую подстроку
select replace('привет мой друг', 'друг', 'товарищ') from dual;
select replace('привет мой друг', ' ', '_') from dual;

-- достать подстроку из строки 
-- 1 параметр - позиция с которой начинается подстрока
-- 2 параметр - длинна получаемой подстроки
select substr('привет мой друг', 8, 3) from dual;
```

## Задание

Таблица **hr.employees**

1. Сформируйте поле, которое указывает последние 4 цифры номера телефона сотрудника
2. Уберите в телефоне символ .
3. Определите работников с самым длинным email адресом
4. Сформируйте поле с именем и фамилией сотрудника

```sql
-- Сформируйте поле, которое указывает последние 4 цифры номера телефона сотрудника

select substr(PHONE_NUMBER,length(PHONE_NUMBER)-3 ) from hr.employees;

-- Уберите в телефоне символ .

select replace(PHONE_NUMBER, '.', '') from hr.employees;
-- в случае если подстрока меняется на пустую строку, второй параметр не обязателен
select replace(PHONE_NUMBER, '.') from hr.employees; 

-- Определите работников с самым длинным email адресом

select * from hr.employees 
where length(email) = ( 
    select max(length(email)) from hr.employees;
)

-- Сформируйте поле с именем и фамилией сотрудника
select FIRST_NAME || ' ' || LAST_NAME from hr.employees;
```

|         |                                      |                                                |
| ------- | ------------------------------------ | ---------------------------------------------- |
| UPPER   | UPPER('Abc')                         | Перевод в нижний регистр                       |
| INITCAP | INITCAP('hi  there')                 | Первая буква большая, остальные маленькие      |


Примеры

```sql
select 
	lower('пРиВеТ'), 
	upper('пРиВеТ'), 
	initcap('пРиВеТ')  
from dual;
```


|         |                                      |                                                |
| ------- | ------------------------------------ | ---------------------------------------------- |
| LPAD    | LPAD('ABC',5,'*')                    | Добавляет указанное кол-во символов слева      |
| RPAD    | RPAD('ABC',5,'*')                    | Добавляет указанное кол-во символов справа     |
| LTRIM   | LTRIM(' ABC ')                       | Убирает пробелы слева                          |
| RTRIM   | LTRIM(' ABC ')                       | Убирает пробелы справа                         |
| TRIM    | LTRIM(' ABC ')                       | Убирает пробелы по краям                       |



Примеры

```sql
select 
	lpad(n, 10, '0'),
	rpad(n, 10, '0')    
from ( 
    select 12 as n from dual 
    union all 
    select 1232 as n from dual 
    union all 
    select 6533 as n from dual 
    union all 
    select 22145 as n from dual 
) t1;

select 
	ltrim(n), 
	rtrim(n), 
	trim(n) from ( 
    select 
			'      привет друг        ' as n 
		from dual 
) t1
```

## Практическое задание

Необходимо загрузить данные и очистить их.

https://raw.githubusercontent.com/HaykInanc/mtsData/master/sql_for_oracle.sql

Давайте загрузим данные и изучим их. Мы можем наблюдать следующие особенности

1) поле Name и Lastname заполнены со сдвигами. То есть если name пустой (значение null), то в lastname лежит и имя и фамилия и наоборот, если lastname пустой, то имя и фамилия лежат вместе. Важно заметить, что порядок не нарушается. В начале идет имя, потом фамилия, даже если они в одной ячейке. Значения из этих полей необходимо расставить по своим местам.

2) поле email может содержать почту, телефон или и почту и телефон. Это поле следует разбить на два, email и phone

3) поле gender заполнено несколькими вариантами значений, необходимо его привести к бинарному (1 - мужчина, 0 - женщина).

Давайте начнем с простого и очистим поле gender, тут может быть несколько решений, здесь будет указано самое простое. 

```sql
select  
case  
    when substr(GENDER, 1, 1) = 'F' then 0 
    else 1 
end as GENDER, 
GENDER as column_ 
from dataSource
```

Для проверки в запросе выводятся и новое и старое значение. 

Следующим шагом давайте разобьем name и lastname.

```sql
select  
    case  
        when instr(FIRST_NAME, ' ') = 0 
            then FIRST_NAME 
        when LAST_NAME is null  
            then substr(FIRST_NAME, 1, instr(FIRST_NAME, ' ')-1) 
        when FIRST_NAME is null  
            then substr(LAST_NAME, 1, instr(LAST_NAME, ' ')-1) 
    end as new_FIRST_NAME, 
     
    case  
        when instr(LAST_NAME, ' ') = 0 
            then LAST_NAME 
        when FIRST_NAME is null  
            then substr(LAST_NAME, instr(LAST_NAME, ' ')+1) 
        when LAST_NAME is null  
            then substr(FIRST_NAME, instr(FIRST_NAME, ' ')+1) 
    end as new_LAST_NAME, 
    first_name, 
    LAST_NAME
from dataSource
```

Тут можно видеть, что name и lastname обрабатываются подобным образом. Разберем пример FIRST_NAME.

У нас есть 3 возможные ситуации

- first name заполнен корректно (тогда мы просто оставляем его как он есть)
- first name содержит и first name и last name (отрезаем все до пробела)
- first name содержит null (отрезаем все до пробела у last name)

Отлично! Нам остается разделить значения поля email

```sql
select  
		case 
        when instr(email, '@') <> 0 
            case  
                when instr(email, ' ') = 0 
                    then email 
                else substr(email, 0,  instr(email, ' ')) 
            end  
    end as new_email, 
    email
from dataSource;
```

В данном запросе мы проверяем:
вхождение символа @, это позволит нам определить, есть ли в строке почта.

- **да.** Тогда проверяем, наличие пробела, это позволит определить, есть ли в этом поле еще и телефон
    - **да.** Тогда мы отрезаем строку от начала до пробела
    - **нет.** Тогда у нас в поле только email, можем добавлять
- **нет.** Тогда email в поле отсутствует, можем вписывать null

Поле с телефоном мы добавляем подобным образом, отталкиваясь от @ и пробела, но подставляем иные значения.

```sql
select  
		case 
        when instr(email, '@') <> 0 
            then case  
                when instr(email, ' ') = 0 
                    then null 
                else substr(email,instr(email, ' ')+1) 
            end  
        else email 
    end as new_phone,
    email
from dataSource;
```

Остается собрать все эти запросы в один и провести очистку поля phone. Итоговый скрипт может выглядеть следующим образом.

```sql
-- создание промежуточного представления

create table dataSource_01 as  
select  
    FIRST_NAME,	 
    LAST_NAME,	 
    EMAIL,	 
    case 
        when substr(PHONE, 1, 1) = '8' then  '+7' || substr(PHONE, 2) 
        else phone 
    end as phone, 
    GENDER 
from ( 
    select  
        case  
            when instr(FIRST_NAME, ' ') = 0 
                then FIRST_NAME 
            when LAST_NAME is null  
                then substr(FIRST_NAME, 1, instr(FIRST_NAME, ' ')-1) 
            when FIRST_NAME is null  
                then substr(LAST_NAME, 1, instr(LAST_NAME, ' ')-1) 
        end as FIRST_NAME, 
        case  
            when instr(LAST_NAME, ' ') = 0 
                then LAST_NAME 
            when FIRST_NAME is null  
                then substr(LAST_NAME, instr(LAST_NAME, ' ')+1) 
            when LAST_NAME is null  
                then substr(FIRST_NAME, instr(FIRST_NAME, ' ')+1) 
        end as LAST_NAME, 
        case 
            when instr(email, '@') <> 0 
                then case  
                    when instr(email, ' ') = 0 
                        then email 
                    else substr(email, 0,  instr(email, ' ')) 
                end  
        end as email, 
         
        case 
            when instr(email, '@') <> 0 
                then case  
                    when instr(email, ' ') = 0 
                        then null 
                    else substr(email,instr(email, ' ')+1) 
                end  
            else email 
        end as phone, 
        case  
            when substr(GENDER, 1, 1) = 'F' then 0 
            else 1 
        end as GENDER 
    from dataSource 
) t1;

-- итоговый запрос
select      
    FIRST_NAME,	 
    LAST_NAME,	 
    EMAIL,	 
    replace(replace(PHONE, '-', ''), ' ', '') as PHONE, 
    GENDER 
from dataSource_01
```