Most of bioinformaticians know how to analyze data with Perl or Python programming. However, not all of them realize that it is not always necessary to write programs. Sometimes, using UNIX commands is much more convenient and can save a lot of time spent on some trivial, yet tedious, routines.

#  UNIX commands 

### Xargs

Xargs is one of my most frequently used UNIX commands. Unfortunately, many UNIX users overlook how powerful it is.

`delete all *.txt files in a directory:

```
find . -name "*.txt" | xargs rm
```

`package all *.pl files in a directory:

```
find . -name "*.pl" | xargs tar -zcf pl.tar.gz
```

`kill all processes that match "something":`

```
ps -u whoami | awk '/something/{print $1}' | xargs kill
```

`rename all *.txt as *.bak:

```
find . -name "*.txt" | sed "s/\.txt$//" | xargs -i echo mv {}.txt {}.bak | sh
```

`run the same command for 100 times (in case of bootstraing, for example):`

```
perl -e 'print "$_\n"for(1..100)' | xargs -i bsub echo bsub -o {}.out -e {}.err some_cmd | sh
```

`submit all commands in a command file (one command per line):`

```
cat my_cmds.sh | xargs -i echo bsub {} | sh
```

`The last three examples only work with GNU xargs. BSD xargs does not accept '-i'.`

### Find

In a directory, find command finds all the files that meet certain criteria. You can write very complex rules at the command line, but I think the following examples are most useful:

`find all files with extension as "*.txt" (files can exist in subdirectory):

```
find . -name "*.txt"
```

`find all directories:`

```
find . -type d
```

### Awk

Awk is a programming language that is specifically designed for quickly manipulating space delimited data. Although you can achieve all its functionality with Perl, awk is simpler in many practical cases. You can find a lot of online tutorials, but here I will only show a few examples which cover most of my daily uses of awk.

`choose rows where column 3 is larger than column 5:`

```
awk '$3>$5' input.txt > output.txt
```

`extract column 2,4,5:`

```
awk '{print $2,$4,$5}' input.txt > output.txt
```

`awk 'BEGIN{OFS="\t"}{print $2,$4,$5}' input.txt`
`show rows between 20th and 80th:`
`awk 'NR>=20&&NR<=80' input.txt > output.txt`
`calculate the average of column 2:`

```
awk '{x+=$2}END{print x/NR}' input.txt
```

`regex (egrep):`
`awk '/^test[0-9]+/' input.txt`
`calculate the sum of column 2 and 3 and put it at the end of a row or replace the first column:`

```
`awk '{print $0,$2+$3}' input.txt`
```

```
awk '{$1=$2+$3;print}' input.txt
```

`join two files on column 1:`
`awk 'BEGIN{while((getline<"file1.txt")>0)l[$1]=$0}$1 in l{print $0"\t"l[$1]}' file2.txt > output.txt`
`count number of occurrence of column 2 (uniq -c):`
`awk '{l[$2]++}END{for (x in l) print x,l[x]}' input.txt`
`apply "uniq" on column 2, only printing the first occurrence (uniq):`

```
`awk '!($2 in l){print;l[$2]=1}' input.txt`
```

`count different words (wc):`
`awk '{for(i=1;i!=NF;++i)c[$i]++}END{for (x in c) print x,c[x]}' input.txt`
`deal with simple CSV:`
`awk -F, '{print $1,$2}'`
`substitution (sed is simpler in this case):`
`awk '{sub(/test/, "no", $0);print}' input.txt`

### Cut

Cut cuts specified columns. The default delimiter is a single TAB.

`cut the 1st, 2nd, 3rd, 5th, 7th and following columns:`
`cut -f1-3,5,7- input.txt`
`cut the 3rd column with columns separated by a single space:`
`cut -d" " -f 3 input.txt`
`Note that awk, like Perl's split, takes continuous blank characters as the delimiter, but cut only takes a single character as the delimiter.`

### Sort

Almost all the scripting languages have built-in sort, but none of them are so flexible as sort command. In addition, GNU sort is also space efficient. I used to sort a 20Gb file with less than 2Gb memory. It is not trivial to implement so powerful a sort by yourself.

`sort a space-delimited file based on its first column, then the second if the first is the same, and so on:`
`sort input.txt`
`sort a huge file (GNU sort ONLY):`
`sort -S 1500M -t $HOME/tmp input.txt > sorted.txt`
`sort starting from the third column, skipping the first two columns:`
`sort +2 input.txt`
`sort the second column as numbers, descending order; if identical, sort the 3rd as strings, ascending order:`
`sort -k2,2nr -k3,3 input.txt`
`sort starting from the 4th character at column 2, as numbers:`
`sort -k2.4n input.txt`

### Other Tips

use brackets:
`(echo hello; echo world; cat foo.txt) > output.txt`
`(cd foo; ls bar.txt)`
`save stderr output to a file:`
`some_cmd 2> output.err`
`direct stderr output to stdout:`
`some_cmd 2>&1 | more`
`some_cmd >output.outerr 2>&1`
`view a text file using 4 as a TAB size and without line wrapping:`
`less -S -x4 text.txt`
`[sed] substitute 'foo(\d+)' as "(\d+)bar":`
`sed "s/foo\([0-9]*\)/\1bar/g"`
`perl -pe 's/foo(\d+)/$1bar/g"`
`[uniq] count the occurrence of different strings at column 2:`
`cut -f2 input.txt | uniq -c`
`grep "--enable" in a file (use "--" to prevent grep from parsing command options):`
`grep -- "--enable" input.txt`