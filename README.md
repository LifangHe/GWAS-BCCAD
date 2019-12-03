# GWAS-BCCAD



## Download the Data Files

Please download the image features (global and local connectivity metrics) in the [Data folder](https://github.com/elssam/GWAS-BCCAD/tree/master/Data)

## Feature Preparing

This script is meant to prepare the data for analysis. It:
* Import the individual files and get their global features,
* Do some descriptive statistics and plots
* Import the genes data and integrate them with the brain features

Input:
    Individual files (global), gene expression files.
Output:
    Save two datasets and important figures

"""
#############################################################################################################
### Import modules/Libraries

import pandas 
import matplotlib.pylab as plt
import seaborn
#import scipy.stats as stats
import glob

### Functions:
<pre><code>def combine(directory,visit):
    ''' This function combines csv files of MRI features in a directory - files should start with 0 (or 1) - directory should be the name of the directory you sotred your data; which normaly contains the subjects features of a specific visit and for either case/controls'''
    infiles=glob.glob(str(directory)+'/*'+str(visit)+"*")#glob.glob is similar to "ls" in linux command line, it saves all the files in current directory in a list
    outfile=pandas.DataFrame()
    for i in infiles:
        infile=pandas.read_csv(i)
        outfile=outfile.append(infile,ignore_index=True)
    return(outfile)
def add_column(dataframe,column_name,column_values):
    '''This function appends a column with a fix values at the end of it. Pass as parameter: dataframe,column_name,column_value'''
    dataframe_result=dataframe[:]
    dataframe_result[str(column_name)]=pandas.Series([str(column_values)]*len(dataframe))
    return(dataframe_result)
def append_both(dataframe1,dataframe2):
    '''This function takes two dataframes and append them into one'''
    return(dataframe1.append(dataframe2,ignore_index=True))
def calculate_diff(dataframe,column_name):
    '''This function calculates the difference between the metric in baseline and in follow-up'''
    before=" "+str(column_name)+"_"+"x"
    after=" "+str(column_name)+"_"+"y"
    dataframe[str(column_name)]=abs(dataframe[before]-dataframe[after])
    return(dataframe)</code></pre>
#############################################################################################################
### Combine the AD and Control subjects as followup/basline 

<pre><code>case='awdataontheway/AD_FA_02_binary'
control='awdataontheway/CONTROL_FA_02_binary'
print("Starting to work on the local features...")
AD_infile_followup = combine(case,"followup")
AD_infile_baseline = combine(case,"baseline")
Control_infile_followup=combine(control,'followup')
Control_infile_baseline=combine(control,'baseline')

#Creating the Visit column
AD_infile_followup = add_column(dataframe=AD_infile_followup,column_name="Visit",column_values="FOLLOWUP")
AD_infile_baseline = add_column(AD_infile_baseline,column_name="Visit",column_values="BASELINE")
Control_infile_followup=add_column(Control_infile_followup,column_name="Visit",column_values="FOLLOWUP")
Control_infile_baseline=add_column(Control_infile_baseline,column_name="Visit",column_values="BASELINE")

#Combining cases together, and controls together
AD_infile = append_both(AD_infile_followup,AD_infile_baseline)
Control_infile=append_both(Control_infile_followup,Control_infile_baseline)

#Creating the Status column
AD_infile= add_column(AD_infile,column_name="Status",column_values="AD")
Control_infile= add_column(Control_infile,column_name="Status",column_values="CONTROL")

#Combine both above to one MRI dataframe
DTI_infile=append_both(AD_infile,Control_infile)

#Save/Import to csv file
DTI_infile.to_csv("Local_Features_Duplicated.csv")

#Reshaping the data
Reshapred_Data = pandas.DataFrame() #It was Combined_Data
print("Reshaping the data to aggregate Status together, having two columns for baseline and follow-up:")
CONTROLS= pandas.merge(Control_infile_baseline,Control_infile_followup,on="Subject")
CONTROLS=add_column(dataframe=CONTROLS,column_name="Status",column_values="CONTROL")
CASES= pandas.merge(AD_infile_baseline,AD_infile_followup,on="Subject")
CASES=add_column(dataframe=CASES,column_name="Status",column_values="AD")
print("Done reshaing.\n\nNew columns of the reshaped data are:\n\n",)
Reshapred_Data = CASES.append(CONTROLS,ignore_index=True)
Reshapred_Data.to_csv("Local_Features_Reshapred.csv")

#Calculating the differencecs
NEW=pandas.read_csv("Local_Features_Reshapred.csv")
print(NEW.head())
calculate_diff(dataframe=NEW,column_name="transitivity ")
calculate_diff(dataframe=NEW,column_name="global_eff")
calculate_diff(dataframe=NEW,column_name="louvain")
calculate_diff(dataframe=NEW,column_name="char_path_len")
NEW.to_csv("Local_Features_Differences.csv")


#Adding MCI
mci='awdataontheway/EMCI_FA0.2_binary'
print("Starting to work on joining the MCI data...")
MCI_infile_followup = combine(mci,"followup")
MCI_infile_baseline = combine(mci,"baseline")

#Creating the Visit column
MCI_infile_followup = add_column(dataframe=MCI_infile_followup,column_name="Visit",column_values="FOLLOWUP")
MCI_infile_baseline = add_column(MCI_infile_baseline,column_name="Visit",column_values="BASELINE")

#Descriptive (sample size)
print("\n\nTotal of MCI subjects at follow-up is",len(MCI_infile_followup))
print("Total of MCI subjects at baseline is",len(MCI_infile_baseline))

#Combining cases together, and controls together
MCI_infile=append_both(MCI_infile_followup,MCI_infile_baseline)

#Descriptive to chek the previous sample size (supposed to be double the above)
print("Total of MCI, for both visits is",len(MCI_infile))

#Creating the Status column
MCI_infile= add_column(MCI_infile,column_name="Status",column_values="MCI")


#Combine both above to one MRI dataframe
DTI2_infile=append_both(DTI_infile,MCI_infile)
DTI2_infile.to_csv("Local_Features_Duplicated_withMCI.csv")

#Reshaping the data
print("Reshaping the data with MCI to aggregate Status together, having two columns for baseline and follow-up:")
MCIS= pandas.merge(MCI_infile_baseline,MCI_infile_followup,on="Subject")
MCIS=add_column(dataframe=MCIS,column_name="Status",column_values="MCI")
Reshapred_Data2 = Reshapred_Data.append(MCIS,ignore_index=True)
Reshapred_Data2.to_csv("Local_Features_Reshapred_withMCI.csv")

#Calculating the differencecs
NEW2=pandas.read_csv("Local_Features_Reshapred_withMCI.csv")
print(NEW2.head())
calculate_diff(dataframe=NEW2,column_name="transitivity ")
calculate_diff(dataframe=NEW2,column_name="global_eff")
calculate_diff(dataframe=NEW2,column_name="louvain")
calculate_diff(dataframe=NEW2,column_name="char_path_len")
print(NEW2.columns)
NEW2.to_csv("Local_Features_Differences_withMCI.csv")</code></pre>

