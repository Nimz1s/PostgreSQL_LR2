# Лабораторна робота 2. Створення складних SQL запитів

## Загальна інформація

**Здобувач освіти:** Вознюк Ілля
**Група:** ІПЗ-31
**Обраний рівень складності:** 1, 2, 3
**Посилання на проєкт:** https://supabase.com/dashboard/project/rwggnjxkdmrhruujuvnq

## Виконання завдань

### Список таблиць

```sql
-- Запит для отримання списку таблиць
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```

Результат: У базі даних створено 8 основних таблиць: categories, customers, employees, order_items, orders, products, regions, suppliers.

<img width="1452" height="429" alt="1" src="https://github.com/user-attachments/assets/7ffd9b2b-18f8-4f6b-b852-9def42bc9282" />


...


### 2.1 Створити запити з використанням INNER JOIN для отримання інформації з кількох пов'язаних таблиць

```sql
SELECT p.product_name, c.category_name, s.company_name, p.unit_price
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN suppliers s ON p.supplier_id = s.supplier_id
ORDER BY c.category_name, p.product_name;
```

Результат: Отримано запити з використанням INNER JOIN для отримання інформації з кількох пов'язаних таблиць

<img width="748" height="430" alt="2" src="https://github.com/user-attachments/assets/e3579549-6854-4f3f-81e3-75b5afd67c43" />

[Продовжити для всіх завдань обраного рівня]


### 2.2 Використати LEFT JOIN для включення всіх записів з головної таблиці

```sql
SELECT product_name, 
        unit_price 
FROM products;
```

Результат: отримано використано LEFT JOIN для включення всіх записів з головної таблиці


![alt text](C:\Users\vozni\Desktop\2.png)


### 2.3 Множинне з'єднання - детальна інформація про замовлення:

```sql
SELECT 
    o.order_id,
    o.order_date,
    c.customer_id,
    c.contact_name,
    p.product_id,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS total_price
FROM orders o
JOIN customers c 
    ON o.customer_id = c.customer_id
JOIN order_items oi 
    ON o.order_id = oi.order_id
JOIN products p 
    ON oi.product_id = p.product_id
ORDER BY o.order_id;
```

Результат: множинне з'єднання взяте з декількох таблиць

<img width="1471" height="477" alt="3" src="https://github.com/user-attachments/assets/ac87a43d-1be5-48e6-8d02-15c176ef00b8" />





### 3.1 Підрахувати кількість товарів у кожній категорії та середню ціну:

```sql
SELECT c.category_name,
      COUNT(p.product_id) as product_count,
      AVG(p.unit_price) as avg_price,
      MIN(p.unit_price) as min_price,
      MAX(p.unit_price) as max_price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY product_count DESC;
```

Результат: отримано  кількість товарів у кожній категорії та середню ціну.


<img width="762" height="468" alt="4" src="https://github.com/user-attachments/assets/e037eb04-c352-452b-b009-89bdc9580a75" />




### 3.2 Визначити загальні продажі за регіонами:

```sql
SELECT 
    r.region_name,
    SUM(oi.quantity * oi.unit_price) AS total_sales
FROM orders o
JOIN order_items oi 
    ON o.order_id = oi.order_id
JOIN customers c 
    ON o.customer_id = c.customer_id
JOIN regions r 
    ON c.region_id = r.region_id
GROUP BY r.region_name
ORDER BY total_sales DESC;
```

Результат: отримано  загальні продажі за регіонами.

<img width="750" height="460" alt="5" src="https://github.com/user-attachments/assets/21b1fffc-879f-47ca-8ce7-03d67983ad88" />




### 3.3 Знайти постачальників з кількістю товарів більше 2:

```sql
SELECT 
    s.supplier_id,
    s.company_name,
    COUNT(p.product_id) AS product_count
FROM suppliers s
JOIN products p
    ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY product_count DESC;
```

Результат: отримано постачальників з кількістю товарів більше 2.

<img width="750" height="457" alt="6" src="https://github.com/user-attachments/assets/c6a0be2f-8d9a-4cc0-8dbc-7a3fa194d501" />




### 4.1 Знайти товари з ціною вищою за середню ціну товарів у своїй категорії:

```sql
SELECT p.product_name, p.unit_price, c.category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE p.unit_price > (
    SELECT AVG(p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.category_name, p.unit_price DESC;
```

Результат: отримано товари з ціною вищою за середню ціну товарів у своїй категорії.

<img width="750" height="463" alt="7" src="https://github.com/user-attachments/assets/3dee0a02-b037-41e3-bf3a-e5f8fab22c60" />




### 4.2 Отримати клієнтів, які мали замовлення у 2024 році:

```sql
SELECT DISTINCT 
    c.customer_id,
    c.contact_name
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
WHERE EXTRACT(YEAR FROM o.order_date::date) = 2024
ORDER BY c.contact_name;
```

Результат: отримано замовлення від найновіших до найстаріших.

![alt text](C:\Users\vozni\Desktop\8.png)


### 4.3 Додати до списку товарів інформацію про загальну кількість продажів:

```sql
SELECT 
    p.product_id,
    p.product_name,
    COALESCE(SUM(oi.quantity), 0) AS total_sold
FROM products p
LEFT JOIN order_items oi
    ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name
ORDER BY total_sold DESC;
```

Результат: отримано список товарів інформацію про загальну кількість продажів.

<img width="745" height="453" alt="8" src="https://github.com/user-attachments/assets/30b6886a-f8ff-47b8-a8be-8e2968145c92" />




### 5.1 RIGHT JOIN для аналізу категорій товарів та їх наявності:

```sql
SELECT c.category_name,
      COUNT(p.product_id) as products_count,
      COALESCE(AVG(p.unit_price), 0) as avg_price
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY products_count DESC;
```

Результат: отримано RIGHT JOIN для аналізу категорій товарів та їх наявності.

<img width="744" height="474" alt="5 1" src="https://github.com/user-attachments/assets/3b65032f-70f4-4e9f-a6a3-4d302a5048ee" />




### 5.2 Self-join для знаходження співробітників та їх керівників:

```sql
SELECT 
    e1.first_name || ' ' || e1.last_name AS employee,
    e1.title AS employee_title,
    e2.first_name || ' ' || e2.last_name AS manager,
    e2.title AS manager_title
FROM employees e1
LEFT JOIN employees e2 
    ON e1.reports_to::bigint = e2.employee_id
ORDER BY e2.last_name, e1.last_name;
```

Результат: отримано Self-join для знаходження співробітників та їх керівників.

<img width="754" height="471" alt="5 2" src="https://github.com/user-attachments/assets/fdce3117-d3af-4813-b450-1a6e78dded05" />






### 5.3 Ранжувати товари за ціною в межах категорії:

```sql
SELECT p.product_name,
      c.category_name,
      p.unit_price,
      RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) as price_rank,
      DENSE_RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) as price_dense_rank,
      ROW_NUMBER() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) as row_num
FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_name, p.unit_price DESC;
```

Результат: отримано ренжовані товари за ціною в межах категорії.

<img width="1190" height="460" alt="5 3" src="https://github.com/user-attachments/assets/a02612f3-b15e-4429-a83f-0e422632000a" />




### 5.4 Порівняти замовлення кожного клієнта з попереднім за датою:

```sql
SELECT
    customer_id,
    order_id,
    order_date,
    freight,
    LAG(order_date) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS previous_order_date,
    LAG(freight) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS previous_order_amount
FROM orders
ORDER BY customer_id, order_date;
```

Результат: отримано порівняні замовлення кожного клієнта з попереднім за датою.

<img width="743" height="478" alt="5 4" src="https://github.com/user-attachments/assets/7bdb9b19-396d-4375-9b74-de63e6757521" />




### 6.1 Створити матеріалізоване представлення для аналізу продажів:


```sql
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT
    EXTRACT(YEAR FROM o.order_date::date) as year,
    EXTRACT(MONTH FROM o.order_date::date) as month,
    c.category_name,
    r.region_name,
    SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) as total_revenue,
    COUNT(DISTINCT o.order_id) as orders_count,
    AVG(oi.quantity * oi.unit_price * (1 - oi.discount)) as avg_order_value
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
JOIN customers cu ON o.customer_id = cu.customer_id
LEFT JOIN regions r ON cu.region_id = r.region_id
WHERE o.order_status = 'delivered'
GROUP BY year, month, c.category_name, r.region_name;

-- Створення індексу для матеріалізованого представлення
CREATE INDEX idx_mv_monthly_sales_date ON mv_monthly_sales(year, month);

```

Результат: отримано матеріалізоване представлення для аналізу продажів.

<img width="733" height="459" alt="6 1 1" src="https://github.com/user-attachments/assets/84da9211-4616-4980-b51e-782b3c1f6fc0" />

<img width="1470" height="903" alt="6 1 2" src="https://github.com/user-attachments/assets/ee3b40c5-dea0-40ce-af48-10bc4cfc2052" />




### 6.2 Реалізувати рекурсивний запит для ієрархії керівників:

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Базовий випадок: топ-менеджери (без керівника)
    SELECT 
        employee_id, 
        first_name, 
        last_name, 
        title, 
        reports_to,
        0 as level,
        CAST(last_name || ' ' || first_name AS VARCHAR(1000)) as hierarchy_path
    FROM employees
    WHERE reports_to IS NULL

    UNION ALL

    -- Рекурсивний випадок: підлеглі
    SELECT 
        e.employee_id, 
        e.first_name, 
        e.last_name, 
        e.title, 
        e.reports_to,
        eh.level + 1,
        CAST(eh.hierarchy_path || ' -> ' || e.last_name || ' ' || e.first_name AS VARCHAR(1000))
    FROM employees e
    JOIN employee_hierarchy eh 
        ON e.reports_to::bigint = eh.employee_id
)
SELECT * FROM employee_hierarchy
ORDER BY hierarchy_path;

```

Результат: отримано рекурсивний запит для ієрархії керівників.

<img width="1208" height="471" alt="6 2" src="https://github.com/user-attachments/assets/14bbd8e9-a434-4071-a13f-e01c3341aa4e" />




### 7.1

<img width="804" height="907" alt="7 1 1" src="https://github.com/user-attachments/assets/3f3641f6-fb2b-4bb2-8156-bc88b3c4b8e0" />
<img width="796" height="904" alt="7 1 2" src="https://github.com/user-attachments/assets/4d817bca-67bc-4b3e-9a4e-fd9f62aa0759" />
<img width="805" height="915" alt="7 1 3" src="https://github.com/user-attachments/assets/31fd1f30-1ff0-43bc-9d00-e3bf04cf3aa2" />




### 7.2

```sql
SELECT p.product_name, p.unit_price, c.category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE p.unit_price > (
    SELECT AVG(p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.category_name, p.unit_price DESC;
```
Можливі проблеми:

Підзапит для кожного рядка у products → може бути повільно при великій кількості товарів.




Ранжування та віконні функції
```sql
RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) as price_rank,
```
Можливі проблеми:

Віконні функції можуть бути дорогими на великих таблицях, особливо з ORDER BY у PARTITION.

Якщо таблиця products велика, варто додати індекс на (category_id, unit_price DESC).





Рекурсивний CTE
```sql
WITH RECURSIVE employee_hierarchy AS (...)
```


Можливі проблеми:

Рекурсія повільна, якщо багато співробітників і глибока ієрархія.

Перевіряти через EXPLAIN ANALYZE.



### 7.3

        1Таблиці products, categories, suppliers

Для JOIN products → categories та products → suppliers:
```sql
CREATE INDEX idx_products_category_id ON products(category_id);
CREATE INDEX idx_products_supplier_id ON products(supplier_id);
CREATE INDEX idx_categories_category_id ON categories(category_id);
CREATE INDEX idx_suppliers_supplier_id ON suppliers(supplier_id);
```

Для сортування та швидкого доступу по unit_price:
```sql
CREATE INDEX idx_products_unit_price ON products(unit_price);
```

        2Таблиці orders, order_items, customers, regions

Для JOIN і агрегацій:
```sql
-- orders → customers
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- orders → order_date (для EXTRACT/YEAR)
CREATE INDEX idx_orders_order_date ON orders(order_date);

-- order_items → orders
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- order_items → products
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

-- customers → regions
CREATE INDEX idx_customers_region_id ON customers(region_id);

-- regions → region_name (для GROUP BY/ORDER BY)
CREATE INDEX idx_regions_region_name ON regions(region_name);
```

        3Self-join та рекурсивна ієрархія співробітників

Для employees:
```sql
CREATE INDEX idx_employees_employee_id ON employees(employee_id);
CREATE INDEX idx_employees_reports_to ON employees(reports_to);

-- Для сортування та швидкого доступу по last_name/first_name
CREATE INDEX idx_employees_name ON employees(last_name, first_name);
```

        4Віконні функції (RANK, ROW_NUMBER)

Для ранжування товарів за ціною в межах категорії:
```sql
CREATE INDEX idx_products_category_price ON products(category_id, unit_price DESC);
```

        5Матеріалізоване представлення mv_monthly_sales
```sql
-- Для швидкого фільтрування по даті
CREATE INDEX idx_mv_monthly_sales_date ON mv_monthly_sales(year, month);

-- Для категорії та регіону
CREATE INDEX idx_mv_monthly_sales_category_region ON mv_monthly_sales(category_name, region_name);

-- Для швидкого сортування по доходу
CREATE INDEX idx_mv_monthly_sales_total_revenue ON mv_monthly_sales(total_revenue DESC);
```

        6Додаткові індекси для часто використовуваних колонок

```sql
orders.order_status – якщо фільтруєш по статусу:

CREATE INDEX idx_orders_status ON orders(order_status);


products.product_name – для швидкого ORDER BY / LIKE-пошуку:

CREATE INDEX idx_products_name ON products(product_name);


customers.contact_name – для швидкого сортування:

CREATE INDEX idx_customers_contact_name ON customers(contact_name);
```



## Висновки

**Самооцінка**: Заслуговую на 4+

**Обгрунтування**: Я зробив завдання на 1, 2 та 3 рівень, але із запізненням скору здачі, що пітверджує оцінку не більше 4. Я зрозумів принцип роботи легких і складних запитів та можу ефективно використовувати їх самостійно, без сторонньої допомоги. Крім того, я дотримувався вимог щодо читабельності коду та оформлення звіту, і вважаю, що виконані роботи відповідають критеріям для отримання високого балу.
