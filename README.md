# BCB546X\_Unix\_Assignment

##1. Data Inspection

First I examined the files for non-ASCII characters, file size, and line/word/character counts:

    $ file fang\_et\_al\_genotypes.txt snp\_position.txt  
    fang\_et\_al\_genotypes.txt: ASCII text, with very long lines  
    snp\_position.txt: ASCII text

    $ du -h fang_et_al_genotypes.txt snp_position.txt
    11M     fang_et_al_genotypes.txt
    84K     snp_position.txt

    $ wc fang_et_al_genotypes.txt snp_position.txt
        2783  2744038 11051939 fang_et_al_genotypes.txt
         984    13198    82763 snp_position.txt
        3767  2757236 11134702 total


Using `head` I inspected the first 10 lines of each file, neither of which began with any comment lines (results show portions of first lines only due to length):

    $ head fang_et_al_genotypes.txt snp_position.txt
    Sample_ID       JG_OTU  Group   abph1.20        abph1.22        ae1.3   [...]
    SNP_ID  cdv_marker_id   Chromosome      Position         alt_pos mult_positions   [...]

The data in each file appeared to be tab-delimited with the first rows consisting of column headings, so I used `Awk` to count the number of columns in the first row (and verified the same count for another row mid-file). I also used `cut` and `grep` to check for any rows which might contain entries beyond the stated number of columns:

    $ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
    986

    $ awk -F "\t" 'NR==1783 {print NF; exit}' fang_et_al_genotypes.txt
    986

    $ cut -f 987-997 fang_et_al_genotypes.txt | grep "." | wc -l
    0



    $ awk -F "\t" '{print NF; exit}' snp_position.txt
    15

    $ awk -F "\t" 'NR==783 {print NF; exit}' snp_position.txt
    15

    $ cut -f 16-30 snp_position.txt | grep "." | wc -l
    0

In short, `fang_et_al_genotypes.txt` appears to be a (2,783 x 986)-entry table and `snp_position.txt` a (984 x 15)-entry table (including column headers as first row).

Given the close correlation between the number of columns in `fang_et_al_genotypes.txt` and the number of rows in `snp_position.txt`, I examined the first and last 5 columns in `fang_et_al_genotypes.txt` and the first and last 5 rows in `snp_position.txt`:

    $ head -n 5 fang_et_al_genotypes.txt | cut -f 1-5,982-986 | column -t
    Sample_ID  JG_OTU    Group  abph1.20  abph1.22  zen1.1  zen1.2  zen1.4  zfl2.6  zmm3.4
    SL-15      T-aust-1  TRIPS  ?/?       ?/?       ?/?     ?/?     ?/?     G/G     T/T
    SL-16      T-aust-2  TRIPS  ?/?       ?/?       ?/?     ?/?     ?/?     G/G     T/T
    SL-11      T-brav-1  TRIPS  ?/?       ?/?       ?/?     A/A     ?/?     G/G     T/T
    SL-12      T-brav-2  TRIPS  ?/?       ?/?       ?/?     A/A     ?/?     G/G     T/T

    $ cat snp_position.txt | (head -n 5; tail -n 5) | cut -f 1-5 | column -t
    SNP_ID    cdv_marker_id  Chromosome  Position   alt_pos
    abph1.20  5976           2           27403404
    abph1.22  5978           2           27403892
    ae1.3     6605           5           167889790
    ae1.4     6606           5           167889682
    zen1.1    3519           unknown     unknown
    zen1.2    3520           unknown     unknown
    zen1.4    3521           unknown     unknown
    zfl2.6    6463           2           12543294
    zmm3.4    3527           9           16966348

Thus it appears the 983 (non-header) rows in `snp_position.txt` correspond to the final 983 columns in `fang_et_al_genotypes.txt`. This inspection also reveals that each file contains fields with missing or incomplete information.


&& || and $?


##2. Data Processing

Since we eventually want to have missing data in the formatted files encoded by particular symbols, I started by inserting place-holder symbols into any empty fields:

    $ grep -v "[@#]" fang_et_al_genotypes.txt | wc -l
    2783

    $ grep -v "[@#]" snp_position.txt | wc -l
    984

    $ sed 's/\t/@:/g' snp_position.txt | sed 's/:@/:#@/g' | sed 's/@:/\t/g'

    $ sed 's/\t/@:/g' snp_position.txt | sed 's/:@/:#@/g' | sed 's/@:/\t/g' | cut -f 16-30 snp_position.txt | grep "." | wc -l
    0


For all files generated during the data processing steps, the first three columns will be "SNP_ID", "Chromosome", and "Position", which are respectively the first, third, and fourth columns of `snp_position.txt`. The output files will be organized based on the location of SNPs, which in addition to the "Chromosome" and "Position" columns may also 

    $ tail -n +2 snp_position.txt | cut -f 3 | sort | uniq | wc -l
    12

    $ tail -n +2 snp_position.txt | cut -f 3 | sort -V | uniq -c
        155 1
        127 2
        107 3
         91 4
        122 5
         76 6
         97 7
         62 8
         60 9
         53 10
          6 multiple
         27 unknown

    $ tail -n +2 snp_position.txt | cut -f 3 | sort -V | wc -l
    983



    $ cut -f 3 fang_et_al_genotypes.txt | grep -E "(ZMMIL|ZMMLR|ZMMMR)" | sort | uniq -c
        290 ZMMIL
       1256 ZMMLR
         27 ZMMMR

    $ cut  -f 3 fang_et_al_genotypes.txt | grep -E "(ZMPBA|ZMPIL|ZMPJA)" | sort | uniq -c
        900 ZMPBA
         41 ZMPIL
         34 ZMPJA
