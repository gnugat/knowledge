# SQL cheat sheet

A highly opinionated SQL cheat sheet:

* [Create table](#create-table)
* [Insert](#insert)
* [Update](#update)
* [Delete](#delete)
* [Alter](#alter)
  * [Add column](#add-column)
  * [drop column](#drop-column)
* [Join](#join)

## Create table

```
CREATE TABLE table_name (
    column type(length),
    ...
)
```

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

## Join

![Join diagram](https://lukaseder.files.wordpress.com/2016/07/venn-join1.png)

[See](https://blog.jooq.org/2016/07/05/say-no-to-venn-diagrams-when-explaining-joins/)
