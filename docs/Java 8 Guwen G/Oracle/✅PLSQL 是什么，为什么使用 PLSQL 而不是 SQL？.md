**PL/SQL**（Procedural Language/SQL）是 Oracle 提供的一种过程化编程语言，它结合了 SQL 的数据操作功能和过程化语言的控制结构。**PL/SQL 可以用来编写复杂的数据库操作，提供了更高效的数据库程序设计和更强的控制能力。**



虽然 SQL 是数据库交互的标准语言，**<u>PL/SQL 也在很多情况下比 SQL 更有优势</u>**。以下是一些常见的使用 PL/SQL 的原因以及对应的实例：



#### **更强的逻辑控制能力**
SQL 是一种声明式语言，它通过声明查询或数据操作来告诉数据库该做什么，而没有内建的流程控制功能。对于需要复杂逻辑的任务，SQL 很难满足需求。而 **PL/SQL 提供了过程化的结构，如 **`**IF**`**、**`**LOOP**`**、**`**FOR**`** 循环、**`**WHILE**`** 循环、异常处理等，这使得开发者能够编写具有更复杂业务逻辑的程序。**

**举个例子**：

+ 如果你想检查某个条件，如果满足则执行特定操作，否则执行另一个操作，使用纯 SQL 语句比较困难，而在 PL/SQL 中可以非常轻松地使用 `IF-THEN-ELSE` 来实现。



```plain
DECLARE
  v_salary employees.salary%TYPE;
BEGIN
  -- 获取员工 1001 的薪水
  SELECT salary INTO v_salary FROM employees WHERE employee_id = 1001;

  -- 根据薪水条件执行操作
  IF v_salary > 5000 THEN
    UPDATE employees 
    SET salary = salary * 1.1 
    WHERE employee_id = 1001;
    DBMS_OUTPUT.PUT_LINE('Salary updated');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Salary too low for increase');
  END IF;
END;
```



#### **批量操作与性能优化**
使用 SQL 只能一次性处理一行数据或多个独立的 SQL 语句。如果你需要对大量数据进行处理，频繁地发出多次 SQL 请求会造成较高的开销，导致性能瓶颈。

**PL/SQL 支持批量处理（例如，通过循环语句操作多行数据）**，并且允许通过 `FORALL` 语句执行批量的 DML（数据操作语言）操作，从而减少数据库与应用之间的交互次数，提高性能。

**举个例子**：

+ 通过 PL/SQL，您可以批量插入或更新多条记录，而不必单独为每一条记录发出一个 SQL 请求。



```plain
DECLARE
  TYPE emp_rec IS RECORD (employee_id employees.employee_id%TYPE, salary employees.salary%TYPE);
  TYPE emp_table IS TABLE OF emp_rec;
  v_employees emp_table;
BEGIN
  -- 将临时表中的数据读取到 PL/SQL 表中
  SELECT employee_id, salary
  BULK COLLECT INTO v_employees
  FROM temp_employees;

  -- 使用 FORALL 批量插入数据
  FORALL i IN INDICES OF v_employees
    INSERT INTO employees (employee_id, salary)
    VALUES (v_employees(i).employee_id, v_employees(i).salary);
  
  COMMIT;
END;

```



#### **异常处理**
SQL 没有内建的异常处理机制。如果执行的查询或数据操作发生错误，SQL 会返回错误信息，但不会主动处理错误。而 P**L/SQL 提供了 **`**EXCEPTION**`** 块，可以捕获并处理各种异常，确保程序的健壮性和稳定性。**

**举个例子**：

+ 当你执行一个插入操作时，如果数据违反了约束（比如违反了唯一性约束），PL/SQL 可以通过 `EXCEPTION` 块捕获这个错误并进行处理（如记录日志或执行补救操作），而 SQL 只会返回一个错误，无法主动处理。



```plain
BEGIN
  -- 尝试插入数据
  INSERT INTO employees (employee_id, employee_name, salary)
  VALUES (1001, 'John Doe', 6000);
  
  COMMIT;
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN
    -- 如果违反唯一性约束，捕获异常并处理
    DBMS_OUTPUT.PUT_LINE('Error: Employee ID already exists.');
    ROLLBACK;  -- 回滚事务
  WHEN OTHERS THEN
    -- 捕获其他未知异常
    DBMS_OUTPUT.PUT_LINE('An unexpected error occurred.');
    ROLLBACK;  -- 回滚事务
END;

```



#### **事务控制**
虽然 SQL 也支持事务（`BEGIN TRANSACTION`, `COMMIT`, `ROLLBACK`），但 PL/SQL 在处理事务时更加灵活，能够在 PL/SQL 块内部根据不同的条件进行事务控制。

**举个例子**：

+ 在 PL/SQL 中，你可以在一个大的事务块中使用条件语句和循环，根据业务需求决定是否提交或回滚事务，确保数据的一致性和完整性。



```plain
BEGIN
  -- 执行某些操作
  UPDATE employees
  SET salary = salary * 1.1
  WHERE employee_id = 1001;
  
  -- 检查是否满足某个业务规则
  IF (SELECT salary FROM employees WHERE employee_id = 1001) > 5000 THEN
    COMMIT;  -- 如果薪水大于 5000，则提交事务
    DBMS_OUTPUT.PUT_LINE('Transaction committed.');
  ELSE
    ROLLBACK;  -- 否则回滚事务
    DBMS_OUTPUT.PUT_LINE('Transaction rolled back.');
  END IF;
END;
```



#### **代码重用与模块化**
PL/SQL 支持创建存储过程、函数和触发器等，这使得开发者可以将常用的代码封装在数据库中，便于重用和维护。而 SQL 本身不支持代码的封装和模块化。



```plain
CREATE OR REPLACE PROCEDURE increase_salary (
  p_percentage IN NUMBER
) IS
BEGIN
  UPDATE employees
  SET salary = salary * (1 + p_percentage / 100);
  COMMIT;
  DBMS_OUTPUT.PUT_LINE('Salary increase completed.');
END increase_salary;

```

# 扩展知识
## PL/SQL 与 SQL 的区别
虽然 PL/SQL 和 SQL 都用于与数据库交互，但它们有显著的区别：

| **特点** | **SQL** | **PL/SQL** |
| --- | --- | --- |
| **类型** | 声明式语言（Declarative Language） | 过程化语言（Procedural Language） |
| **功能** | 用于查询和操作数据 | 提供了控制流程、条件判断、循环、异常处理等功能 |
| **执行方式** | 每次执行一条 SQL 语句（单一的操作） | 可以执行多个 SQL 语句并处理复杂的逻辑流程 |
| **语言特性** | 不支持控制结构和逻辑操作（如循环、分支） | 支持变量、条件语句、循环、异常处理、事务控制等 |
| **适用场景** | 单纯的数据操作，如查询、插入、更新、删除等 | 复杂的逻辑处理、数据验证、批量处理等 |




