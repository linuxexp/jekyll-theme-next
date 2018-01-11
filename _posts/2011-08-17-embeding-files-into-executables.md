---
title: Embeding files into executables, that works across different platform
date: 2011-08-17 09:20:00.000000000 +05:30
---

You can embed data into execs in Windows using Executable Resources, This Idea can work across multiple platform and doesn't rely upon the underlying OS. Here's a tiny little bash script. This script compiles the files in the specified folder into a executable, and when you run it, it extracts them. It uses partial linking, external variables, embedding binary data into execs etc.

The scripts compiles the files into object files, compiles its extraction source code, links all the object files into one..Then when it's executed it, extracts them..


There are no dependencies other than bash and standard c++ libraries.

## Steps
* Run compileit.sh in your terminal, specify the folder you want to compile as parameter
* This will generate few files, the main is executable file
* You can run "main" file in terminal, this will extract the files

## Compileit
```bash
#
# Compileit : tiny bash script to compile files into executables
#
#	Copyright (C) 2011 Raja Jamwal <linux1@zoho.com>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
cd $1
mkdir -p objects
## create object files
for file in $( ls -l | grep ^- | awk '{print $8}')
do
echo "Compiling $file..."
ld -r -b binary $file -o objects/$file.o
done
cd objects

for file in $( ls -l | grep ^- | awk '{print $8}')
do
name=`objdump -x $file | grep -o "_binary_[^ ]*_start"`
echo "$name" >> ../../tables
done

IFS=$'n'

for line in $(cat ../../tables)
do
echo "extern char $line[];extern int `echo $line | sed 's/start/size/'`;" >> ../../var
done

echo -e "#include <iostream>n#include <fstream>n
#include <stdlib.h>nusing namespace std;n" >> ../../main.c

cat ../../var >> ../../main.c
echo -e "int extract(char * szData, char * filename, unsigned int size){n
cout << "extracting " << filename << endl;n
ofstream fout(filename, ios::out | ios::binary);n
fout.write(szData, size);n
fout.close();n
}" >> ../../main.c

echo "int main()" >> ../../main.c
echo "{" >> ../../main.c
echo "cout << "<linux1@zoho.com> (C) Raja Jamwal" << endl;" >> ../../main.c

for line in $(cat ../../tables)
do
echo "extract($line, "`echo $line | sed 's/start//' | sed 's/_binary//'`", (int) &`echo $line | sed 's/start/size/'`);" >> ../../main.c
done
echo "return 0;" >> ../../main.c
echo "}" >> ../../main.c

g++ ../../main.c -o ../../main $( ls -l | grep ^- | awk '{print $8}')
```
