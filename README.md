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

Since we eventually want any missing data in the formatted files to be encoded using particular symbols, I started by using "@" as a proxy delimiter to insert the place-holder symbol "#" into all empty fields (which first required verifying that neither symbol appeared anywhere else in either file):

    $ grep -v "[@#]" fang_et_al_genotypes.txt | wc -l
    2783

    $ grep -v "[@#]" snp_position.txt | wc -l
    984

    $ sed 's/\t/@:/g' snp_position.txt | sed 's/:@/:#@/g' | sed 's/@:/\t/g' > snp_position_noblanks.txt

    $ cut -f 16-30 snp_position_noblanks.txt | grep "." | wc -l
    0

The last command verifies that no row in the output contains more than 15 columns (indicating that no exexcess fields were added to any row 

For all files generated during the data processing steps, the first three columns will be "SNP_ID", "Chromosome", and "Position", which are respectively the first, third, and fourth columns of `snp_position.txt`. The output files will be organized based on the location of SNPs, which in addition to the "Chromosome" and "Position" columns may also 

awk -f transpose.awk fang_et_al_genotypes.txt | tail -n +4 | sort -V -k1,1 | sort -V -k1,1 -c

    $ tail -n +2 snp_position_noblanks.txt | cut -f 3 | sort -V | uniq -c
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

    $ tail -n +2 snp_position_noblanks.txt | cut -f 3 | wc -l
    983

    $ tail -n +2 snp_position_noblanks.txt | cut -f 4 | sort -V | uniq -c | (head -n 5; tail -n 5)
          1 139753
          1 139810
          1 157104
          1 920922
          1 945545
          1 298082627
          1 298412984
         11 multiple
         27 unknown
          6 #

    $ tail -n +2 snp_position_noblanks.txt | cut -f 4 | sort -V | grep "[^0-9]" | uniq -c
         11 multiple
         27 unknown
          6 #

    $ tail -n +2 snp_position_noblanks.txt | cut -f 6 | sort -V | uniq -c
          1 Chr1(9691660);Chr2(225530627);
          1 Chr1(275076660);Chr8(71078729);
          1 Chr2(17581865);Chr3(143082611);Chr6(158386495);
          1 Chr2(209957515;210039173);
          1 Chr4(106637739;106644768;157652727);Chr8(111504317);
          1 Chr4(226884645);Chr5(115246667);
          1 Chr4(241144995;241178306);
          1 Chr4(241145101;241178412);
          1 Chr4(241145151;241178462);
          1 Chr5(150771372);Chr10(139167376);
          1 Chr6(25035259;25122956);
          1 Chr6(25035262;25122959);
          1 Chr6(25035303;25123000);
          1 Chr7(108218425;108952795);
          1 Chr9(43692524;43720849);
          1 Chr9(43692603;43720928);
          1 Chr9(43692633;43720958);
        966 #

    $ (awk 'NR == 1 {print "Line \t" $0}' snp_position_noblanks.txt; awk '$6 ~ /Chr.*/ {print NR "\t" $0}' \
    snp_position_noblanks.txt) | cut -f 1-7 | column -t
    Line  SNP_ID       cdv_marker_id  Chromosome  Position  alt_pos  mult_positions
    41    PZA00030.11  3616           multiple    #         #        Chr1(275076660);Chr8(71078729);
    188   PZA00270.3   3790           multiple    #         #        Chr5(150771372);Chr10(139167376);
    197   PZA00296.6   3810           2           multiple  #        Chr2(209957515;210039173);
    229   PZA00335.12  3833           multiple    #         #        Chr2(17581865);Chr3(143082611);Chr6(158386495);
    335   PZA00486.2   3964           7           multiple  #        Chr7(108218425;108952795);
    340   PZA00493.1   11295          4           multiple  #        Chr4(241144995;241178306);
    341   PZA00493.2   11296          4           multiple  #        Chr4(241145101;241178412);
    342   PZA00493.5   11299          4           multiple  #        Chr4(241145151;241178462);
    399   PZA00566.5   4026           multiple    #         #        Chr1(9691660);Chr2(225530627);
    410   PZA00589.10  7329           9           multiple  #        Chr9(43692524;43720849);
    411   PZA00589.8   4051           9           multiple  #        Chr9(43692603;43720928);
    412   PZA00589.9   4052           9           multiple  #        Chr9(43692633;43720958);
    436   PZA00636.5   4093           multiple    #         #        Chr4(226884645);Chr5(115246667);
    567   PZA02948.19  4286           6           multiple  #        Chr6(25035262;25122959);
    568   PZA02948.21  4287           6           multiple  #        Chr6(25035303;25123000);
    569   PZA02948.22  11655          6           multiple  #        Chr6(25035259;25122956);
    808   PZB00188.6   6198           multiple    #         #        Chr4(106637739;106644768;157652727);Chr8(111504317);

Examining the entries in Column 6 shows there are in fact a total of 17 SNPs with multiple positions in the genome -- 6 located on multiple chromosomes and 11 located at multiple positions on the same chromosome.





    $ cut -f 3 fang_et_al_genotypes.txt | grep -E "(ZMMIL|ZMMLR|ZMMMR)" | sort | uniq -c
        290 ZMMIL
       1256 ZMMLR
         27 ZMMMR

    $ (head -n 1 fang_et_al_genotypes.txt; awk '$3 ~ /(ZMMIL|ZMMLR|ZMMMR)/ {print $0}' fang_et_al_genotypes.txt | \
    sort -k1,1V) > fang_et_al_genotypes_maize.txt

    $ wc -l fang_et_al_genotypes_maize.txt
    1574 fang_et_al_genotypes_maize.txt


    $ cut  -f 3 fang_et_al_genotypes.txt | grep -E "(ZMPBA|ZMPIL|ZMPJA)" | sort | uniq -c
        900 ZMPBA
         41 ZMPIL
         34 ZMPJA

    $ (head -n 1 fang_et_al_genotypes.txt; awk '$3 ~ /(ZMPBA|ZMPIL|ZMPJA)/ {print $0}' fang_et_al_genotypes.txt | \
    sort -k1,1V) > fang_et_al_genotypes_teosinte.txt

    $ wc -l fang_et_al_genotypes_teosinte.txt
    976 fang_et_al_genotypes_teosinte.txt
