# SQL cheat sheet

A highly opinionated SQL cheat sheet:

* [Insert](#insert)
* [Update](#update)
* [Delete](#delete)
* [Alter](#alter)
  * [Add column](#add-column)
  * [drop column](#drop-column)

## Insert

    INSERT INTO table_name (column1, column2, column3,...)
    VALUES (value1, value2, value3,...)

Short form:

    INSERT INTO table_name
    VALUES (value1, value2, value3,...)

## Update

    UPDATE table_name
    SET column1=value, column2=value2,...
    WHERE some_column=some_value

## Delete

    DELETE FROM table_name
    WHERE some_column = some_value

## Alter

### Add column

    ALTER TABLE table_name
    ADD column_name datatype

### Drop column

    ALTER TABLE table_name
    DROP COLUMN column_name
