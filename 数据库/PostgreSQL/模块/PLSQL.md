# 存储过程

~~~sql
-- 创建存储过程
CREATE OR REPLACE PROCEDURE calculate_avg_salary()
LANGUAGE plpgsql
AS $$
DECLARE
  avg_salary NUMERIC;
BEGIN
  -- 计算平均工资
  SELECT AVG(salary) INTO avg_salary
  FROM employees;

  -- 打印结果
  RAISE NOTICE '平均工资为：%s', avg_salary;
END;
$$;

-- 调用存储过程
CALL calculate_avg_salary();
~~~

# 函数

**用法**

~~~sql
CREATE OR REPLACE FUNCTION insert_record()
RETURNS TRIGGER AS
$$
BEGIN
  IF NEW."ID" % 100 = 0 THEN
    INSERT INTO record_time (id, in_time)
    VALUES (NEW."ID", current_timestamp);
  END IF;
  RETURN NEW;
END;
$$
LANGUAGE plpgsql;
~~~

上面是一个返回触发器的函数，因此可以调用NEW代指插入的一条数据

如果是一般的函数，可以这样写

~~~sql
sql
-- 创建函数
CREATE OR REPLACE FUNCTION calculate_avg_salary()
RETURNS NUMERIC AS $$
DECLARE
  avg_salary NUMERIC;
BEGIN
  -- 计算平均工资
  SELECT AVG(salary) INTO avg_salary
  FROM employees;

  RETURN avg_salary;
END;
$$ LANGUAGE plpgsql;

-- 调用函数
SELECT calculate_avg_salary();
~~~

# 触发器

**用法**

~~~sql
CREATE TRIGGER trg_commit_timestamp
AFTER INSERT ON ta
FOR EACH ROW
EXECUTE FUNCTION insert_record();
~~~



