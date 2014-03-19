# SQL cheat sheet

A highly opinionated SQL cheat sheet:

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
