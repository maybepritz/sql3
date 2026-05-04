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
SELECT С.ФИО, НЗ.Код_начисления, НЗ.Дата_зарплаты, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
```
<img width="509" height="341" alt="{8DEAA97A-9508-45BE-B1C5-BDE2EBC93F5B}" src="https://github.com/user-attachments/assets/c87210ef-600a-479e-9184-65bf789c77fb" />


##### После курсора

```sql
-- Динамический курсор видит изменения в реальном времени,
-- поэтому повторная выборка той же строки вернёт уже обновлённое значение
SELECT С.ФИО, НЗ.Код_начисления, НЗ.Дата_зарплаты, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
```
<img width="507" height="345" alt="{7EF1FD0E-D473-491E-B85E-635CCBC32496}" src="https://github.com/user-attachments/assets/005a2125-d653-4735-9c3c-7914b6ebfdf4" />

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
<img width="507" height="345" alt="{9B9A488C-0B42-4DFB-B8E6-9CC07C95C85F}" src="https://github.com/user-attachments/assets/78c3a8b8-2918-4df7-bf63-ac32abcc7562" />

##### После курсора

```sql
-- Статический курсор работает со снимком данных на момент открытия,
-- поэтому сам курсор вернёт старые значения, но таблица будет обновлена
SELECT С.ФИО, НЗ.Итого_зарплаты
FROM Начисление_зарплаты НЗ
JOIN Сотрудник С ON НЗ.Табельный_номер = С.Табельный_номер
WHERE С.ФИО = 'Алексеев Алексей Алексеевич'
```
<img width="507" height="345" alt="{9B9A488C-0B42-4DFB-B8E6-9CC07C95C85F}" src="https://github.com/user-attachments/assets/78c3a8b8-2918-4df7-bf63-ac32abcc7562" />

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


#### Проверка

##### До курсора

```sql
SELECT С.ФИО, ШР.Название_должности, С.Разряд, ШР.Оклад
FROM Сотрудник AS С
JOIN Штатное_расписание AS ШР ON С.Код_должности = ШР.Код_должности
```
<img width="478" height="230" alt="{8F5E1EA6-76F2-4DAF-B8C3-AAA8777573AE}" src="https://github.com/user-attachments/assets/e49acc25-eac1-4565-aca0-fe9610bae60a" />

##### После курсора

```sql
-- Статический курсор работает со снимком данных на момент открытия,
-- поэтому сам курсор вернёт старые значения, но таблица будет обновлена
SELECT С.ФИО, ШР.Название_должности, С.Разряд, ШР.Оклад
FROM Сотрудник AS С
JOIN Штатное_расписание AS ШР ON С.Код_должности = ШР.Код_должности
```
<img width="484" height="230" alt="{B5AFE5DE-0B37-41A2-A3BB-09622BC5D1AD}" src="https://github.com/user-attachments/assets/bd2f6b46-b097-4e3d-ac87-cd37bf5969d4" />


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
<img width="410" height="228" alt="{30217967-0635-4B63-A0D0-606E9727CFCE}" src="https://github.com/user-attachments/assets/3e5137c9-4eb8-4240-bf59-ad15280b3654" />


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
<img width="344" height="345" alt="{8531AE6F-6ACC-4246-AD9C-9C2D159C01B2}" src="https://github.com/user-attachments/assets/40f1a42a-79c1-4f73-8967-92a368f13990" />


---

### Вид из вида

```sql
CREATE VIEW Получатели_в_день AS
    SELECT С.ФИО AS Имя, НЗ.Итого_зарплаты AS Сумма
    FROM Инфо_о_зарплате
    WHERE День IN ('2026-02-20')
```

#### Проверка

```sql
SELECT * FROM Получатели_в_день
```
<img width="267" height="117" alt="{5361BF3C-75DF-4B55-B460-08296DA0D189}" src="https://github.com/user-attachments/assets/a09bffd4-daec-40fb-8d93-c7f61ce852ed" />

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
<img width="552" height="96" alt="{90A409B6-F42D-4CA9-8DF9-A586D90A3A9C}" src="https://github.com/user-attachments/assets/0dadf767-c6bc-47c2-9d0e-b2ff489e0bac" />


##### Изменение должности сотрудника через вид

```sql
UPDATE Программисты
SET Код_должности = 4
WHERE Табельный_номер = 8

SELECT * FROM Программисты
```
<img width="552" height="77" alt="{F1656506-0632-46AE-8ADC-345A73F3AA8D}" src="https://github.com/user-attachments/assets/f832f3f5-989c-4a16-ad3f-744c2c15fd9d" />


##### Вставка сотрудника с должностью не программиста

```sql
INSERT INTO Сотрудник VALUES (12, 'НеИванов Иван Иваныч', 4, 6, 0)

SELECT * FROM Программисты
```
<img width="551" height="73" alt="{AFEE37CA-6BF7-4557-8523-A8C50C1ADF45}" src="https://github.com/user-attachments/assets/f49a4400-900a-4fb0-aafa-1de01ac89880" />

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
WHERE Табельный_номер = 7
```
<img width="1527" height="82" alt="{17A214A5-CFCC-4A5B-8339-3BD01353AD21}" src="https://github.com/user-attachments/assets/6b248973-a8bf-41ba-b8ba-a773cd4b00c0" />

##### Попытка вставить сотрудника с должностью не программиста

```sql
-- Заблокировано CHECK OPTION
INSERT INTO Программисты VALUES (14, 'НеНеНеИванов Иван Иваныч', 4, 6, 0)
```
<img width="1527" height="86" alt="{DE4A47A3-2CD5-4455-BD9D-EE0D2C9293D5}" src="https://github.com/user-attachments/assets/78e39fdd-430c-4877-a5cd-80c6cffb1753" />

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
<img width="382" height="70" alt="{7820D141-5E25-4792-AE83-15949E475B95}" src="https://github.com/user-attachments/assets/6875b0f6-411b-4ee8-bc60-abe9cc5f07b6" />


##### Ключевой вызов

```sql
DECLARE @РезультатЗП MONEY
EXEC ВывестиЗП
    @Фамилия  = 'Иванов',
    @Дата     = '2026-03-27',
    @Зарплата = @РезультатЗП OUTPUT
PRINT @РезультатЗП
```
<img width="379" height="74" alt="{6258D567-0E51-47AF-B503-93FD712C085B}" src="https://github.com/user-attachments/assets/dfb3407b-010e-4acf-8194-34b39b1c895a" />


##### Смешанный вызов

```sql
DECLARE @РезультатЗП MONEY
EXEC ВывестиЗП @Фамилия = 'Иванов', @РезультатЗП OUTPUT
PRINT @РезультатЗП
```
<img width="373" height="63" alt="{C29F8ABA-F9F6-4893-BD81-53DC4421AB29}" src="https://github.com/user-attachments/assets/2ba8c93d-61f4-4b9a-9248-4a8841b71dd2" />

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
<img width="183" height="94" alt="{E9F67CD1-BD17-4924-9319-519B9850A1D6}" src="https://github.com/user-attachments/assets/1bb7de09-e7e9-45a1-ad65-f77d4a50f4bb" />

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
<img width="569" height="345" alt="{F482CC72-70B3-41D6-9A03-61C207019BF4}" src="https://github.com/user-attachments/assets/62228b06-b63c-4fe4-bd65-2ad22a1a412e" />

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
<img width="794" height="116" alt="{ACC24DED-DF18-4F7E-91CD-A320658DA70F}" src="https://github.com/user-attachments/assets/ea557ad6-1271-418d-a927-f4d5bc80e3c3" />


##### Удаление пустого отдела

```sql
INSERT INTO Отдел VALUES (5, 'Пустой отдел')
DELETE FROM Отдел WHERE Код_отдела = 5
```
<img width="406" height="109" alt="{DCE58142-EE5F-4435-8154-DC92F14D7F98}" src="https://github.com/user-attachments/assets/29d1f9e1-2334-4ac2-949f-e0103a0027d7" />

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
<img width="392" height="120" alt="{E6187286-821F-4ACD-961F-3905F3918915}" src="https://github.com/user-attachments/assets/0a912b1c-1fea-4882-b28d-af42ec8d39ad" />

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
<img width="403" height="93" alt="{82993B20-9397-4DE6-8E7F-92CA2FE5C82A}" src="https://github.com/user-attachments/assets/9ed8a9f9-6605-4740-acc4-0dc30ca8609a" />

##### INSERT с дублирующим ФИО

```sql
-- Заблокировано триггером
INSERT INTO Сотрудник2 VALUES (6, 'Иванов Иван', 1)
```
<img width="777" height="117" alt="{C036B08A-0A5C-4BD7-945A-D08C0DE41597}" src="https://github.com/user-attachments/assets/69ca3576-58d4-4ad2-8ee7-d6fcbffe8b92" />

##### UPDATE на уникальное ФИО

```sql
UPDATE Сотрудник2 SET ФИО = 'Морозов Максим' WHERE Табельный_номер = 4
```
<img width="391" height="89" alt="{54511BD8-FE07-485D-BDA3-5ECDCFE48F93}" src="https://github.com/user-attachments/assets/8900ec8d-6712-4739-b000-8d2c1daf4ef6" />

##### UPDATE на уже существующее ФИО

```sql
-- Заблокировано триггером
UPDATE Сотрудник2 SET ФИО = 'Иванов Иван' WHERE Табельный_номер = 3
```
<img width="774" height="115" alt="{817FB019-DEBD-4B20-B5CF-1F0DC925756C}" src="https://github.com/user-attachments/assets/50e25cfb-8c03-4155-91a7-58a52793d818" />

