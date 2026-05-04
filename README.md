# Лабораторная работа: Курсоры, Виды, Процедуры, Функции, Триггеры

---

## 1. Курсоры

### Динамический курсор

```sql
DECLARE dynamic_cursor CURSOR SCROLL DYNAMIC
FOR
    SELECT С.ФИО, НЗ.Код_начисления, НЗ.Дата_зарплаты, НЗ.Итого_зарплаты
    FROM Начисление_зарплаты AS НЗ
    JOIN Сотрудник AS С ON НЗ.Табельный_номер = С.Табельный_номер

OPEN dynamic_cursor

DECLARE @name         VARCHAR(100),
        @salarycode   INT,
        @salarydate   DATE,
        @salaryamount MONEY

FETCH FIRST FROM dynamic_cursor INTO @name, @salarycode, @salarydate, @salaryamount

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @name = 'Алексеев Алексей Алексеевич'
    BEGIN
        UPDATE Начисление_зарплаты
        SET Итого_Зарплаты = Итого_зарплаты * 1.2
        WHERE Код_начисления = @salarycode
    END

    SELECT
        ФИО            = @name,
        Код_начисления = @salarycode,
        Дата           = @salarydate,
        Итого          = @salaryamount

    FETCH NEXT FROM dynamic_cursor INTO @name, @salarycode, @salarydate, @salaryamount
END

CLOSE dynamic_cursor
DEALLOCATE dynamic_cursor
```

#### Проверка

##### До курсора

```sql
SELECT С.ФИО, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
WHERE С.ФИО = 'Алексеев Алексей Алексеевич'
```

##### После курсора

```sql
-- Динамический курсор видит изменения в реальном времени,
-- поэтому повторная выборка той же строки вернёт уже обновлённое значение
SELECT С.ФИО, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
WHERE С.ФИО = 'Алексеев Алексей Алексеевич'
```

---

### Статический курсор

```sql
DECLARE static_cursor CURSOR SCROLL STATIC
FOR
    SELECT С.ФИО, НЗ.Код_начисления, НЗ.Дата_зарплаты, НЗ.Итого_зарплаты
    FROM Начисление_зарплаты AS НЗ
    JOIN Сотрудник AS С ON НЗ.Табельный_номер = С.Табельный_номер

OPEN static_cursor

DECLARE @name         VARCHAR(100),
        @salarycode   INT,
        @salarydate   DATE,
        @salaryamount MONEY

FETCH FIRST FROM static_cursor INTO @name, @salarycode, @salarydate, @salaryamount

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @name = 'Алексеев Алексей Алексеевич'
    BEGIN
        UPDATE Начисление_зарплаты
        SET Итого_Зарплаты = Итого_зарплаты * 1.2
        WHERE Код_начисления = @salarycode
    END

    SELECT
        ФИО            = @name,
        Код_начисления = @salarycode,
        Дата           = @salarydate,
        Итого          = @salaryamount

    FETCH NEXT FROM static_cursor INTO @name, @salarycode, @salarydate, @salaryamount
END

CLOSE static_cursor
DEALLOCATE static_cursor
```

#### Проверка

##### До курсора

```sql
SELECT С.ФИО, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
WHERE С.ФИО = 'Алексеев Алексей Алексеевич'
```

##### После курсора

```sql
-- Статический курсор работает со снимком данных на момент открытия,
-- поэтому сам курсор вернёт старые значения, но таблица будет обновлена
SELECT С.ФИО, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
WHERE С.ФИО = 'Алексеев Алексей Алексеевич'
```

---

### Ключевой курсор

```sql
DECLARE key_cursor CURSOR SCROLL KEYSET
FOR
    SELECT С.ФИО, ШР.Название_должности, С.Разряд, ШР.Оклад
    FROM Сотрудник AS С
    JOIN Штатное_расписание AS ШР ON С.Код_должности = ШР.Код_должности

OPEN key_cursor

DECLARE @name    VARCHAR(100),
        @jobname VARCHAR(50),
        @rank    TINYINT,
        @salary  MONEY

FETCH FIRST FROM key_cursor INTO @name, @jobname, @rank, @salary

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @jobname = 'Бухгалтер'
    BEGIN
        UPDATE Штатное_расписание
        SET Оклад = Оклад * 1.15
        WHERE Название_должности = @jobname
    END

    IF @jobname = 'Охранник'
    BEGIN
        DELETE FROM Сотрудник
        WHERE Табельный_номер = (
            SELECT Табельный_номер
            FROM Сотрудник AS С
            JOIN Штатное_расписание AS ШР ON С.Код_должности = ШР.Код_должности
            WHERE ШР.Название_должности = 'Охранник'
        )
    END

    FETCH NEXT FROM key_cursor INTO @name, @jobname, @rank, @salary
END

CLOSE key_cursor
DEALLOCATE key_cursor
```

---

## 2. Виды (Views)

### Простой вид

```sql
CREATE VIEW Сотрудники_с_льготами AS
    SELECT Табельный_номер, ФИО, Наличие_льгот
    FROM Сотрудник
```

#### Проверка

```sql
SELECT * FROM Сотрудники_с_льготами
```

---

### Составной вид

```sql
CREATE VIEW Инфо_о_зарплате AS
    SELECT
        С.ФИО             AS Имя,
        НЗ.Дата_зарплаты  AS День,
        НЗ.Итого_зарплаты AS Сумма
    FROM Начисление_зарплаты AS НЗ
    JOIN Сотрудник AS С ON НЗ.Табельный_номер = С.Табельный_номер
```

#### Проверка

```sql
SELECT Имя, День, Сумма FROM Инфо_о_зарплате
```

---

### Вид из вида

```sql
CREATE VIEW Получатели_в_день AS
    SELECT С.ФИО AS Имя, НЗ.Итого_зарплаты AS Сумма
    FROM Инфо_о_зарплате
    WHERE День IN ('2026-03-26')
```

#### Проверка

```sql
SELECT * FROM Получатели_в_день
```

---

### Вид без `WITH CHECK OPTION`

```sql
CREATE VIEW Программисты AS
    SELECT С.*
    FROM Сотрудник AS С
    JOIN Штатное_расписание AS ШР ON С.Код_должности = ШР.Код_должности
    WHERE ШР.Название_должности = 'Программист'
```

#### Проверка

##### Исходное состояние вида

```sql
SELECT * FROM Программисты
```

##### Изменение должности сотрудника через вид

```sql
UPDATE Программисты
SET Код_должности = 4
WHERE Табельный_номер = 9

SELECT * FROM Программисты
```

##### Вставка сотрудника с должностью не программиста

```sql
INSERT INTO Сотрудник VALUES (12, 'НеИванов Иван Иваныч', 4, 6, 0)

SELECT * FROM Программисты
```

---

### Вид с `WITH CHECK OPTION`

```sql
CREATE VIEW Программисты AS
    SELECT С.*
    FROM Сотрудник AS С
    JOIN Штатное_расписание AS ШР ON С.Код_должности = ШР.Код_должности
    WHERE ШР.Название_должности = 'Программист'
    WITH CHECK OPTION
```

#### Проверка

##### Попытка изменить должность через вид

```sql
-- Заблокировано CHECK OPTION
UPDATE Программисты
SET Код_должности = 4
WHERE Табельный_номер = 9
```

##### Попытка вставить сотрудника с должностью не программиста

```sql
-- Заблокировано CHECK OPTION
INSERT INTO Сотрудник VALUES (12, 'НеИванов Иван Иваныч', 4, 6, 0)
```

---

## 3. Хранимые процедуры и функции

### Хранимая процедура с входным, выходным и дефолтным параметром

```sql
CREATE PROCEDURE ВывестиЗП(
    @Фамилия  VARCHAR(50),
    @Зарплата MONEY OUTPUT,
    @Дата     DATE = '2026-03-25'
)
AS
BEGIN
    SELECT @Зарплата = НЗ.Итого_зарплаты
    FROM Начисление_зарплаты AS НЗ
    JOIN Сотрудник AS С ON С.Табельный_номер = НЗ.Табельный_номер
    WHERE (ФИО LIKE @Фамилия + '%')
      AND (НЗ.Дата_зарплаты = @Дата)
END
```

#### Проверка

##### Позиционный вызов

```sql
DECLARE @РезультатЗП MONEY
EXEC ВывестиЗП 'Иванов', @РезультатЗП OUTPUT
PRINT @РезультатЗП
```

##### Ключевой вызов

```sql
DECLARE @РезультатЗП MONEY
EXEC ВывестиЗП
    @Фамилия  = 'Иванов',
    @Дата     = '2026-03-27',
    @Зарплата = @РезультатЗП OUTPUT
PRINT @РезультатЗП
```

##### Смешанный вызов

```sql
DECLARE @РезультатЗП MONEY
EXEC ВывестиЗП @Фамилия = 'Иванов', @РезультатЗП OUTPUT
PRINT @РезультатЗП
```

---

### Скалярная функция

```sql
CREATE FUNCTION Должность(@ФИО_Сотрудника VARCHAR(100))
RETURNS VARCHAR(50)
AS
BEGIN
    DECLARE @Должность VARCHAR(50)

    SET @Должность = (
        SELECT Название_должности
        FROM Штатное_расписание AS ШР
        JOIN Сотрудник AS С ON ШР.Код_должности = С.Код_должности
        WHERE С.ФИО = @ФИО_Сотрудника
    )

    RETURN @Должность
END
```

#### Проверка

```sql
SELECT dbo.Должность('Иванов Иван Иванович') AS Должность
```

---

### Табличная функция

```sql
CREATE FUNCTION ЗпМесяц(@Номер_месяца INT)
RETURNS TABLE
AS RETURN
(
    SELECT
        С.ФИО,
        ШР.Название_должности,
        НЗ.Итого_зарплаты,
        НЗ.Дата_зарплаты
    FROM Сотрудник AS С
    JOIN Штатное_расписание AS ШР ON ШР.Код_должности = С.Код_должности
    JOIN Начисление_зарплаты AS НЗ ON НЗ.Табельный_номер = С.Табельный_номер
    WHERE MONTH(НЗ.Дата_зарплаты) = @Номер_месяца
)
```

#### Проверка

```sql
SELECT * FROM dbo.ЗпМесяц(3)
```

---

## 4. Триггеры

### Подготовка: вспомогательные таблицы

```sql
CREATE TABLE Отдел (
    Код_отдела INT PRIMARY KEY,
    Название   VARCHAR(50)
)

CREATE TABLE Сотрудник2 (
    Табельный_номер INT PRIMARY KEY,
    ФИО             VARCHAR(100),
    Код_отдела      INT
)

INSERT INTO Отдел VALUES (1, 'Бухгалтерия')
INSERT INTO Отдел VALUES (2, 'IT-отдел')
INSERT INTO Отдел VALUES (3, 'Кадры')

INSERT INTO Сотрудник2 VALUES (1, 'Иванов Иван',   1)
INSERT INTO Сотрудник2 VALUES (2, 'Петров Пётр',   1)
INSERT INTO Сотрудник2 VALUES (3, 'Сидоров Сидор', 2)
INSERT INTO Сотрудник2 VALUES (4, 'Козлов Андрей', 3)
```

---

### Триггер RESTRICT

```sql
CREATE TRIGGER RESTRICT_TRG
ON Отдел
FOR DELETE
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM Сотрудник2 AS С
        JOIN deleted AS d ON С.Код_отдела = d.Код_отдела
    )
    BEGIN
        RAISERROR('Удаление запрещено: в отделе есть сотрудники!', 16, 10)
        ROLLBACK TRAN
    END
    RETURN
END
```

#### Проверка

##### Удаление отдела с сотрудниками

```sql
-- Заблокировано триггером
DELETE FROM Отдел WHERE Код_отдела = 1
```

##### Удаление пустого отдела

```sql
INSERT INTO Отдел VALUES (5, 'Пустой отдел')
DELETE FROM Отдел WHERE Код_отдела = 5
```

---

### Триггер CASCADE

```sql
CREATE TRIGGER CASCADE_TRG
ON Отдел
FOR DELETE
AS
BEGIN
    DELETE FROM Сотрудник2
    WHERE Код_отдела IN (SELECT Код_отдела FROM DELETED)
    RETURN
END
```

#### Проверка

##### Удаление отдела вместе с сотрудниками

```sql
DELETE FROM Отдел WHERE Код_отдела = 1
```

---

### Триггер уникальности поля `ФИО`

```sql
CREATE TRIGGER unique_name
ON Сотрудник2
FOR UPDATE, INSERT
AS
BEGIN
    IF UPDATE(ФИО)
       AND (SELECT COUNT(*) FROM Сотрудник2 AS С JOIN INSERTED AS I ON С.ФИО = I.ФИО) > 1
    BEGIN
        RAISERROR('Изменение запрещено: сотрудник с таким ФИО уже есть!', 16, 10)
        ROLLBACK TRAN
    END
    RETURN
END
```

#### Проверка

##### INSERT с уникальным ФИО

```sql
INSERT INTO Сотрудник2 VALUES (5, 'Новиков Николай', 2)
```

##### INSERT с дублирующим ФИО

```sql
-- Заблокировано триггером
INSERT INTO Сотрудник2 VALUES (6, 'Иванов Иван', 1)
```

##### UPDATE на уникальное ФИО

```sql
UPDATE Сотрудник2 SET ФИО = 'Морозов Максим' WHERE Табельный_номер = 4
```

##### UPDATE на уже существующее ФИО

```sql
-- Заблокировано триггером
UPDATE Сотрудник2 SET ФИО = 'Иванов Иван' WHERE Табельный_номер = 3
```
