<h2><center>Molecular Docking of ligands from DockCoV2 Database and generated ligands</center></h2>

### Introduction
---------------------------------------------------------------
As part of my research in the <a href="https://kiharalab.org/" target="_blank">Kihara Bioinformatics Laboratory</a> at Purdue University, I was responsible for the following tasks below:
- Replicate the docking that was performed in the [DockCoV2](https://academic.oup.com/nar/article/49/D1/D1152/5920447)  database 
- Perform docking of molecules generated from the [ReLeaSE](https://github.com/isayev/ReLeaSE) machine learning algorithm
- Compare the docking scores between the 3 datasets (DockCoV2 scores, replicated docking scores, and generated molecules docking scores)  

**<a href="https://vina.scripps.edu/" target="_blank">Autodock Vina</a>**, an open-source program for doing molecular docking, was used for all the dockings performed. ***Bash*** scripts were used to manipulate data for the docking procedures, and ***R*** was used to visualize the docking results. In addition, ***SLURM*** scripts were written to submit docking jobs to Purdue's HPC.  

### Pre-process files downloaded from DockCov2
---------------------------------------------------------------
###### From DockCoV2's [download](https://covirus.cc/drugs/downloads) page, the following information was retrieved:
1. [6lu7](https://www.rcsb.org/structure/6lu7) protease .pdbqt file - this is the target protein for docking
2. A config file - this provides information/parameters for the docking, such as: grid box coordinates, grid box dimensions, exhaustiveness, etc.   
    *Note:* exhaustiveness was changed from 150 to 8 due to computational limitations; energy_range was changed from 10 to 4; num_modes was changed from 20 to the default of 9  
3. Best pose (docking score) of all of the ligands available in the databse against 7 different Sars-CoV2 proteins.  

The downloaded "best_pose.tar.gz" folder was extracted using `tar -xf best_pose.tar.gz`. The extracted folder contains subdirectories for each ligand, each of which has their own subdirectories storing the docking score against 7 different Sars-CoV-2 proteins.  

###### The codes below process the downloaded files so that:
1. Only the docking scores against the main protease (6lu7) are used in the analysis 
2. The ligand files are in the right format to be docked using Autodock Vina.


```bash
%%bash 
# find the total number of ligands downloaded from DockCoV2
find best_pose/ -mindepth 1 -maxdepth 1 -type d | wc -l
```

    3077



```bash
%%bash 
# purpose: moves all .pdbqt files for protease protein to a separate folder as downloaded ligand 
# files downloaded from database also contains .pdbqt files for docking against other proteins.

# make new directory for .pdbqt files of docking scores of ligands against protease protein
mkdir dock_protease_ligands

for ligand_dir in best_pose/*; do  # loop over every directory in best_pose
    # get the file name of the protease .pdbqt files 
    ligand_fname=$ligand_dir/ALL_protease_1/*
    # copy the files over to the newly created folder
    cp $ligand_fname protease_ligands
done
```

In the code block below, 3078 protease ligands are separated into smaller directories for docking. The docking of ligands is performed by submitting a SLURM script/job to Purdue's HPC. As a student, I have access to Scholar, a small computer cluster, thus the computing time I get for per submission is limited. Therefore the ligands were split into smaller groups of 100, as docking all 3,000 ligands would exceed the alotted computing time. In addition, the ligand files are also modified for docking - by removing the first 2 and the last line.


```bash
%%bash
# 3078 ligand files -> 100 ligands each folder = 31 folders total

cd dock_protease_ligands/

for dir_num in {1..31}; do  
    mkdir $dir_num  # make new directory 
    x=0  # counter for ligand files
    for ligand_file in *.pdbqt; do  # loop over each ligand (has .pdbqt extension)
        if [[ $x == 100 ]]  # no more than 100 ligands per folder
        then
            break
        fi
        
        # modify the ligand file for docking (removing the first 2 and last line)
        sed -i '1,2d' $ligand_file
        sed -i '$d' $ligand_file 
        # move the ligand file to the appropriate folder
        mv $ligand_file $dir_num/
        ((x++))  # increment x by 1
    done
done

cd ../
```

### Pre-process files generated from ML algorithm
---------------------------------------------------------------
From the machine learning algorithm [ReLeaSE](https://github.com/isayev/ReLeaSE), a dataset of SMILE strings was generated. A fellow lab member has converted this dataset from SMILE format into .pdbqt for docking.


```bash
%%bash 
# find the total number of files (number of ligands to dock)
find dock_generated_ligands/ -mindepth 1 | wc -l
```

    1367



```bash
%%bash
# purpose: similar to the DockCov2 ligands, the generated .pdbqt files are separated
# into smaller subsets to accomodate limited computing time on Purdue's cluster
# 1367 ligand files -> 100 ligands each folder = 14 folders total

cd dock_generated_ligands/

for dir_num in {1..14}; do
    mkdir $dir_num  # make new directory 
    x=0
    for ligand_file in *.pdbqt; do  # loop over each ligand (has .pdbqt extension)
        if [[ $x == 100 ]]  # no more than 100 ligands per folder
        then
            break
        fi

        # move the ligand file to the appropriate folder
        mv $ligand_file $dir_num/
        ((x++))  # increment x by 1
    done
done

cd ../
```

### Docking the Ligands
---------------------------------------------------------------
Slurm jobs (script below) were sent to Purdue's Scholar HPC cluster to process each of the 31 DockCoV2 ligands directories, and each of the 14 directories containing generated ligands. The slurm scripts were submitted using the command `sbactch run_docking.sub`. Before the scripts were submitted, files necessary for docking were copied into each directory.


```bash
%%bash
# copying the necessary files into each directory 
for each_dock_dir in dock_*/; do  # loop over all the directories 
    cd $each_dock_dir  
    for each_dir in */; do  # go through every numbered directories
        cp ../vina $each_dir/  # autodock vina
        cp ../protease_DockCoV2.pdbqt $each_dir/  # receptor file 
        cp ../config_protease_DockCoV2.txt $each_dir/  # config file
        cp ../run_docking.sub $each_dir/  # slurm script to run the docking 
    done
    cd ../
done
```


```bash
%%bash
# slurm script for the docking
cat run_docking.sub
```

    #!/bin/bash
    #SBATCH --job-name=autodockvina_DockCoV2_protease
    #SBATCH --account=scholar
    #SBATCH --mail-user=vo21@purdue.edu
    #SBATCH --mail-type=END,FAIL
    #SBATCH --nodes=1
    #SBATCH --ntasks=20
    #SBATCH --time=0-04:00:00     
    #SBATCH --output=%x.o%j
    #SBATCH --error=%x.e%j
    
    source /etc/profile.d/modules.sh
    
    # variable for config file - change accordingly 
    CONFIG="config_protease_DockCoV2.txt"
    # create new directory for storing all docking results
    mkdir docking_outputs
    # create path to output directory
    output_path="docking_outputs/"
    
    # loop to dock each ligand file in the current directory
    for ligand_file in *.pdbqt; do
    	if [ "$ligand_file" != "protease_DockCoV2.pdbqt" ]  # exclude the protein file
    	then
    		# get ligand name without '.pdbqt'
    		ligand_name=`basename $ligand_file .pdbqt`
    		# create names for output files
    		out_name=$ligand_name'_out.pdbqt'
    		log_name=$ligand_name'_log.txt'
    
    		# make directory to store ligand docking output	
    		mkdir -p $output_path$ligand_name
    		
    		# perform the docking
    		./vina --config $CONFIG --ligand $ligand_file \
    		--out $output_path${ligand_name}/$out_name \
    		--log $output_path${ligand_name}/$log_name
    	fi
    done 

### Processing the docking outputs/scores
---------------------------------------------------------------
Only the docking score (which is on the second line of the output file) of each ligand is needed for this analysis. The codes below obtain the docking scores from the dockings performed above, and also the docking scores provided by DockCoV2.
#### Replicate procedure and generated ligands docking scores:


```bash
%%bash
# purpose: to obtain docking scores of replicate docking, and docking of generated compounds

for each_dock_dir in dock_*/; do  # for each of the directory containing dosking results (there are 2)
    cd $each_dock_dir
    echo '#'$'\t''Docking_Score' > top_scores.txt  # create new files with column names

    for each_dir in */; do  # access each of the numbered directories (14 and 31)
        cd $each_dir"docking_outputs/"
        for each_ligand_dir in */; do
            fname=$each_ligand_dir*.pdbqt  # name of the output file   
            fname=`echo $fname`  # convert to string(?) - not sure why but this is needed to obtainligand_name below
            ligand_name=$(basename "$fname" _out.pdbqt)  # get identifier for ligand
            string=($`sed -n '2p' $fname`)  # get the second line from the output file, then turn it into an array
            score=${string[3]}  # access the 3rd element of the array, which is the docking score
            echo $ligand_name$'\t'$score >> ../../top_scores.txt  # write data into .txt file
        done
        cd ../../
    done

    cd ../
done
```

#### DockCoV2 docking scores:


```bash
%%bash
# purpose: to obtain the docking scores performed by DockCov2 and the names of docked ligands, then store the information. 
cd protease_ligands_DC2/
echo '#'$'\t''DC2 Docking Score' > DC2top_scores.txt  # create new files with column names
for ligand_file in *_1.pdbqt; do
    ligand_name=$(basename "$ligand_file" .pdbqt)  # get ligand name without extension
    string=($`sed -n '2p' $ligand_file`)  # get the second line from the output file, then turn it into an array
    score=${string[3]}  # get the docking score
    echo $ligand_name$'\t'$score >> DC2top_scores.txt  # add data to file
done
cd ../
```

## R codes for data visualization
##### Setup:


```python
# load in R to run in jupyter notebook 
import os
os.environ['R_HOME'] = '/apps/spack/scholar/fall20/apps/r/4.1.2-gcc-6.3.0-juf2pke/rlib/R'
# os.environ['R_HOME'] = '/home/vo21/R/x86_64-pc-linux-gnu-library/3.6'
%load_ext rpy2.ipython
# %reload_ext rpy2.ipython
```


```r
%%R
# load in libraries
suppressMessages(library(tidyverse))
suppressMessages(library(magrittr))  # pipe operator
suppressMessages(library(dplyr))  # data manipulation
suppressMessages(library(ggplot2))  # plotting & data
library(knitr)  # for working with kables
```

##### Reading in the docking score data files:


```r
%%R
# purpose: to read in the tab delimited docking score files:

# file names (path to the files):
dc2_fname <- "protease_ligands_DC2/DC2top_scores.txt"
replicate_fname <- "dock_protease_ligands/top_scores.txt"
generated_fname <- "dock_generated_ligands/top_scores.txt"

# reading in the tab delimited files:
suppressMessages(dc2 <- read_delim(file=dc2_fname, delim='\t', col_names=TRUE))
suppressMessages(replicated <- read_delim(file=replicate_fname, delim='\t', col_names=TRUE) %>%
                rename(., replicated_DS = Docking_Score))
suppressMessages(generated <- read_delim(file=generated_fname, delim='\t', col_names=TRUE) %>% 
                rename(., generated_DS = Docking_Score))
```

### Summary data of docking scores
---------------------------------------------------------------


```r
%%R
num <- 1:6
sumt <- tibble(num)
sumt <- add_column(sumt, Summary_generated=summary(generated %>% select(generated_DS)))
sumt <- add_column(sumt, Summary_replicated=summary(replicated %>% select(replicated_DS)))
sumt <- add_column(sumt, Summary_DockCoV=summary(dc2 %>% select(`DC2 Docking Score`)))
kable(sumt)
```

    
    
    | num|Summary_generated |Summary_replicated |Summary_DockCoV |
    |---:|:-----------------|:------------------|:---------------|
    |   1|Min.   :-10.000   |Min.   :-9.900     |Min.   :-10.200 |
    |   2|1st Qu.: -7.200   |1st Qu.:-7.000     |1st Qu.: -7.100 |
    |   3|Median : -6.500   |Median :-6.300     |Median : -6.400 |
    |   4|Mean   : -6.543   |Mean   :-6.136     |Mean   : -6.286 |
    |   5|3rd Qu.: -5.900   |3rd Qu.:-5.400     |3rd Qu.: -5.500 |
    |   6|Max.   : -2.600   |Max.   :-2.300     |Max.   : -2.300 |


### Perform statistical test (t-test)
---------------------------------------------------------------
To test whether the 2 datasets (DockCoV2 docking scores and replicate docking scores) are drawn from populations with different means. 
- The test assumes equal variance, therefore boxplots were used to visually check the assumption of equal variances. It can be seen that the variance is somewhat equal between the 2.   

##### Box plot to test equal variance assumption:


```r
%%R
# obtain the scores from each dataset
replicatedx <- replicated$replicated_DS
dc2x <- dc2$`DC2 Docking Score`
generatedx <- generated$generated_DS

# plotting the boxplots next to each other
lmts <- range(replicatedx, dc2x)
par(mfrow = c(1, 2))  # allows for multiple plots in a single row/line
#plot the data
boxplot(replicatedx,ylim=lmts, font.main = 1,
        main='Boxplot of replicate \ndocking scores', 
        ylab='docking score')
boxplot(dc2x,ylim=lmts, font.main = 1,
        main='Boxplot of DockCoV2 \ndocking scores')
```


    
![png](output_23_0.png)
    


##### T-test analysis
The p-value of the t-test is 1.803e-06 (which equals 0.029) is less than alpha=0.05.
- Therefore the null hypothesis is to be rejected, meaning _the docking scores between the 2 datasets are different_


```r
%%R
t.test(combined$`Docking Score`, combined$`DC2 Docking Score`, 
       alternative = "two.sided", var.equal=TRUE)
```

    
    	Two Sample t-test
    
    data:  combined$`Docking Score` and combined$`DC2 Docking Score`
    t = 4.779, df = 6124, p-value = 1.803e-06
    alternative hypothesis: true difference in means is not equal to 0
    95 percent confidence interval:
     0.08803608 0.21049477
    sample estimates:
    mean of x mean of y 
    -6.136108 -6.285374 
    


### Distribution of docking scores
---------------------------------------------------------------


```r
%%R
par(mfrow = c(2, 2))  # to place the charts next to each other 
hist(generatedx, breaks=50, font.main = 1,
     main='Histogram of generated ligand \n docking scores')
hist(replicatedx, breaks=50, font.main = 1,
     main='Histogram of replicate docking scores')
hist(dc2x, breaks=50, font.main = 1,
     main='Histogram of DockCoV2 \n docking scores')
```


    
![png](output_27_0.png)
    


### Overview of all plots
---------------------------------------------------------------


```r
%%R
par(mfrow = c(3, 2))
hist(generatedx, breaks=50, font.main = 1,
     main='Histogram and boxplot \n of generated ligand docking scores')
boxplot(generatedx, horizontal = TRUE)

hist(replicatedx, breaks=50, font.main = 1,
     main='Histogram and boxplot \n of replicate docking scores')
boxplot(replicatedx, horizontal = TRUE)

hist(dc2x, breaks=50, font.main = 1,
     main='Histogram and boxplot \n of DockCoV2 docking scores')
boxplot(dc2x, horizontal = TRUE)
```


    
![png](output_29_0.png)
    

