---
layout: post
title: Power Tools for Unix
published: true
---


A guide to useful linux commands that I have learnt over my years of scavenging through log files, trying to debug stuff. This is by no means exhaustive or a complete list. These are just some notes to my future self.

### cut
```sh
    cut -d '<DELIMITER>' -f '<field1>,<field2>..' [FILE]
```

Notes:
 * Each line in the input/FILE is split by delimiter and treated as a 1-indexed array.
 * Fields is a comma separated list of indices.
 * Order of Fields doesn't matter. The output is always in order they appear in the input.


### find
```sh
    find [ROOT_DIR] -type [-f, -d] -name "REGEX PATTERN"
```

### grep
* Simple search for a keyword in a DIR recursively (-r), ignoring the case (-i) and print the line number(-n)
```sh
    grep -nri "PATTERN" dir
```
* Print lines in a file that do no match a "PATTERN", ignoring case(-i)
```sh
    grep -vni "PATTERN" file
```
* Either or in grep
```sh
    grep -E "(PATTERN A | PATTERN B)" file
```
* Use a perl regex -P
* Interpret patterns as fixed strings -F
* Count -c, only matching -o

### xargs
Search for a list of keywords, saved in a file called keywords.txt.
```sh
cat keywords.txt | xargs -I '{}' grep '{}' file.txt
```
* -I [PATTERN] can be anything. '{}' is just a convention

### sed
Replace a string in-place in a file
```sh
sed -i 's/STRING_TO_REPLACE/STRING_TO_REPLACE_IT/g' filename
```


### Guides for reference

https://www-users.york.ac.uk/~mijp1/teaching/2nd_year_Comp_Lab/guides/grep_awk_sed.pdf
https://docstore.mik.ua/orelly/unix/sedawk/
