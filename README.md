# B546X Unix Assignment

## File Inspection

1) Use `cat` to print the files to the screen for a first look at the contents:

`cat fang_et_al_genotypes.txt`

- This is a very long file, so it filled the screen repeatedly and so I used `Ctrl + C` to cancel. Briefly, the data seems to include pairs of nucleotide bases. 

`cat snp_position.txt`

- This is also a long file, though shorter than the other. 

2) Because the files are so long, use `head` to inspect only the first few lines:

`head fang_et_al_genotypes.txt`

- This file is still too long and has too many columns for head to display it well with default options.

`head snp_position.txt`

- This now shows the column names, so I can see what each column contains. Briefly, this looks to be SNP IDs and their position in the genome, as well as other information about that SNP.

3) Use `less` to page through the files.

`less fang_et_al_genotypes.txt`

- This now shows at least some of the column names, which include sample ID and what appear to be gene or SNP names. Scrolling down, we see that under these are genotypes at each of these sites.

`less snp_position.txt`

- `head` already worked well on this file because of its fewer number of columns compared to the other file, so `less` just confirms what I saw above using `head`.

4) Use `column -t` to inspect the files with their columns aligned in a more human-readable way:

`head -2 fang_et_al_genotypes.txt | column -t`

- This produced output that said that the lines are too long and so they are truncated, which is logical considering what I saw above. Again, this shows what might be SNP names in the first row and the genotypes at those sites in the next row.

`head -2 snp_position.txt | column -t`

- This shows the columns described above that I saw using `less` and `head`, more in a more readable way so that it is much more obvious to see which value aligns with which column heading. 


5) Use `wc` to find number of lines, words, and characters in these files:

`wc fang_et_al_genotypes.txt`
	
- This file has 2783 lines, 2744038 words, and 11051939 characters.

`wc snp_positions.txt`

- This file has 984 lines, 13198  words, and 82763 characters.

6) Use `du -h` to find file size in a human-readable display:

`du -h fang_et_al_genotypes.txt`

- This file has size 11M.

`du -h snp_position.txt`

- This file has size 84K.

7) Use `awk` to find number of columns; use `-F "\t"` to set the field separators as tabs (these appear to be the separators based on earlier inspection) and the built-in variable `NF` to print the number of fields (here, the number of columns):

`awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt`

- There are 986 tab-separated columns in this file.

`awk -F "\t" '{print NF; exit}' snp_position.txt`

- There are 15 tab-separated columns in this file.


## Data Processing

1) Using `awk`, separate the genotype file into two files, one for maize and one for teosinte:

`awk '$3 ~/Group|ZMMIL|ZMMLR|ZMMMR/ { print $0 }' fang_et_al_genotypes.txt | cat > maize_genotypes.txt`
 
`awk '$3 ~/Group|ZMPBA|ZMPIL|ZMPJA/ { print $0 }' fang_et_al_genotypes.txt | cat > teosinte_genotypes.txt`

- In the pattern part of this `awk` statement, `$3` means to look in column 3 to match the given pattern, which in turn is enclosed in `//` In this case, the pattern to match is either the maize or the teosinte groups, as well as "Group" to include the header row. Then, `print $0` prints the entire record where the pattern was matched. Finally, this is piped to `cat` and printed to a new file.

- To check that this was successful, examine the input and the output files:
 
	`cut -f3 fang_et_al_genotypes.txt | sort -k1,1 | uniq -c`

	- This cuts out the third field (column) from the genotype file, which contains the group, pipes this to sort, and then finds the count of each uniqe entry. The output shows that there were 290 ZMMIL, 1256 ZMMLR, and 27 ZMMMR records for a total of 1573 maize records; also, there were 900 ZMPBA, 41 ZMPIL, and 34 ZMPJA records for a total of 975 teosinte records.


	`cut -f3 maize_genotypes.txt | sort -k1,1 | uniq -c` and `cut -f3 teosinte_genotypes.txt|  sort -k1,1 | uniq -c` 

	- These lines of code cut, sort, and find unique values in the output files as above, and the output confirms that the number of records for each group is as expected from the input file. So, no records or groups were lost or added in the process.


	`wc -l teosinte_genotypes.txt` and `wc -l maize_genotypes.txt`

	- Finally, these lines of code check that the total numbers of lines in the output files are as expected: 975 teosinte lines and 1573 maize lines, plus 1 record for the header in each file.

2) Transpose the teosinte and maize genotype data using provided `teosinte.awk`:

`awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt` and `awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt`
	
3) Next, we want to produce one file for maize and one for teosinte with SNP ID as the first column, chromosome next, position third, and then finally columns of genotype data. 
	
- To do this, first make one file with SNP ID, chromosome, and position column headings only: `cut -f1,3,4 snp_position.txt | head -1 > sorted_snp_position.txt`

- Then, add the SNP ID, chromosome, and position data to the file, sorted by SNP ID: `cut -f1,3,4 snp_position.txt | tail -n +2 | sort -k1,1 >> sorted_snp_position.txt`

- Make files with only genotype column headings for maize and teosinte (Sample ID should be changed to SNP ID in order to match snp position file): `head -1 transposed_teosinte_genotypes.txt | sed 's/Sample_ID/SNP_ID/g' > sorted_transposed_teosinte_genotypes.txt` and `head -1 transposed_maize_genotypes.txt | sed 's/Sample_ID/SNP_ID/g' > sorted_transposed_maize_genotypes.txt`

- Prepare the genotype files by removing unneeded rows (JG_OTU and Group, as well as the header row), and sort by sample ID; add these to the files with the column headers already prepared:
 `awk '$1 !~/JG_OTU|Group|Sample_ID/ { print $0 }' transposed_maize_genotypes.txt | sort -k1,1 >> sorted_transposed_maize_genotypes.txt` and `awk '$1 !~/JG_OTU|Group|Sample_ID/ { print $0 }' transposed_teosinte_genotypes.txt | sort -k1,1 >> sorted_transposed_teosinte_genotypes.txt`

- Now that all the files are sorted, join them by SNP ID: `join -1 1 -2 1 -t $'\t' sorted_snp_position.txt sorted_transposed_maize_genotypes.txt > maize_joined.txt` and `join -1 1 -2 1 -t $'\t' sorted_snp_position.txt sorted_transposed_teosinte_genotypes.txt > teosinte_joined.txt`
	- Check that all input and output files have the same number of lines, to be sure we didn't lose any in the join: `wc -l maize_joined.txt teosinte_joined.txt sorted_snp_position.txt sorted_transposed_maize_genotypes.txt sorted_transposed_teosinte_genotypes.txt`

4) Now that we have a full, joined dataset for both maize and teosinte, create the desired output files for maize.

- Make 1 file for each chromosome with SNPs ordered by increasing position and with missing data encoded as "?" (this is already how missing data is encoded): 
`for i in {1..10}; do ( head -1 maize_joined.txt; awk '$2 ~/^'$i'$/ { print $0 }' maize_joined.txt | sort -k3,3n) > chr"$i"_increasing_maize.txt; done`

	- The code above uses a `for` loop to loop through all the chromosomes. 
	-  Within the loop, use a subshell to first extract the header data from the full data set. 
	-  Then, use `awk` to extract all SNPs from the given chromosome (in column `2`; use `'$i'` to pass awk the variable represented by `i` and use `^` and `$` to make sure that `i` matches the whole field and not just part of it) and `sort` this data by increasing SNP position (in column `3`), which is numeric (use `-n`).
	-  SNPs with multiple positions are sorted to the top of the file. 
	-  SNPs with unknown positions are not included because their chromosome is not known. 
	-  Nothing has to be done to change the missing data because it is already encoded as `?`. 
	-  Finally, write the output to a file. 
	- Now, check that this process was successful by checking the number of SNPs in each chromosome in the original file using `cut -f2 maize_joined.txt | sort -k1,1n | uniq -c` and make sure these match the size of the output files (plus one line for the header) using `for i in {1..10}; do wc -l chr"$i"_increasing_maize.txt; done` .

- Make 1 file for each chromosome with SNPs ordered by decreasing position and with missing data encoded as `-`: `for i in {1..10}; do ( head -1 maize_joined.txt; awk '$2 ~/^'$i'$/ { print $0 }' maize_joined.txt | sort -k3,3nr | sed s/?/-/g) > chr"$i"_decreasing_maize.txt; done`

	- As above, this code uses a `for` loop to work on all chromosomes, `awk` to extract SNPs, and `sort` to sort the output.
	- However, it uses `-r` to sort the SNPs by decreasing position rather than increasing.
	- In order to code all missing data as `-` (previously coded as `?`), use `sed` to find `?` and replace it with `-` wherever it occurs.
	- Again, write output to a file.
	- Finally, check that the process was successful as above, using `cut -f2 maize_joined.txt | sort -k1,1n | uniq -c` to examine the input file and `for i in {1..10}; do wc -l chr"$i"_decreasing_maize.txt; done` to examine the output.

- Make 1 file with all SNPs with unknown positions in the genome: `awk '$3 ~/unknown|Position/ { print $0 }' maize_joined.txt > unknown_pos_maize.txt`
	- This code uses `awk` to extract all SNPs with unknown positions, as well as the header row, and print this to an output file

- Make 1 file with all SNPs with multiple positions in the genome: `awk '$3 ~/multiple|Position/ { print $0 }' maize_joined.txt > multiple_pos_maize.txt`
	- As above, this code uses `awk` to extract SNPs with multiple positions, as well as the header row, and print to an output file.

5) Repeat this process for teosinte. Use the same code as above but with "teosinte" substituted for "maize" in each command:

- Make and check the first set of ten output files:

		for i in {1..10}; do ( head -1 teosinte_joined.txt; awk '$2 ~/^'$i'$/ { print $0 }' teosinte_joined.txt | sort -k3,3n) > chr"$i"_increasing_teosinte.txt; done`

		cut -f2 teosinte_joined.txt | sort -k1,1n | uniq -c

		for i in {1..10}; do wc -l chr"$i"_increasing_teosinte.txt; done

- Make and check the next set of ten output files:

		for i in {1..10}; do ( head -1 teosinte_joined.txt; awk '$2 ~/^'$i'$/ { print $0 }' teosinte_joined.txt | sort -k3,3nr | sed s/?/-/g) > chr"$i"_decreasing_teosinte.txt; done

		cut -f2 teosinte_joined.txt | sort -k1,1n | uniq -c
		for i in {1..10}; do wc -l chr"$i"_decreasing_teosinte.txt; done

- Make the unknown SNP file: `awk '$3 ~/unknown|Position/ { print $0 }' teosinte_joined.txt > unknown_pos_teosinte.txt`


- Make the multiple position SNP file: `awk '$3 ~/multiple|Position/ { print $0 }' teosinte_joined.txt > multiple_pos_teosinte.txt`
