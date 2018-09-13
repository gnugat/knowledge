# sed cheat sheet

A highly opinionated sed cheat sheet.

* replace in file: `sed -i <command> <file>`
* replace in files: `find <files> | xargs sed -i <command>`
* replace in files, excluding some directories: `find <files> | grep -v <exclude> | xargs sed -i <command>`
