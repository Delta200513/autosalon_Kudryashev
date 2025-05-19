# Курсовая по теме: база данных для автосалона
---
# Типовые запросы

1. **Вывести модель автомобиля с самой высокой базовой ценой**

```sql
SELECT id, name, base_price
FROM car_models
ORDER BY base_price DESC
LIMIT 1;
````

2. **Получить список заказов клиента с id=1 с деталями моделей и датами**

```sql
SELECT o.id AS order_id, o.order_date, cm.name AS car_model
FROM orders o
JOIN car_models cm ON o.car_model_id = cm.id
WHERE o.customer_id = 1;
```

3. **Вывести количество моделей автомобилей по классам**

```sql
SELECT cc.name AS class_name, COUNT(cm.id) AS model_count
FROM car_classes cc
LEFT JOIN car_models cm ON cm.class_id = cc.id
GROUP BY cc.name;
```

4. **Вывести среднюю зарплату сотрудников по должностям**

```sql
SELECT position, AVG(salary) AS avg_salary
FROM employees
GROUP BY position;
```

---

# Хранимые процедуры

**Процедура для добавления новой модели автомобиля**

```sql
CREATE PROCEDURE AddCarModel(
    IN pname VARCHAR(100),
    IN pmanufacturer_id INT,
    IN pyear INT,
    IN pbase_price DECIMAL(10,2),
    IN pclass_id INT
)
BEGIN
    INSERT INTO car_models(name, manufacturer_id, year, base_price, class_id)
    VALUES (pname, pmanufacturer_id, pyear, pbase_price, pclass_id);
END //
```

**Вызов:**

```sql
CALL AddCarModel('Tesla Model S', 1, 2024, 9500000.00, 3);
```

*Процедура добавит в таблицу `car_models` новую модель автомобиля с заданными параметрами.*

---

# Триггер

**Пример триггера:**

```sql
CREATE TRIGGER update_car_model_count
AFTER INSERT ON car_models
FOR EACH ROW
BEGIN
    UPDATE car_classes
    SET name = name -- тут можно добавить логику подсчёта или обновления статистики
    WHERE id = NEW.class_id;
END //
```

*Триггер срабатывает после добавления новой модели автомобиля. (Пример демонстрационный, в реальном проекте нужно адаптировать логику.)*

---

# Функция

**Функция проверки наличия автомобиля в базе по ID**

```sql
CREATE FUNCTION IsCarModelExists(pid INT) RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE cnt INT;
    SELECT COUNT(*) INTO cnt FROM car_models WHERE id = pid;
    RETURN cnt > 0;
END //
```

---

# Представления

**Представление с подробной информацией о заказах с моделями и клиентами**

```sql
CREATE OR REPLACE VIEW detail_orders AS
SELECT
    o.id AS order_id,
    c.full_name AS customer_name,
    cm.name AS car_model,
    o.order_date
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
LEFT JOIN car_models cm ON o.car_model_id = cm.id;
```
---

# Роли

**Роль менеджера по продажам**

```sql
CREATE ROLE IF NOT EXISTS SalesManager;

GRANT SELECT, INSERT, UPDATE ON autosalon.orders TO SalesManager;
GRANT SELECT ON autosalon.customers TO SalesManager;
GRANT SELECT ON autosalon.car_models TO SalesManager;
GRANT EXECUTE ON PROCEDURE autosalon.AddCarModel TO SalesManager;
```
*Роль `SalesManager` позволяет менеджерам работать с заказами, просматривать клиентов и модели авто, а также добавлять новые модели.*
