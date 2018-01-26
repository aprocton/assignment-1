# PDSB Assignment 1
Alexander M. Procton / 1.25.2018

#### I. Get the data files

To download the files using `curl`, I entered
   
```
curl http://eaton-lab.org/pdsb/test.fastq.gz > test.fastq.gz
curl http://eaton-lab.org/pdsb/iris-data-dirty.csv > iris-data-dirty.csv
```

The commands to view the files are

```
zless test.fastq.gz
less iris-data-dirty.csv
```

To view only the first 5 rows, I entered

```
zless test.fastq.gz|head -5
head -5 iris-data-dirty.csv
```

#### II. Clean the data

I used Google to find a page which explained how to use `grep` to search for matches with multiple strings ([link](https://www.cyberciti.biz/faq/searching-multiple-words-string-using-grep/)). Using this pattern, I used the command

```
grep -nv 'setosa\|versicolor\|virginica' iris-data-dirty.csv
```

I found that two lines had misspellings:

```
12:4.8,3.4,1.6,0.2,Iris-setsa
51:7.0,3.2,4.7,1.4,Iris-versicolour
```

Using `sed` to fix the spelling errors and `grep` to remove the lines with NAs, I made a clean file

```
grep -v NA iris-data-dirty.csv| \
sed 's/setsa/setosa/g'| \
sed 's/versicolour/versicolor/g' > iris-data-clean.csv
```

To list the number of data values for each species, I used this command:

```
cut -d ',' -f 5 iris-data-clean.csv | uniq -c | sort -r
  50 Iris-virginica
  50 Iris-setosa
  48 Iris-versicolor
   1
```

#### III. Summarize sequence data file

I unzipped the sequence file using `gunzip test.fastq.gz`. I looked up how to match the end of a line using `grep` ([link](https://unix.stackexchange.com/questions/124462/detecting-pattern-at-the-end-of-a-line-with-grep)), so I tried to use the pattern `^TGCAG*GAG$`. Unfortunately this gave me 0 results. I realized that every line of sequence data began with "TGCAG," so I matched only the end of lines and found 44 matches.

```
grep -c GAG$ test.fastq
  44
```

#### IV. Summarize sequence data file (2)


