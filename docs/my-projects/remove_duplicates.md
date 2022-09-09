<h1><center>Removal of Duplicated Sequences</center></h1>  

### Introduction
-------------------------------------------
During the Anchored Hybrid Enrichment pipeline, the dataset that Dr. Martinez, a postdoc working in the Kawahara Lab at the Florida Museum of Natural History, was studying contained duplicates that complicated the phylogenetic analysis process. The multiple sequence alignment (MSA) data contained multiple <a href="https://www.genome.gov/genetics-glossary/Locus" target="_blank">loci</a> (sites/locations of a gene within a genome), each with different numbers of replicates within them.   
To remove the duplicates within each loci, an object oriented Python script was written to iterate through all the loci in a given file, and remove all but 1 copy of each sequences. The full script can be found on my <a href="https://github.com/nhwivo/rm-loci-dupl/blob/main/rm_loci_dupl.py" target="_blank">Github</a>. 

### Procedures
-------------------------------------------
#### Subsetting the MSA file
-------------------------------------------
2 loci, L388 and L389, were chosen to test the script to make sure duplicates are removed correctly. *L388*, as seen in the photo below, contained *2 duplicates* that needed to be removed. 
![L388](L388_dup_problem.png)

As for loci *L389*, there were *4 duplicates* that needed to be removed. Together, loci L388 and L389 had 6 duplicated sequences that needed to be removed. 
![L389](L389_dup_problem.png)


```bash
%%bash 
# counting the total number of sequences
cd ../data/dupl/
grep 'L388' -A 1 sl_SINGLE_COPY_FOR_JM2.fas > L388_subset.fas
subset1_num=`grep '>' L388_subset.fas | wc -l`
echo 'number of sequences in loci L388 =' $subset1_num

grep -E 'L388|L389' -A 1 sl_SINGLE_COPY_FOR_JM2.fas > L388_L389_subset.fas
subset2_num=`grep '>' L388_L389_subset.fas | wc -l`
echo 'number of sequences in loci L388 and L389 =' $subset2_num
```

    number of sequences in loci L388 = 53
    number of sequences in loci L388 and L389 = 106


#### Object Oriented Python script
To detect duplicates and remove all but one copy, a Python script was written and tested on the small subsets created in the section above.  
The general framework was as follow: 
1. Open the file that was given as the input. 
2. Iterate through every line in the file. 
    - The MSA file is a fasta file, therefore each sequences are made up of 2 lines. The first starts with `>` followed by the sequence's name, and the second contains the genomic sequence. 
3. Initiate all the variable, including: 
    - List of seqnames in a loci
    - Previous loci number
    - Dictionary to hold seqname and sequence information
3. Determine whether the line starts with `>`. If yes, then process the seqname as described below 
    - Determine the loci number from the seqname. 
        - The format of the seqname is as follows: `L389|name|of|organism`; therefore, it can be split by `|` and using the index of 0 to obtain the loci number. 
    - If the current loci number does not equal the previous loci number, that means a new loci started, therefore the list of seqnames needs to be emptied. 
    - Adding seqname into list of unique seqname if it is not already in the list and add the seqname as a dictionary key with an empty value, e.g. `{"L389|Agrochola|circellaris":""}.
4. If a line does not start with `>`, then it is a sequence of the previous line. Therefore it is added into the dictionary as the value of the last key in the dictionary. e.g. the sequence would be added to "seqname2" in the following dictionary: `{"L389|seqname1":"GTAGCTAGC", "L389|seqname2":""}`.
- Save the output into a new file.  

#### Testing the program on the 2 smaller subsets  
As seen, the number of duplicates removed by the script matched with the number of duplicates known in the subsetted sample. In addition, the names of the sequences removed were that from the duplicates seen in the screenshot in the data subsetting section.  
Note: --dprint is an optional parameter that prints the seqnames of the removed duplicates. 


```python
!../programs/rm_loci_dupl.py "../data/dupl/L388_subset.fas" --dprint
```

    53 sequences processed, 2 duplicates were removed.
    Duplicate sequences that were removed are:
    >L388|Hypena|proboscidalis,||comp0_consensus
    >L388|Mythimna|albipuncta,||comp0_consensus
    Saving results...
    Resulting file named 'duplicates_removed_L388_subset.fas' saved - program is finished.



```python
!../programs/rm_loci_dupl.py "../data/dupl/L388_L389_subset.fas" --dprint
```

    106 sequences processed, 6 duplicates were removed.
    Duplicate sequences that were removed are:
    >L388|Hypena|proboscidalis,||comp0_consensus
    >L388|Mythimna|albipuncta,||comp0_consensus
    >L389|Agrochola|macilenta,||comp0_consensus
    >L389|Euproctis|similis,||comp0_consensus
    >L389|Hypena|proboscidalis,||comp0_consensus
    >L389|Mythimna|albipuncta,||comp0_consensus
    Saving results...
    Resulting file named 'duplicates_removed_L388_L389_subset.fas' saved - program is finished.


#### Running the program on the actual dataset


```python
!../programs/rm_loci_dupl.py "../data/dupl/sl_SINGLE_COPY_FOR_JM2.fas"
```

    42119 sequences processed, 746 duplicates were removed.
    Saving results...
    Resulting file named 'duplicates_removed_sl_SINGLE_COPY_FOR_JM2.fas' saved - program is finished.


### Results
-------------------------------------------------
Originally, when the file containing duplicates were opened in Aliview, an alignment viewer and editor software, a pop-up window would warn of the duplicates. When the output file from processing the original file was opened in Aliview, the software did not detect any duplicates. In addition, the output file worked as a proper input into Dr. Martinez's phyogenetic analysis pipeline. 
