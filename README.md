# ETT Analyzer 

The purpose of this repository is to create cmssw configuration files to perform ECAL Trigger Team tasks such as emulator development and general data / MC anaysis. This was originally setup to work with the data global tag 113X_dataRun2_relval_v1 in CMSSW_11_3_0_pre1.

## Odd Weights Filter and TP mode from DB

To run over events with even + odd filters and plot reconstructed ampltiudes, first perform a setup similar to the default:

	cmsrel CMSSW_11_3_0_pre1
	cd CMSSW_11_3_0_pre1/src
	cmsenv
	git cms-init
	git cms-merge-topic CMS-ECAL-Trigger-Group:ecal_double_weights_11_3_X 
	git clone git@github.com:CMS-ECAL-Trigger-Group/ECALDoubleWeights.git -b CMSSW_11_3_0_pre1
	scram b -j
	cd ECALDoubleWeights/ETTAnalyzer

This command will move your local CMSSW repository to the ecal_double_weights_11_3_X branch for odd weights emulator development. 

The odd weights branches of the ETT analyzer and SimCalorimetry repositories are listed above. After obtaining and scramming the relevant repositories, you can run the configuration in the ETTAnalyzer directory and run the even + odd filters on an event. The even filter weights will be obtained from the DB by default, but the Odd filter weights can be set with the local sqlite files, generated thanks to CondTools/Ecal scripts. 

The TP mode needs also to be read from a DB tag
    
	cmsRun conf_11_3_0_pre1.py Debug=1    TPinfoPrintout=1 maxEvents=10 \
	  TPModeSqliteFile=TPModes/EcalTPGTPMode_Run2_default.db TPModeTag=EcalTPG_TPMode_Run2_default \
	  OddWeightsSqliteFile=weights/EcalTPGOddWeightIdMap.db 
	

The available TPmodes are described in these slides https://indico.cern.ch/event/995229/contributions/4189814/attachments/2173034/3669116/21_01_18%20-%20ECAL%20Trigger%20meeting%20-%20Emulator%20configuration.pdf

## Generate a new TP mode

Just put the wanted options in a text file similar to EcalTPGTPMode_Run2_default.txt and then use the `updateTPGTPMode.py` script to generate a new condDB tag.  

	cmsRun updateTPGTPMode.py inputTxtFile=EcalTPGTPMode_Run3_zeroing.txt TPModeTag=test outputDBFile=my_fancy_tag.db

### Run 2 

To run with the Run 2 ECAL L1 configuration, the TPmode should be set to zero, shown in binary form in the following command's for visual purposes, as this sets all of the optional configuration flags to their defaults: 

	cmsRun conf_11_3_0_pre1.py Debug=1 TPModeSqliteFile=TPModes/EcalTPGTPMode_Run2_default.db TPModeTag=EcalTPG_TPMode_Run2_default TPinfoPrintout=1 maxEvents=1 OddWeightsSqliteFile=weights/EcalTPGOddWeightIdMap.db BarrelOnly=1 

This command will print various debug statements in the ECAL L1 data stream steps, run on one event, obtain odd filter weights from the sqlite file in the weights directory, and run only on the EB. 

### Zeroing configuration

To run with a candidate zeroing mechanism, configuration 1, the only thing that changes is the TPmode tag and sqlite file:

	cmsRun conf_11_3_0_pre1.py Debug=1 TPModeTag=EcalTPG_TPMode_Run3_zeroing  \
     TPModeSqliteFile=TPModes/EcalTPGTPMode_Run3_zeroing.db\
	 TPinfoPrintout=1 maxEvents=1 OddWeightsSqliteFile=weights/EcalTPGOddWeightIdMap.db BarrelOnly=1

This zeroing mechanism zeroes at the strip level. If a strip's odd filter returns a greater value than the even filter, the even filter output will not be included in the TCP sum. As long as the total odd filter energy among the TT strips is greater than the even filter energy sum, this will be flagged in the form of the EB infobit1, where the FGVB is replaced. This is useful for monitoring zeroing, but at the cost of removing the FGVB.  
