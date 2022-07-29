<h1><center>Removal of Duplicated Sequences</center></h1>  


### Subsetting the MSA file
-------------------------------------------
2 loci have been chosen to test the script to make sure duplicates are removed correctly. L388, as seen in photo below, contains 2 duplicates that need to be removed. 
![L388](L388_dup_problem.png)

As for loci L389, there are 4 duplicates that need to be removed.  
![L389](L389_dup_problem.png)


```bash
%%bash 
grep 'L388' -A 1 sl_SINGLE_COPY_FOR_JM2.fas > JM2_L388.fas
subset1_num=`grep '>' JM2_L388.fas | wc -l`
echo 'number of sequences in loci L388 =' $subset1_num

grep -E 'L388|L389' -A 1 sl_SINGLE_COPY_FOR_JM2.fas > JM2_L388_L389.fas
subset2_num=`grep '>' JM2_L388_L389.fas | wc -l`
echo 'number of sequences in loci L388 and L389 =' $subset2_num
```

    number of sequences in loci L388 = 53
    number of sequences in loci L388 and L389 = 106



```bash
%%bash
head -n4 JM2_L388.fas
tail -n4 JM2_L388_L389.fas
```

    >L389|Trichoplusia|ni,||comp0_consensus
    CTGCACTTATCGTCGATGCCCTCGTAGGGGTAGGTCTTCTCGGTGTCGATGCCGCCGTTGTCCTTGATGTACTTAAAGGCGTTATCCATGAGGCCGCCGTTGCAGCCGTTGTTGCCGTACGCCGCCGAGCAGTCCACCAGGTTCTGCTCCGACAGAGACACCAGGTAGTGTGTCTTGCGGAAGTGCTGGCCCT-------------
    >L389|Xestia|xanthographa,||comp0_consensus
    CTGCACTTGTCATCGACGGCCTCGTACGGGTAAGACTTCTCGGTGTCGATGCCGCCGTTGTCCTTGATGTACTTGAAGGCGTTGTCCATGAGGCCACCGTTGCAGCCGTTGTTCCCGTACGCGGCCGAGCAGTCCACCAGATTTTGCTCCGACAGGGACACTAGGAAGCCGGTCTTGCGGAAGTGCTGGCCCTCC-----------
    >L388|Abrostola|tripartita,||comp0_consensus
    ---TGAAGGCCCAGCAGGAGCCGCACTTGCCCTGGTCCTTGACCTCAGTGACGGCGCCCTTCTTGCGCCAGTCCACCTGGTCGGGGTACGTCACGTGCGCGGGCGCAATGAACGTCGCGCCAC-----------------------------G
    >L388|Agrochola|circellaris,||comp0_consensus
    -----AACGCCCAGCAGGAGCCGCACTTGCCCTGGTCCTTGACATCGGTGACAGCGCCCTTCTTGCGCCAGTCCACCTGGTCGGGGTAGGACACGTGCGCCGGCGCGATGAAC----------------------------------------



```python
class Sequence:
    def __init__(self, filename=''):
        """
        Initializes the object
        
        param mode: string, path of file to be edited
        """
        self.filename = filename  # name of file to be read 
        self.seqdict = {}  # dictionary of sequence information ('seqname':genomic_sequence)
        self.total_dup = 0
        self.total_seq = 0 
        
        if filename:
            self.open_file('r')
            
    def open_file(self, mode):
        """
        Opens file contained in self.filename and sets the filehandle as self.file 
        If the file can't be opened, exit status will equal 1

        param mode: string, mode for opening file, usually 'r' or 'w'
        return: filehandle of opened file
        """
        try:
            self.file = open(self.filename, mode)
        except OSError:
            print(f'Error opening file ({self.filename})')
            # exit with status = 1
            exit(1)
            
    def next_line(self):
        """
        Returns the next line from a file
        
        return: logical, True if line is read, False when end of file 
        """
        line = self.file.readline().strip() 
        if not line: 
            # no more line, file is finished
            return False 
        
        self.line = line 
        return True  #return True when there is still line to be read 
    
    def read_file(self): 
        """
        description
        """
        prev_loci_num = -100
        seqnamelist = []  # list of seqnames (to find duplciates)
                
        while self.next_line():  # iterate through every lines in the file
            if self.line.startswith('>'):  # line containing seqname 
                self.total_seq += 1  # increase count of total number of sequences being processed 
                loci_num, seqname = self.line.split('|', 1)  # split to get loci number and seqname 
                if loci_num != prev_loci_num:  # new loci 
                    seqnamelist = []  # reset the list 
                prev_loci_num = loci_num
                if seqname in seqnamelist:  # duplicate
                    self.total_dup += 1
                if seqname not in seqnamelist:  # seqname is new (not a duplicate)
                    seqnamelist.append(seqname)  # add seqname to list of seqnames 
                    self.seqdict[self.line] = ''  # record the seqname 
            else:  # actual sequence of that loci  
                last_key = list(self.seqdict.keys())[-1]  # access the last key created in dictionary 
                self.seqdict[last_key] += self.line  # add line (sequence) as value of key 
                
    def save_output(self):
        """
        description 
        """
        out = 'duplicates_removed_'+self.filename  # name for output file 
        with open(out, 'w') as f:
            for key, value in self.seqdict.items():
                f.write(key)
                f.write('\n')
                f.write(value)
                f.write('\n')
        
        print("Results saved - program is finished.")
        print(str(self.total_seq) + ' sequences processed, ' + str(self.total_dup) + ' duplicates removed.')
```


```python
if __name__ == '__main__':
    JM2_subset1_path = 'JM2_L388.fas'  # path to the subsetted file 
    JM2_subset1 = Sequence(JM2_subset1_path)  # create Sequence object and open file
    JM2_subset1.read_file()  # process the file 
    JM2_subset1.save_output()  # save file
    
    
    JM2_subset2_path = 'JM2_L388_L389.fas'  # path to the subsetted file 
    JM2_subset2 = Sequence(JM2_subset2_path)  # create Sequence object and open file
    JM2_subset2.read_file()  # process the file 
    JM2_subset2.save_output()  # save file
```

    Results saved - program is finished.
    53 sequences processed, 2 duplicates removed.
    Results saved - program is finished.
    106 sequences processed, 6 duplicates removed.



```python
if __name__ == '__main__':
    JM2_path = 'sl_SINGLE_COPY_FOR_JM2.fas'  # path to the file 
    JM2 = Sequence(JM2_path)  # create Sequence object and open file
    JM2.read_file()  # process the file 
    JM2.save_output()  # save file
```

    Results saved - program is finished.
    42119 sequences processed, 746 duplicates removed.



```python

```
