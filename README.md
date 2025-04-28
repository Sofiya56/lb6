# SQL Project: Транзакції та курсори в T-SQL

## Опис завдань:
Цей проект містить скрипти для виконання практичних завдань з транзакціями, обробки помилок та роботи з курсорами в Microsoft SQL Server (T-SQL).

### Завдання:
1. **Транзакція з ROLLBACK** — якщо умова не виконана.
2. **Перевірка @@ERROR** — для керування транзакцією.
3. **TRY...CATCH** — для обробки помилок під час виконання транзакцій.
4. **Складні запити з JOIN та фільтруванням**.
5. **Використання індексів для прискорення запитів**.
6. **Курсори для обробки результатів запитів**.
7. **Функції для обчислення вартості товарів**.

### SQL файли:
- `SETUP.sql` — створення таблиць.
- `INSERT.sql` — вставка даних.
- `UPDATE.sql` — оновлення даних.
- `CUSOR.sql` — транзакції та обробка помилок.
- `QUERY.sql` — складні запити.
- `FUNCTIONS.sql` — функції для обчислень.


  
  
  SETUP.sql — створення таблиці
                                                                                                        
 -- Створення таблиці Products
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    ProductName NVARCHAR(100) NOT NULL,
    Category NVARCHAR(50),
    Price DECIMAL(10, 2) NOT NULL,
    StockQuantity INT NOT NULL,
    LastUpdated DATETIME DEFAULT GETDATE()
);

INSERT.sql — вставка 10 000 товарів
-- Додавання 10 000 товарів до таблиці
DECLARE @i INT = 1;

WHILE @i <= 10000
BEGIN
    INSERT INTO Products (ProductName, Category, Price, StockQuantity)
    VALUES (
        'Product' + CAST(@i AS NVARCHAR(10)),  -- Назва товару
        'Category' + CAST((@i % 10) + 1 AS NVARCHAR(10)),  -- Категорія товару (від 1 до 10)
        ROUND(RAND() * 100 + 1, 2),  -- Вартість товару (від 1 до 100)
        FLOOR(RAND() * 1000)  -- Кількість на складі (від 0 до 1000)
    );
    
    SET @i = @i + 1;
END

UPDATE.sql — оновлення товарів
-- Оновлення даних товарів (приклад)
BEGIN TRANSACTION;

UPDATE Products
SET Price = Price * 1.05  -- Збільшуємо ціну на 5%
WHERE StockQuantity > 500;  -- Товари з кількістю більше 500

IF @@ERROR <> 0
BEGIN
    ROLLBACK TRANSACTION;  -- Якщо сталася помилка, скасувати транзакцію
    PRINT 'Error occurred during the update.';
END
ELSE
BEGIN
    COMMIT TRANSACTION;  -- Якщо все пройшло успішно, зафіксувати зміни
    PRINT 'Update successful.';
END

CUSOR.sql — Транзакції з ROLLBACK, @@ERROR, TRY...CATCH

-- Завдання 3: Транзакція з ROLLBACK, якщо умова не виконана
BEGIN TRAN;
UPDATE authors SET au_fname = 'Ivan' WHERE au_id = '172-32-1176';
IF (SELECT COUNT(*) FROM authors WHERE city = 'NonExist') = 0 
    ROLLBACK;
ELSE
    COMMIT;

-- Завдання 4: Перевірка @@ERROR для керування транзакцією
BEGIN TRAN;
UPDATE authors SET au_fname = 'Oksana' WHERE au_id = '000-00-0000';
IF @@ERROR <> 0 
    ROLLBACK;
ELSE
    COMMIT;

-- Завдання 5: Використання TRY...CATCH у транзакціях
BEGIN TRAN;
BEGIN TRY
    UPDATE authors SET au_fname = 'Taras' WHERE au_id = '123-45-6789';
    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT ERROR_MESSAGE();
END CATCH;

QUERY.sql — Складні запити з використанням JOIN та фільтрування
-- Завдання 8: Запит з використанням JOIN
SELECT p.ProductID, p.ProductName, p.Price, c.CategoryName
FROM Products p
INNER JOIN Categories c ON p.CategoryID = c.CategoryID
WHERE p.StockQuantity > 100 AND c.CategoryName LIKE '%Electronics%'
ORDER BY p.Price DESC;

-- Завдання 9: Запит з індексами
CREATE INDEX idx_CategoryID ON Products(CategoryID);

SELECT p.ProductID, p.ProductName, p.Price, c.CategoryName
FROM Products p
INNER JOIN Categories c ON p.CategoryID = c.CategoryID
WHERE p.StockQuantity > 100 AND c.CategoryName LIKE '%Electronics%'
ORDER BY p.Price DESC;

-- Завдання 10: Використання курсорів для обробки результатів
DECLARE @ProductID INT, @ProductName NVARCHAR(100);
DECLARE product_cursor CURSOR FOR
SELECT ProductID, ProductName FROM Products WHERE StockQuantity > 100;

OPEN product_cursor;
FETCH NEXT FROM product_cursor INTO @ProductID, @ProductName;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Product ID: ' + CAST(@ProductID AS NVARCHAR) + ', Product Name: ' + @ProductName;
    FETCH NEXT FROM product_cursor INTO @ProductID, @ProductName;
END;

CLOSE product_cursor;
DEALLOCATE product_cursor;

FUNCTIONS.sql — Створення функцій (наприклад, для обчислення загальної вартості товарів)
-- Завдання 7: Створення функції для розрахунку вартості товарів
CREATE FUNCTION GetTotalPrice()
RETURNS DECIMAL(18, 2)
AS
BEGIN
    DECLARE @totalPrice DECIMAL(18, 2);
    SELECT @totalPrice = SUM(Price * StockQuantity) FROM Products;
    RETURN @totalPrice;
END;

-- Виклик функції
SELECT dbo.GetTotalPrice() AS TotalInventoryValue;



