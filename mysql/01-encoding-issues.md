# MySQL and encoding issues

## Check insertion with foreign encoding

    # Example of <expected_encoding>: utf8
    # Example of <foreign_encoding>: latin1
    
    SELECT
        CONVERT(CONVERT(<column_name> USING BINARY) USING <foreign_encoding>) AS <foreign_encoding>,
        CONVERT(CONVERT(<column_name> USING BINARY) USING <expected_encoding>) AS <expected_encoding>
    FROM <table_name>
    WHERE CONVERT(<column_name> USING BINARY) RLIKE CONCAT('[', UNHEX('80'), '-', UNHEX('FF'), ']')
    LIMIT 0,25; # To avoid listing very long tables.

Source: [StackOverflow - How to detect UTF-8 characters in a latin1 encoded column](http://stackoverflow.com/questions/9304485/how-to-detect-utf-8-characters-in-a-latin1-encoded-column-mysql/9305294#9305294)

## Dumping and importing

Dumping when encoding is good:

    shell> mysqldump --user=<name> -p <database> -r <dump-file>

Dumping when encoding is bad (e.g. latin1 inserted in utf8 columns):

    shell> mysqldump --user=<name> -p <database> --skip-set-charset --default-character-set=<foreign-encoding> -r <dump-file>

Importing:

    shell> mysql --user=<name> -p  --default-character-set=<expected-encoding> <database>
    mysql> SOURCE <dump-file>

Source: [MakandraCards - Dumping and importing from/to MySQL in an UTF-8 safe way](http://makandracards.com/makandra/595-dumping-and-importing-from-to-mysql-in-an-utf-8-safe-way)


