# KPNNs
Knowledge-primed neural networks developed in the [Bock lab](http://medical-epigenomics.org) at [CeMM](http://cemm.at). KPNNs are neural networks that are trained using a knowledge-based network structure, which enables interpretability after training. Networks used in KPNNs should consist of nodes with labels (for example proteins in biological networks) that are connected based on prior knolwedge. After training, KPNNs enable extraction of node weights (importance scores) that represent the importance of individual nodes for the prediction.

[![DOI](https://zenodo.org/badge/190380526.svg)](https://zenodo.org/badge/latestdoi/190380526)

# System requirements
1. KPNNs were developed on linux and on Mac, and also tested on Windows 10 with Git Bash installed through Git for Windows (version 2.21.0.windows.1). 
2. Training of KPNNs is performed by a python script (KPNN_Function.py). This program has been developed and tested using python (tested on versions 2.7.6, 2.7.13, and 3.7.3) 
3. Downstream analysis is performed in R (tested on versions 3.2.3 or 3.5.1)

# Installation
1. Create a directory to contain all analyses:
      ```
	  mkdir KPNN/
	  cd KPNN/
      ```
2. Use a virtual environment (or [Conda](Conda.md)):
      ```
	  # pip install virtualenv # if not installed yet
	  mkdir virtenv
	  virtualenv virtenv --no-site-packages
	  # for python 3: virtualenv virtenv --no-site-packages -p python3
	  source virtenv/bin/activate
      ```
3. Installation instructions:
	  ```
	  # Clone this repository
	  git clone https://github.com/epigen/KPNN.git
	  cd KPNN/
	  
	  # Sets up environmental variables for the demo example
	  source Demo_setup.sh

	  # Install requirements (python and R)
	  pip install -r Requirements_python.txt
	  Rscript Requirements_R.R

	  # Generate test data and run test
	  Rscript Test_Data.R $KPNN_INPUTS
	  python KPNN_Function.py
      ```
4. The last command should finish without errors and you should see the message "KPNN TRAINING COMPLETED SUCCESSFULLY". Then installation is complete and KPNN training works.

# Demo to train one network
1. Download the Demo data from http://kpnn.computational-epigenetics.org/. If curl is set up on your system, you can do this using the scripts under Download_Data/.
      ```
	  sh Download_Data/Download_SIM1.sh 
      ```
2. To train a KPNN, run the python program with four arguments: (1) Input data, (2) an edge list, (3) class labels, (4) a path to store the outputs, for example:
      ```
	  data="$KPNN_INPUTS/SIM1_Data.csv"
	  edges="$KPNN_INPUTS/SIM1_Edgelist.csv"
	  classLabels="$KPNN_INPUTS/SIM1_ClassLabels.csv"
	  outPath=$KPNN_DEMOS
      python $KPNN_CODEBASE/KPNN_Function.py --alpha=0.001 --lambd=0.2 $data $edges $classLabels $outPath
      ```
3. The program should run for approximately 30 minutes or less. It will then produce a folder in the location defined by $outPath, where the output of the KPNN is stored. Outputs include:
  - tf_cost.csv - tracks training progress over iterations
  - tf_NumGradMeans.csv - Node weights (from numerical gradient estimation)
  - tf_NumGradTestError.txt - Test error of the latest saved model that was used to calculate node weights
  - tf_settings.csv - The hyperparameters used to train this model
  - tf_weights.csv - Edge weights of the final network
  - tf_yHat_val.csv - Predicted class labels on test data
  - tf_yTrue_val.csv - True class labels on test data
4. Once these outputs are provided, we can summarize the results. To do so, use the provided scripts and point it to the folder where the results above are found:
      ```
	  Rscript Analysis_Scripts/Analysis_SummarizeOneRun.R $KPNN_DEMOS/run_1/
      ```
5. The last command produces two plots in provided location.
  - TrainingCurve.pdf - plot of training progress. Loss on train and validation data is shown over time (training iterations)
  - NodeWeights.pdf - plot of node weights in the network.

# Training multiple network replicates
1. To train multiple KPNNs in parallel, examples scripts to do so are provided in the folder Slurm_Scripts/. These scripts are based on [SLURM](https://slurm.schedmd.com). The script to run simulated demo networks is provided in run_SIM_KPNN.sh.
2. After training is complete, to summarize test error across trained networks, use the R script Analysis_Scripts/Analysis_CollectOutputs.R
3. Finally, to summarize and plot node weights in trained networks, use the R script Analysis_Scripts/Analysis_SIM.R (this requires running of Analysis_CollectOutputs.R first).

# Instructions on how to run KPNNs on your data
1. Example scripts to run additional datasets, such as the dataset on T cell receptor (TCR) stimulation or on predicting cell types in the Human Cell Atlas (HCA), or provided in the folder Slurm_Scripts/.
2. All required inputs can be downloaded using the scripts in Download_Data/, except for single-cell expression data from the Human Cell Atlas, which should be downloaded under https://preview.data.humancellatlas.org.
3. After training, use scripts in Analysis_Scripts: Analysis_CollectOutputs.R to summarize the results across network replicates.
4. To adjust these analyses to your dataset, adjust the inputs to provide your data, class labels, and edgelist.
