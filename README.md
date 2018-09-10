B546X Unix Assignment

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
