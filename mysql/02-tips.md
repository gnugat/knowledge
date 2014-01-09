# MySQL tips

Force data removal, ignoring constraints:

    SET foreign_key_checks = 0;
    DROP TABLE <table-name>;
    TRUNCATE TABLE <table-name>;
    SET foreign_key_checks = 1;
