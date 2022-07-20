<h1><center>Fingerprinting and Molecular Similarity of Drug Molecules</center></h1>  

### Introduction
--------------------------------------------------------------------------------
As part of my research in [Kihara Bioinformatics Laboratory](https://kiharalab.org/) at Purdue University, I was responsible for comparing the molecular similarity within and between different drug datasets. In the codes below, I used **[RDKit](https://www.rdkit.org/)**, an open-Source Cheminformatics Software, to generate molecular fingerprints of compounds, which are then used to calculate the similarities between compounds in different datasets. 

There are several similarity metrics that can be used to compute molecular similarity, but I used **Tanimoto similarity**, which is also the default metric used in RDKit. The distribution of computed similarities are then visualized with a histogram. To visually check the results, molecules from the 3 highest scoring and lowest scoring pairs were drawn using RDKit (_Note_: I was not able to display molecules within a pair next to each other, therefore each molecule is displayed 1 by 1).

This code can be adjusted to calculate similarities for other datasets containing SMILE strings, but the datasets I used below are: 
1. **COVID**: a dataset of potential COVID drug candidates collected from numerous papers. 
2. **generated_COVID**: a dataset of molecules generated by [ReLeaSE](https://github.com/isayev/ReLeaSE), a machine learning algorithm for de novo design of molecules. I used the COVID dataset above as input for the ML algorithm to generated this new drug dataset. 
3. **JAK2**: a drug dataset used to test the ReLeaSe algorithm in a [paper](https://www.science.org/doi/10.1126/sciadv.aap7885) by Popova et at.

The reason for each of the comparisons doen below: 
- *JAK2 vs. JAK2* & *COVID vs. COVID*: to use as a reference/base line - to find what a relatively 'normal' similarity scores distribution among a drug dataset should be (the scores from within the generated drug dataset will be compared with these baselines) 
- *COVID vs. generated_COVID* & *JAK2 vs. generated_COVID*: to see if the ML algorithm generated molecules based on the new inputs (COVID drug molecules), or if it was based on the JAK2 dataset that the paper used. 

### Setup
---------------------------------------------------------------

#### Installing RDKit using pip: 
RDKit can be installed using _conda_, or using pip like below


```python
# installing rdkit using pip:
pip install rdkit-pypi
```

#### Importing all the necessary libraries:


```python
import itertools
import random
import statistics
from rdkit import DataStructs
from rdkit import Chem
from rdkit.Chem import AllChem
from rdkit.Chem import Draw
import numpy as np
import matplotlib.mlab as mlab
from matplotlib import pyplot as plt
import pandas as pd
from scipy.stats import norm
from scipy.stats import mannwhitneyu
```

### Creating DrugData Class to handle the datasets: 
---------------------------------------------------------------
Below, I create a class called _Drugdata_. This class contains functions that will subset datasets, perform fingerprinting, calculate molecular similarities between each molecules in the subset, and visualize the results. Object oriented programming was used as the 3 datasets are to be analyzed the same way, even though the datasets themselves are different. 


```python
class DrugData:
    def __init__(self):
        #The attributes of each DrugData object (COVID, generated_COVID, JAK2 datasets):
        self.file_handle = None
        self.line = ''  # each smile string in the file
        self.smiles_list = []  # list of smiles string from dataset 
        self.subset_smiles = []  # smaller list of smiles for tanimoto calculation
        self.tanimoto_list = []  # list of tanimoto values
        self.highest3_tanimoto = {} # contains 3 highest scoring tanimoto
        self.lowest3_tanimoto = {}  # 3 lowest scoring tanimoto
        
    def open_file(self,fname):
        self.file_handle = open(fname)
        
    def next_line(self):
        # returns the next line in the file containing the drug data
        line = self.file_handle.readline().strip('\n').strip('"')
        if not line:
            return False
        self.line = line
        return True
    
    def read_COVID(self):
        """
        Reads both COVID and generated_COVID data - both files contain only SMILE strings then adds SMILES to a list
        
        example lines in the file:
            NC(=O)c1ccc2c(c1)C(=O)C(=O)N2Cc1ccccc1
            CCC(Sc1nc(O)c(C#N)c(-c2cccc(OC)c2)n1)C(=O)Nc1ccc(C(C)=O)cc1
        """
        while self.next_line():
            self.smiles_list.append(self.line)
    
    def read_JAK2(self):
        """
        Reads JAK2 data - contains both SMILES and pIC50, comma delimited - then adds SMILES to list
        
        example lines in the file:
            O=S(=O)(Nc1cccc(-c2cnc3ccccc3n2)c1)c1cccs1,4.26
            O=c1cc(-c2nc(-c3ccc(-c4cn(CCP(=O)(O)O)nn4)cc3)[nH]c2-c2ccc(F)cc2)cc[nH]1,4.34
        """
        while self.next_line():
            self.line.split(',')
            self.smiles_list.append(self.line.split(',')[0])
            
    def subset_list(self,subset_num):
        """
        Create a list that is a subset of the larger list
        input: subset_num = numbers of values/elements in subset 
        """
        self.subset_smiles = random.sample(self.smiles_list,subset_num)

    def tanimoto_sim(self):
        """
        Calculates tanimoto similarity between each pair of smiles in within a list
        Returns:
            - list of tanimoto coefficient for all pairs
            - dictionary of top 3 highest tanimoto coeff
            - dictionary of lowest 3 tanimoto coeff
        """
        hight,lowt = {-0.1:'',-0.2:'',-0.3:''},{1.1:'',1.2:'',1.3:''}
        for mol1, mol2 in itertools.combinations(self.subset_smiles,2):
            tmol1 = Chem.RDKFingerprint(Chem.MolFromSmiles(mol1))
            tmol2 = Chem.RDKFingerprint(Chem.MolFromSmiles(mol2))
            tanimoto = DataStructs.FingerprintSimilarity(tmol1,tmol2)
            self.tanimoto_list.append(tanimoto)  # add new tanimoto to list
            
            # get the highest tanimoto (for visual comparison)
            if tanimoto > min(hight,key=hight.get):
                hight[tanimoto] = mol1+','+mol2  # create new key and value
                del hight[min(hight,key=hight.get)]  # delete old key
            # get the lowest tanimoto:
            if tanimoto < max(lowt,key=lowt.get):
                lowt[tanimoto] = mol1+','+mol2  # create new key and value
                del lowt[min(lowt,key=lowt.get)]  # delete old key
        # return the lowest and highest tanimoto values along with their 
        # smiles pair
        self.highest3_tanimoto,self.lowest3_tanimoto = hight,lowt
        
    def dif_tanimoto_sim(self,list1,list2):
        """
        Calculates tanimoto similarity between each pair of smiles in 2 different lists
        Input: list1 = first list to compare; list2 = second list to compare
        Returns: (same as tanimoto_sim() function above)
        """
        hight,lowt = {-0.1:'',-0.2:'',-0.3:''},{1.1:'',1.2:'',1.3:''}
        for mol1 in list1:
            tmol1 = Chem.RDKFingerprint(Chem.MolFromSmiles(mol1))
            for mol2 in list2:
                tmol2 = Chem.RDKFingerprint(Chem.MolFromSmiles(mol2))
                tanimoto = DataStructs.FingerprintSimilarity(tmol1,tmol2)
                self.tanimoto_list.append(tanimoto)  # add new tanimoto to list
                
                # get the highest tanimoto (for visual comparison)
                if tanimoto > min(hight,key=hight.get):
                    hight[tanimoto] = mol1+','+mol2  # create new key and value
                    del hight[min(hight,key=hight.get)]  # delete old key
                # get the lowest tanimoto:
                if tanimoto < max(lowt,key=lowt.get):
                    lowt[tanimoto] = mol1+','+mol2  # create new key and value
                    del lowt[min(lowt,key=lowt.get)]  # delete old key
                    
        self.highest3_tanimoto,self.lowest3_tanimoto = hight,lowt

    def tanimoto_plot(self,bin_num):
        """
        Create a histogram of the distribution of all the tanimoto results and prints out the mean, median, and sd
        Input: bin_num = number of bins to be drawn in the histogram
        """
        t_list = self.tanimoto_list
        mu, bins, sigma = plt.hist(t_list, bin_num, density=1, alpha=0.5)
        mu, sigma = norm.fit(t_list)
        best_fit_line = norm.pdf(bins, mu, sigma)
        plt.plot(bins, best_fit_line)
        
        #recording mean and median:
        self.mean = round(statistics.mean(t_list),4)
        self.median = round(statistics.median(t_list),4)
        
        print('mean = '+str(self.mean))
        print('median = '+str(self.median))
        print('standard deviation = '+str(round(statistics.stdev(t_list),4)))
        
    def draw_top3(self,d):
        """
        Draws the top/bottom 3 molecules from dictionary + display tanimoto
        input: d = dictionary of molecules to be drawn (self.highest3_tanimoto or self.lowest3_tanimoto)
        """
        size = 100,100
        for tanimoto in d:
            print(round(tanimoto,4))
            smile_list = d[tanimoto].split(',')
            for smile in smile_list:
                m = Chem.MolFromSmiles(smile)
                fig = Draw.MolToMPL(m, size)
                
    def U_rank_test(self,list1,list2):
        """
        Mann-Whitney U test: non-parametric version of t-test for independent
        samples (test for non-normally distributed samples)
        Test of the null hypothesis that the distribution of sample x is the same
        as distribution of sample y (test of difference in location between 
        distributions)
        Confidence Interval of 95% to reject the null (=different distribution)
        P-value <= 0.05 --> statistically significant difference (reject null)
        P > 0.05 --> difference is not statistically significant (fail 2 reject)
        Input: 2 lists to be compared
        """
        U1,p = mannwhitneyu(list1, list2)
        nx, ny = len(list1), len(list2)
        U2 = nx*ny - U1
        if p <= 0.05:
            print("p-value of "+str(p)+" is lesser than or equal to 0.05")
            print("The difference is statistically significant")
        if p > 0.05:
            print("p-value of "+str(p)+" is greater than 0.05")
            print("The difference is not statistically significant")
```

### Molecular Similarity Comparisons of the datasets
-----------------------------------------------------------------------
#### List of comparisons done (click to jump to section): 
1. [JAK2 vs. JAK2](#JAK2)
2. [COVID vs. COVID](#COVID)
3. [generated_COVID vs. generated_COVID](#g_COVID)
4. [COVID vs. generated_COVID](#C_v_g_COVID)
5. [JAK2 vs. generated_COVID](#J_v_g_COVID)
6. [Result Summary](#result)

#### 1. JAK2 vs. JAK2: <a class="anchor" id="JAK2"></a>


```python
# CALCULATING TANIMOTO:
JAK2 = DrugData()  # create object
JAK2.open_file('jak2_data.csv')  # open the file
JAK2.read_JAK2()  # read the file to get list of SMILE strings
JAK2.subset_list(200)  # create subset of 200 molecules
JAK2.tanimoto_sim()  # calculate tanimoto coeff between pairs of molecules
```


```python
# HISTOGRAM of TANIMOTO SCORE DISTRIBUTION:
JAK2.tanimoto_plot(100)  # histogram with 100 bins
```

    mean = 0.4259
    median = 0.3955
    standard deviation = 0.1264



    
![png](output_9_1.png)
    



```python
# TOP 3 TANIMOTO PAIRS
JAK2.draw_top3(JAK2.highest3_tanimoto)
```

    0.9697
    0.9622
    0.9697



    
![png](output_10_1.png)
    



    
![png](output_10_2.png)
    



    
![png](output_10_3.png)
    



    
![png](output_10_4.png)
    



    
![png](output_10_5.png)
    



    
![png](output_10_6.png)
    



```python
# LOWEST 3 TANIMOTO PAIRS
JAK2.draw_top3(JAK2.lowest3_tanimoto)
```

    0.3506
    0.11
    0.0948



    
![png](output_11_1.png)
    



    
![png](output_11_2.png)
    



    
![png](output_11_3.png)
    



    
![png](output_11_4.png)
    



    
![png](output_11_5.png)
    



    
![png](output_11_6.png)
    


#### 2. COVID vs. COVID   <a class="anchor" id="COVID"></a>
Comparing known COVID drug molecules (from literature) - to use as baseline


```python
# CALCULATING TANIMOTO:
COVID = DrugData()  # create object
COVID.open_file('COVID_SMILES.csv')  # open the file
COVID.read_COVID()  # read the file to get list of SMILE strings
COVID.subset_list(200)  # create subset
COVID.tanimoto_sim()  # calculate tanimoto coeff for pairs
```


```python
# HISTOGRAM of TANIMOTO SCORE DISTRIBUTION:
COVID.tanimoto_plot(100)
```

    mean = 0.3519
    median = 0.3448
    standard deviation = 0.1547



    
![png](output_14_1.png)
    



```python
# TOP 3 TANIMOTO PAIRS
COVID.draw_top3(COVID.highest3_tanimoto)
```

    1.0
    0.8039



    
![png](output_15_1.png)
    



    
![png](output_15_2.png)
    



    
![png](output_15_3.png)
    



    
![png](output_15_4.png)
    



```python
# LOWEST 3 TANIMOTO PAIRS
COVID.draw_top3(COVID.lowest3_tanimoto)
```

    0.107
    0.1056
    0.1046



    
![png](output_16_1.png)
    



    
![png](output_16_2.png)
    



    
![png](output_16_3.png)
    



    
![png](output_16_4.png)
    



    
![png](output_16_5.png)
    



    
![png](output_16_6.png)
    


#### 3. generated_COVID vs. generated_COVID <a class="anchor" id="g_COVID"></a>
Comparing generated COVID molecules with each other


```python
# CALCULATING TANIMOTO:
g_COVID = DrugData()  # create object
g_COVID.open_file('generated_COVID_SMILES.txt')  # open the file
g_COVID.read_COVID()  # read the file to get list of SMILE strings
g_COVID.subset_list(200)  # create subset
g_COVID.tanimoto_sim()  # calculate tanimoto coeff for pairs
```


```python
# HISTOGRAM of TANIMOTO SCORE DISTRIBUTION:
g_COVID.tanimoto_plot(100)
```

    mean = 0.2456
    median = 0.2455
    standard deviation = 0.0956



    
![png](output_19_1.png)
    



```python
# TOP 3 TANIMOTO PAIRS
g_COVID.draw_top3(g_COVID.highest3_tanimoto)
```

    0.4967
    0.5448
    0.6009



    
![png](output_20_1.png)
    



    
![png](output_20_2.png)
    



    
![png](output_20_3.png)
    



    
![png](output_20_4.png)
    



    
![png](output_20_5.png)
    



    
![png](output_20_6.png)
    



```python
# LOWEST 3 TANIMOTO PAIRS
g_COVID.draw_top3(g_COVID.lowest3_tanimoto)
```

    0.0202
    0.0201
    0.0154



    
![png](output_21_1.png)
    



    
![png](output_21_2.png)
    



    
![png](output_21_3.png)
    



    
![png](output_21_4.png)
    



    
![png](output_21_5.png)
    



    
![png](output_21_6.png)
    


#### 4. COVID vs. generated_COVID <a class="anchor" id="C_v_g_COVID"></a>
Comparing generated COVID molecules with COVID molecules


```python
# CALCULATING TANIMOTO:
COVID_v_generatedCOVID = DrugData()  # create object
# calculate tanimoto coeff among pairs between 2 different datasets:
COVID_v_generatedCOVID.dif_tanimoto_sim(COVID.subset_smiles,g_COVID.subset_smiles)  
```


```python
# HISTOGRAM of TANIMOTO SCORE DISTRIBUTION:
COVID_v_generatedCOVID.tanimoto_plot(100)
# statistical test to see if there is a significant difference between tanimoto results of the 2 datasets
COVID_v_generatedCOVID.U_rank_test(COVID.tanimoto_list,g_COVID.tanimoto_list)  
```

    mean = 0.273
    median = 0.2748
    standard deviation = 0.1043
    p-value of 0.0 is lesser than or equal to 0.05
    The difference is statistically significant



    
![png](output_24_1.png)
    



```python
# TOP 3 TANIMOTO PAIRS
COVID_v_generatedCOVID.draw_top3(COVID_v_generatedCOVID.highest3_tanimoto)
```

    0.614
    0.4779
    0.4389



    
![png](output_25_1.png)
    



    
![png](output_25_2.png)
    



    
![png](output_25_3.png)
    



    
![png](output_25_4.png)
    



    
![png](output_25_5.png)
    



    
![png](output_25_6.png)
    



```python
# LOWEST 3 TANIMOTO PAIRS
COVID_v_generatedCOVID.draw_top3(JAK2.lowest3_tanimoto)
```

    0.3506
    0.11
    0.0948



    
![png](output_26_1.png)
    



    
![png](output_26_2.png)
    



    
![png](output_26_3.png)
    



    
![png](output_26_4.png)
    



    
![png](output_26_5.png)
    



    
![png](output_26_6.png)
    


#### 5. JAK2 vs. generated_COVID <a class="anchor" id="J_v_g_COVID"></a>
Comparing JAK2 molecules with generated COVID molecules (to see if the algorithm generated molecules based on JAK2)


```python
# CALCULATING TANIMOTO:
JAK2_v_generatedCOVID = DrugData()
JAK2_v_generatedCOVID.dif_tanimoto_sim(JAK2.subset_smiles,g_COVID.subset_smiles)
```


```python
# HISTOGRAM of TANIMOTO SCORE DISTRIBUTION:
JAK2_v_generatedCOVID.tanimoto_plot(100)
COVID_v_generatedCOVID.U_rank_test(JAK2.tanimoto_list,g_COVID.tanimoto_list)
```

    mean = 0.2912
    median = 0.2969
    standard deviation = 0.0974
    p-value of 0.0 is lesser than or equal to 0.05
    The difference is statistically significant



    
![png](output_29_1.png)
    



```python
# TOP 3 TANIMOTO PAIRS
JAK2_v_generatedCOVID.draw_top3(JAK2_v_generatedCOVID.highest3_tanimoto)
```

    0.6009
    0.5754
    0.5664



    
![png](output_30_1.png)
    



    
![png](output_30_2.png)
    



    
![png](output_30_3.png)
    



    
![png](output_30_4.png)
    



    
![png](output_30_5.png)
    



    
![png](output_30_6.png)
    



```python
# LOWEST 2 TANIMOTO PAIRS
JAK2_v_generatedCOVID.draw_top3(JAK2_v_generatedCOVID.lowest3_tanimoto)
```

    0.0215
    0.0173
    0.0133



    
![png](output_31_1.png)
    



    
![png](output_31_2.png)
    



    
![png](output_31_3.png)
    



    
![png](output_31_4.png)
    



    
![png](output_31_5.png)
    



    
![png](output_31_6.png)
    


### Result Summary <a class="anchor" id="result"></a>
------------------------------------------------------------------------------
The statistical tests (Mann-Whitney U test) showed that the differences between the generated COVID dataset and the 2 other datasets were significant. This shows that the ML did not generate molecules based on the JAK2 dataset. However, this also means that the generated dataset is different from that of the input COVID dataset. In addition, the mean for similarity scores between g_COVID dataset and JAK2 is higher than that of g_COVID and COVID (though this difference has not been tested for statistical significance).  
Below, the means of each comparisons done are presented in a table.


```python
# Create a dataFrame using dictionary
df=pd.DataFrame({"COVID":[COVID.mean,'',''],
                 "JAK2":['n/a',JAK2.mean,''],
                 "g_COVID":[COVID_v_generatedCOVID.mean,JAK2_v_generatedCOVID.mean,g_COVID.mean]})
df.index = ['COVID', 'JAK2', 'g_COVID',]  
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>COVID</th>
      <th>JAK2</th>
      <th>g_COVID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>COVID</th>
      <td>0.3519</td>
      <td>n/a</td>
      <td>0.2730</td>
    </tr>
    <tr>
      <th>JAK2</th>
      <td></td>
      <td>0.4259</td>
      <td>0.2912</td>
    </tr>
    <tr>
      <th>g_COVID</th>
      <td></td>
      <td></td>
      <td>0.2456</td>
    </tr>
  </tbody>
</table>
</div>



### Misc.
----------------------------------------------------------------------------------------

##### Sample Tanimoto Similarity (template to calculate tanimoto)
Below is a simple example for calculating tanimoto coefficient. The result is 1 because the molecule is being compared to itself, so the similarity is highest.  


```python
#Have a molecule/SMILES string:
m = Chem.MolFromSmiles('NC(=O)c1ccc2c(c1)C(=O)C(=O)N2Cc1ccccc1')
m
```




    
![png](output_36_0.png)
    




```python
n = Chem.RDKFingerprint(m)  # get the molecular fingerprint
DataStructs.FingerprintSimilarity(n,n)  # calculate tanimoto coefficient
```




    1.0



##### Checking the U-rank test
When comparing 2 datasets that are exactly the same, the result should be 1. 


```python
JAK2.U_rank_test(JAK2.tanimoto_list,JAK2.tanimoto_list)
```

    p-value of 1.0 is greater than 0.05
    The difference is not statistically significant
