# PDSB Assignment 1
Alexander M. Procton / 1.25.2018

#### I. Get the data files

To download the files using `curl`, I entered
   
```bash
curl http://eaton-lab.org/pdsb/test.fastq.gz > test.fastq.gz
curl http://eaton-lab.org/pdsb/iris-data-dirty.csv > iris-data-dirty.csv
```

The commands to view the files are

```bash
zless test.fastq.gz
less iris-data-dirty.csv
```

To view only the first 5 rows, I entered

```bash
zless test.fastq.gz|head -5
head -5 iris-data-dirty.csv
```

#### II. Clean the data

I used Google to find a page which explained how to use `grep` to search for matches with multiple strings ([link](https://www.cyberciti.biz/faq/searching-multiple-words-string-using-grep/)). Using this pattern, I used the command

```bash
grep -nv 'setosa\|versicolor\|virginica' iris-data-dirty.csv
```

I found that two lines had misspellings:

```bash
12:4.8,3.4,1.6,0.2,Iris-setsa
51:7.0,3.2,4.7,1.4,Iris-versicolour
```

Using `sed` to fix the spelling errors and `grep` to remove the lines with NAs, I made a clean file

```bash
grep -v NA iris-data-dirty.csv| \
sed 's/setsa/setosa/g'| \
sed 's/versicolour/versicolor/g' > iris-data-clean.csv
```

To list the number of data values for each species, I used this command:

```bash
cut -d ',' -f 5 iris-data-clean.csv | uniq -c | sort -r
  50 Iris-virginica
  50 Iris-setosa
  48 Iris-versicolor
   1
```

#### III. Summarize sequence data file

I unzipped the sequence file using `gunzip test.fastq.gz`. I looked up how to match the end of a line using `grep` ([link](https://unix.stackexchange.com/questions/124462/detecting-pattern-at-the-end-of-a-line-with-grep)), so I tried to use the pattern `^TGCAG*GAG$`. Unfortunately this gave me 0 results. I realized that every line of sequence data began with "TGCAG," so I matched only the end of lines and found 44 matches.

```bash
grep -c GAG$ test.fastq
  44
```

#### IV. Summarize sequence data file (2)

To find each read, I used the regular expression `grep -EA3 '@[0-9]{5}_' test.fastq`. [This page](https://askubuntu.com/questions/27838/how-to-grep-2-or-3-lines-one-containing-the-text-i-want-and-the-others-just-be) explained how to find 3 lines after a match using `-A3`, and [this cheatsheet](http://web.mit.edu/hackl/www/lab/turkshop/slides/regex-cheatsheet.pdf) explained how to use `{5}` to repeat a match on exacly 5 digits, because the first line of each set of read data begins with an @ followed by 5 digits. I found that this did not work unless I enabled extended regular expressions using `-E`.

I found a [link](https://stackoverflow.com/questions/16317961/how-to-process-each-line-received-as-a-result-of-grep-command) that suggested a `while read` loop would be an easier way to loop over the output of `grep`, and [another source](http://www.compciv.org/topics/bash/loops/) provided more details. Within the loop, `"taxon"` stores the taxon name as a string, found using `grep` to once again subset each read to just the first line and `cut` to find just the name. The read is then appended to a file with the same name as `taxon`.

The code to combine all of these commands is as follows:

```bash
mkdir sorted_reads

grep -EA3 '@[0-9]{5}_' test.fastq | while read line; do
    taxon="$(echo $line|grep -E '@[0-9]{5}_'|\
    cut -d ' ' -f 1|cut -d '.' -f 1|cut -d '_' -f 2)"
    echo $taxon
done


grep -EA3 '@[0-9]{5}_' test.fastq |\
while read line
do 
   "taxon"=grep -E '@[0-9]{5}_' $line |\ 
   cut -d ' ' -f 1|cut -d '.' -f 1| cut -d '_' -f 2
   $line >> sorted_reads/
done
```
