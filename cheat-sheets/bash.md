# bash cheat sheet

A highly opinionated bash cheat sheet.

* [Conditions](#conditions):
  * [string comparison](#string-comparison)
  * [integer comparison](#integer-comparison)
  * [file checking](#file-checking)
* [getopts](#getopts)

## Conditions

Basics:

```bash
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
```

### String comparison

```bash
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
```

### Integer comparison

```bash
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
```

### File checking

```bash
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
```

## getops

Synopis:

```
while getopts OPTSTRING opt; do
    case $opt in
        # Conditions
    esac
done
```

With `OPTSTRING` describing the short options:

* starting `OPTSTRING` with a `:` will set Error Reporting to "silent mode"
* each character describes a flag
  (eg for `script.sh -h -e -y`, `OPTSTRING` would be `hey`)
* a character followed by a `:` describes a flag with an argument
  (eg for `script -f /tmp/filename`, `OPTSTRING` would be `f:`)

In case of error, `opt` will be set with the following value:

* invalid option:
  * `?`
    * _in "silent mode", `OPTARG` is set with the invalid option character_
* missing required argument:
  * in "verbose mode": `?`, and an error message is printed
  * in "silent mode": `:`, and `OPTARG` is set with the option character

Example:

```
#!/bin/bash
 
while getopts ":ab:" opt; do
    case $opt in
        a)
            echo "-a was triggered!" >&2
            ;;
        b)
            echo "-b was triggered, Parameter: $OPTARG" >&2
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires an argument." >&2
            exit 1
            ;;
    esac
done
```

## References

* [File test operators](http://tldp.org/LDP/abs/html/fto.html)
* [String and integer operators](http://tldp.org/LDP/abs/html/comparison-ops.html)
* [the bracket construct](http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS)
* [Small getops tutorial](https://wiki.bash-hackers.org/howto/getopts_tutorial)
