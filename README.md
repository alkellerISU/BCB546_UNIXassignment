# BCB546_UNIXassignment
Repository for BCB546 UNIX assignments
# UNIX Assignment

## Data Inspection

### Attributes of `fang_et_al_genotypes`

I inspected this file with the following commands in the order listed.

```
$ head fang_et_al_genotypes.txt
$ wc fang_et_al_genotypes.txt
$ ls -lh fang_et_al_genotypes.txt
$ du -h fang_et_al_genotypes.txt
$ tail -n +6 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
$ grep -v "^#" fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
$ file fang_et_al_genotypes.txt
$ hexdump -c fang_et_al_genotypes.txt
```

By inspecting this file I learned that:

* There is 1 header row.
* There apears to be some sort of description of sample ID and pared bases with lots of "?" characters.
* There are 2783 lines, 2744038 words, and 11051939 bytes.
* The permissions are as follows: -rw-r--r--
* Disk usage: 6.1M.
* There are 986 columns (doesn't seem right but got the same results with two differetn codes).
* This is a ASCII text with very long lines and lots of non-ASCII characters.

### Attributes of `snp_position.txt`

I inspected this file with the following commands in the order listed.

```
$ head snp_position.txt
$ wc snp_position.txt
$ ls -lh snp_position.txt
$ du -h snp_position.txt
$ tail -n +6 snp_position.txt | awk -F "\t" '{print NF; exit}'
$ grep -v "^#" snp_position.txt | awk -F "\t" '{print NF; exit}'
$ file snp_position.txt
$ hexdump -c snp_position.txt
```

By inspecting this file I learned that:

* There are 1 or 2 header rows (having hard time telling) that contain labels for each column.
* There does not seem to be any type of sorting to the data.
* The file definitely contains information about snp locations on chromosomes.
* There are 984 lines, 13198 words, 82763 bytes.
* The permissions are as follows: -rw-r--r--
* Disk usage: 38k.
* There are 15 columns (hopefully).
* This is an ASCII text with lots of non-ASCII characters.

## Data Processing

### Seperating Maize and Teosinte Data
We want to get the maize data and teosinte data in their own seperate files. First we need to grep all the maize data together using the group names ZMMIL, ZMMLR, and ZMMMR in the third column of fang_et_al_genotypes.txt. We will need to do the same with the teosinte data as well using ZMPBA, ZMPIL, and ZMPJA group names. 

This command will grep and output all the maize and teosinte into seperate files.

```
$ grep 'ZMM' fang_et_al_genotypes.txt > maize_genotypes_only.txt
$ grep 'ZMP' fang_et_al_genotypes.txt > teosinte_genotypes_only.txt

```

We need to give our files the propper headers now. 
This command pulls the header out of the original file and places in to in a new file.

```
$ head -n 1 fang_et_al_genotypes.txt > header_fang_genotypes.txt
```

Now I need to combine the header file with the maize and teosinte only files.

```
$ cat header_fang_genotypes.txt maize_genotypes_only.txt > maize_head_genotypes.txt
$ cat header_fang_genotypes.txt teosinte_genotypes_only.txt > teosinte_head_genotypes.txt
```

Now we need to transpose both of the genotype files so that the rows and columns are swapped.

```
$ awk -f transpose.awk maize_head_genotypes.txt > transposed_maize_genotypes.txt
$ awk -f transpose.awk teosinte_head_genotypes.txt > transposed_teosinte_genotypes.txt
```

Now I need to sort the transposed data and the snp_positions file based on the first column (SNP_ID) so that they have a common column. In order to sort the snp_posisitons.txt file, I need to remove the header first.

```
$ sort -k1,1 transposed_maize_genotypes.txt > sorted_transposed_head_maize_genotypes.txt
$ sort -k1,1 transposed_teosinte_genotypes.txt > sorted_transposed_head_teosinte_genotypes.txt
$ tail -n +2 snp_position.txt | sort -k1,1 > sorted_snp.txt
```

I checked to make sure there were no errors by running the echo command where 0 means no errors and 1 means errors.

```
$ sort -k1,1 transposed_maize_genotypes.txt > sorted_transposed_head_maize_genotypes.txt
$ echo $?
$ sort -k1,1 transposed_teosinte_genotypes.txt > sorted_transposed_head_teosinte_genotypes.txt
$ echo $?
$ tail -n +2 snp_position.txt | sort -k1,1 > sorted_snp.txt
$ echo $?
```
I got a 0 after all of the echos.

Next I need to join the SNP and genotype data.
```
$ join -1 1 -2 1 -t $'\t' sorted_snp.txt sorted_transposed_head_maize_genotypes.txt > combined_maize.txt
$ join -1 1 -2 1 -t $'\t' sorted_snp.txt sorted_transposed_head_teosinte_genotypes.txt > combined_teosinte.txt
```

### Maize Data
The commands below should get me individual files for each chromosome sorted by increasing position with missinf data codes as ? symbols.
```
$ awk -F "\t" '$3 ~ /1/' combined_maize.txt | sort -t $'\t' -k4,4n > chr1_maize.txt
$ awk -F "\t" '$3 ~ /2/' combined_maize.txt | sort -t $'\t' -k4,4n > chr2_maize.txt
$ awk -F "\t" '$3 ~ /3/' combined_maize.txt | sort -t $'\t' -k4,4n > chr3_maize.txt
$ awk -F "\t" '$3 ~ /4/' combined_maize.txt | sort -t $'\t' -k4,4n > chr4_maize.txt
$ awk -F "\t" '$3 ~ /5/' combined_maize.txt | sort -t $'\t' -k4,4n > chr5_maize.txt
$ awk -F "\t" '$3 ~ /6/' combined_maize.txt | sort -t $'\t' -k4,4n > chr6_maize.txt
$ awk -F "\t" '$3 ~ /7/' combined_maize.txt | sort -t $'\t' -k4,4n > chr7_maize.txt
$ awk -F "\t" '$3 ~ /8/' combined_maize.txt | sort -t $'\t' -k4,4n > chr8_maize.txt
$ awk -F "\t" '$3 ~ /9/' combined_maize.txt | sort -t $'\t' -k4,4n > chr9_maize.txt
$ awk -F "\t" '$3 ~ /10/' combined_maize.txt | sort -t $'\t' -k4,4n > chr10_maize.txt
```
Now I am making sorted data with decreasing positions with missing data replaced with - symbol.

```
$ awk -F "\t" '$3 ~ /1/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr1d_maize.txt
$ awk -F "\t" '$3 ~ /2/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr2d_maize.txt
$ awk -F "\t" '$3 ~ /3/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr3d_maize.txt
$ awk -F "\t" '$3 ~ /4/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr4d_maize.txt
$ awk -F "\t" '$3 ~ /5/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr5d_maize.txt
$ awk -F "\t" '$3 ~ /6/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr6d_maize.txt
$ awk -F "\t" '$3 ~ /7/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr7d_maize.txt
$ awk -F "\t" '$3 ~ /8/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr8d_maize.txt
$ awk -F "\t" '$3 ~ /9/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr9d_maize.txt
$ awk -F "\t" '$3 ~ /10/' combined_maize.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr10d_maize.txt
```
This should give me a file with SNPs with unknown positions.
```
$ awk -F '\t' '$4 == "unknown"' combined_maize.txt > unknown_SNPs_maize.txt
```
This should give me a file with SNPs with multiple positions.
```
$ awk -F '\t' '$4 == "multiple"' combined_maize.txt > multiple_SNPs_maize.txt
```

### Teosinte Data
The commands below should get me individual files for each chromosome sorted by increasing position with missinf data codes as ? symbols.
```
$awk -F "\t" '$3 ~ /1/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr1_teosinte.txt
$awk -F "\t" '$3 ~ /2/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr2_teosinte.txt
$awk -F "\t" '$3 ~ /3/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr3_teosinte.txt
$awk -F "\t" '$3 ~ /4/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr4_teosinte.txt
$awk -F "\t" '$3 ~ /5/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr5_teosinte.txt
$awk -F "\t" '$3 ~ /6/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr6_teosinte.txt
$awk -F "\t" '$3 ~ /7/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr7_teosinte.txt
$awk -F "\t" '$3 ~ /8/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr8_teosinte.txt
$awk -F "\t" '$3 ~ /9/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr9_teosinte.txt
$awk -F "\t" '$3 ~ /10/' combined_teosinte.txt | sort -t $'\t' -k4,4n > chr10_teosinte.txt
```
Now I am making sorted data with decreasing positions with missing data replaced with - symbol.
```
$ awk -F "\t" '$3 ~ /1/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr1d_teosinte.txt
$ awk -F "\t" '$3 ~ /2/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr2d_teosinte.txt
$ awk -F "\t" '$3 ~ /3/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr3d_teosinte.txt
$ awk -F "\t" '$3 ~ /4/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr4d_teosinte.txt
$ awk -F "\t" '$3 ~ /5/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr5d_teosinte.txt
$ awk -F "\t" '$3 ~ /6/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr6d_teosinte.txt
$ awk -F "\t" '$3 ~ /7/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr7d_teosinte.txt
$ awk -F "\t" '$3 ~ /8/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr8d_teosinte.txt
$ awk -F "\t" '$3 ~ /9/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr9d_teosinte.txt
$ awk -F "\t" '$3 ~ /10/' combined_teosinte.txt | sort -t $'\t' -k4,4n -r | sed 's/?/-/g' > chr10d_teosinte.txt
```
This should give me a file with SNPs with unknown positions.
```
$ awk -F '\t' '$4 == "unknown"' combined_teosinte.txt > unknown_SNPs_teosinte.txt

```
This should give me a file with SNPs with multiple positions.
```
$ awk -F '\t' '$4 == "multiple"' combined_teosinte.txt > multiple_SNPs_teosinte.txt
```

