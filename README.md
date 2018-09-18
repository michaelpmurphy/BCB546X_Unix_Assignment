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


Using `head` I inspected the first 10 lines of each file, neither of which began with any comment lines (results show first lines only due to length):


    $ head fang_et_al_genotypes.txt snp_position.txt
    Sample_ID       JG_OTU  Group   abph1.20        abph1.22        ae1.3   [...]
    SNP_ID  cdv_marker_id   Chromosome      Position        alt_pos [...]

The data in each file appeared to be tab-delimited with the first rows consisting of column headings, so I used `Awk` to count the number of columns in the first row (and verified the same count for another row mid-file). I also used `cut` to try and find any rows that might contain data beyond that number of columns:

    $ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
    986

    $ awk -F "\t" 'NR==1783 {print NF; exit}' fang_et_al_genotypes.txt
    986

    $ cut -f 987-997 fang_et_al_genotypes.txt | sort | (head -n 1; tail -n 1)
    # Output lines blank.


    $ awk -F "\t" '{print NF; exit}' snp_position.txt
    15

    $ awk -F "\t" 'NR==783 {print NF; exit}' snp_position.txt
    15

    $ cut -f 16-30 snp_position.txt | sort | (head -n 1; tail -n 1)
    # Output lines blank.



##2. Data Processing




