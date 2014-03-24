# bash cheat sheet

A highly opinionated bash cheat sheet.

## Conditions

Basics:

    #!/usr/bin/env bash
    
    if [ expr ] && [ expr ]; then
        echo 'and operator'
    elif [ expr ] || [ expr ]; then
        echo 'or operator'
    elif [ ! expr ]; then
        echo 'not operator'
    else
        echo 'third'
    fi

### String comparison

    #!/usr/bin/env bash
    
    if [[ $a == z* ]]; then
        echo 'starts with "z"'
    elif [ $a == "z*" ]; then
        echo 'is equal to "z*"'
    elif [ $a == z* ]; then
        echo 'file globbing and word splitting take place'
    elif [ "$a" != "$b" ]; then
        echo 'is not equal'
    elif [ -z '' ]; then
        echo 'is null, or empty'
    elif [ -n "$string1" ]; then
        echo 'is not null, or is initialized'
    fi

### Integer comparison

    #!/usr/bin/env bash
    
    if [ "$a" -eq "$b" ]; then
        echo 'is equal to'
    elif [ "$a" -ne "$b" ]; then
        echo 'is not equal to'
    elif [ "$a" -gt "$b" ]; then
        echo 'is greater than'
    elif [ "$a" -ge "$b" ]; then
        echo 'is greater than or equal to'
    elif [ "$a" -lt "$b" ]; then
        echo 'is less than'
    elif [ "$a" -le "$b" ]; then
        echo 'is less than or equal to'
    fi

### File checking

    #!/usr/bin/env bash
    
    if [ -e $file]; then
        echo 'File exists'
    elif [ -f $file ]; then
        echo 'is a regular file'
    elif [ -d $directory]; then
        echo 'is a directory'
    elif [ -L $symbolic_link ]; then
        echo 'is a symbolic link'
    elif [ -r $readable_file ]; then
        echo 'has read permission'
    elif [ -w $writable_file ]; then
        echo 'has write permission'
    elif [ -x $executable_file ]; then
        echo 'has execution permission'
    elif [ -O $your_file ]; then
        echo 'is owned by you'
    elif [ -N $edited_file ]; then
        echo 'modified since last read'
    elif [ $newer -nt $older ]; then
        echo 'is newer than'
    elif [ $older -ot $newer ]; then
        echo 'is older than'
    fi

## References

* [File test operators](http://tldp.org/LDP/abs/html/fto.html)
* [String and integer operators](http://tldp.org/LDP/abs/html/comparison-ops.html)
* [the bracket construct(http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS)]
