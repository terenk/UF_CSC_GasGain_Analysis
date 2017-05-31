   The  work flow description in the  gas gain study for 2016  CMS CSC data. 
                            May 10, 2017.

------------------------------------------------------------------------------
0. Introduction

- Goals
= "gas gain" measurements in each CSC/layer/HV segment using ntuples
  "gas gain" - trimmed MEAN of distribution of 3X3 ADC sum of charges measured 
               by ADC (in 3 time slices and 3 CSC cathode strips).
= corresponding HV corrections calculations if needed for the gas gain 
  uniformity
= CSC hit relative efficiency for monitoring
= CSC spatial resolution for monitoring

- The code
= C++ compiled and linked in ROOT (currently in ROOT 6.06/01)
= ROOT macros
= bash shell scripts to run C++ code in ROOT

- Initial setup
= currently ROOT environment is defined for CMS setup CMSSW_8_0_12  
To use ROOT after setting up CMSSW release do AFTER EACH LOGIN:
source /cvmfs/cms.cern.ch/cmsset_default.sh
cd $CMSSW/CMSSW_8_0_12/src  (folder $CMSSW is defined by user for CMSSW setup)
eval `scramv1 runtime -sh`
= there are two folders in $USERFOLDER ($USERFOLDER is path defined by user, 
  see script_setup in Work) and folder $OUT (also set by user) needed  for work:
  Work - scripts, ROOT *.C macros and other files 
  Src  - *.cxx, *.h code and ROOT *.C macros to compile and link the code.
  OUT - folders for outputs of AnalysisGasGain code (see below).
= in addition AFTER EACH LOGIN run script_setup in $USERFOLDER/Work

  source script_setup

  Modify it first time to put your pathnames  for data 
  and results to setup environment variables used in the code:
  USERFOLDER=yourpath
  WORK=$USERFOLDER/Work
  SRC=$USERFOLDER/Src
  OUT=your path to folder for results

- input data
=  ROOT ntuples produced by UF rootmaker on uf clusters
   For 2016H-PromptReco-v2 dataset the ntuples were made 
   by Hualin Mei (UF)
=  text files (lists of ntuples etc)

- the main analysis code AnalysisGasGain.cxx is running interactively 
  from script script_gasgain in Work folder as separate job for each input 
  ntuple (10-30 sec per job depending on input parameters in script_gasgain)
  and produces one ROOT file with histograms and one log file for each
  input ntuple in $OUT. Then all histogram ROOT files are joined to one output 
  ROOT file by ROOT hadd macros in correspondning scripts) for further analysis.
- there are other codes and  macros to analyze results of  AnalysisGasGain.cxx,
  see Src and Work folders.
------------------------------------------------------------------------------

1.Running  AnalysisGasGain code (see script script_gasgain)

- Make folders for output results, $OUTPUT,$OUTPUT/0000, $OUTPUT/0001, ...
  (if needed, according to folders 0000, 0001 ... where input ntuples are
  sitting) and folder $Prod (where the current script, code and service file
  will be saved), modify accordingly script_setup.

- Being in $SRC compile and link first time:
  = HistMan by build_histman.C
    root -b -q build_histman.C

  = AnalysisGasGain (also when it needed after  AnalysisGasGain code 
                     modifications) by macros build_analysisgasgain.C:
    root -b -q build_analysisgasgain.C

- Being in $WORK modify and run script_setup

  source script_setup

- Being in $WORK prepare input files (see script_gasgain in $WORK and examples
  there):

= lists of input ntuples for subfolders 0000, 0001 etc, like (for 2016 data):
  SingleMuon2016H_v2_0000_list.txt,
  SingleMuon2016H_v2_0001_list.txt
  Script script_gasgain has also possiblity to use list.txt having names of
  one or few input ntuples for debugging, see line  LIST=list.txt

= list of runs to be analyzed, like
  run_start_end_final_Run_3_2016H_v2.txt if you are going to analyze selected
  preliminary runs or
  run_start_end_final_Run_3_2016H_v2_nodata.txt if no selected runs required. 
  Then and all runs in ntuples will be analyzed.

= list with atm pressure like
  atmpres_unix_hours_2016H.txt if you need gas gain to be corrected for 
  atm. pressure or
  atmpres_unix_hours_2016H_nodata.txt if no correction is needed

= list of slopes of gas gain vs atm. pressure like
  gasgain_atm_slope_list_2016H.txt if you need gas gain to be corrected for 
  atm. pressure or
  gasgain_atm_slope_list_2016H_nodata.txt if no correction is needed

- Parameters in  script_gasgain
=     SET=0000  for folder 0000 in name of list of input ntuples and output 
                folder
=     STAT=0 all events analyzed ( if > 0 when STAT events is analyzed)
=     PRINT=0 no print of event number,  PRINT > 0 print event number in all 
      ntuple entries unless their number exceeds PRINT.

- flags in script_gasgain
FLAG_ATM  (0,1)   1 - to fill hists in method GetSumQAtmCorr
FLAG_TEST (0,1)   1 - to fill hists in folder "Test" of the output ROOT file in 
                      different methods for monitoring purpose
FLAG_HISTS (0,1)  1 - to fill hists in method FillSumQHists
FLAG_HISTSSEGM (0,1)  1 -  to fill hists in method FillSumQHists
FLAG_EFF (0,1)        1 - to call method GetEffHits 
FLAG_RESID (0,1)      1 - to call method CscResolution

Recommended combinations:
-------------------------------------------------------------------------------
 FLAG                 For gas gain      For efficiency     For resolution
------------------------------------------------------------------------------
 FLAG_ATM            1 (0 if no         0                  0
                        correction)
 FLAG_TEST           1                  0                  0
 FLAG_HISTS          1                  0                  0
 FLAG_HISTSSEGM      1                  0                  0
 FLAG_EFF            0                  1                  0
 FLAG_RESID          0                  0                  1
------------------------------------------------------------------------------
Note that these 3 combinations need to be run separately with outputs going
to different output folders, say GasGain, Eff, Resol in output directory defined
by user. See output examples in folders GasGain, Eff, Resol in
/raid/raid9/terentiev/Test3 (for one and the same input ntuple).

- run the code

source script_gasgain &

It takes about 4-6 hours to run the code for ~1000 input ntuples

- joining root files with histogram from one set (being in $WORK)

= modify script_sum_hists_one_dataset to put corresponding $SET
  (don't forget to run script_setup before that)
= run the script
source  script_sum_hists_one_dataset
Output will be 0000_hadd.root for SET=0000 in $OUT folder

- join  0000_hadd.root,  0001_hadd.root etc to ALL_hadd.root by script
  script_sum_hists

source script_sum_hists 

Output will be ALL_hadd.root in $OUT folder. This ALL_hadd.root is used as input
in various macros and codes analyzing results of AnalysisGasGain.
------------------------------------------------------------------------------

2. Getting CSC/layer/HV segment  gas gain
and HV corrections from output results of AnalysisGasGain.

- the code
= in $SRC - SumQTrimMean.cxx, SumQTrimMean.h and build_sumqtrimmean.C
  Compile and link:
  root -b -q build_sumqtrimmean.C
= in $WORK - sumqtrimmean.C and script_sumqtrimmean
  Run the script:
  source script_setup
  source script_sumqtrimmean

- input
= ALL_hadd.root (result of AnalysisGasGain) in $OUT
= files
== list of bad HV segments (bad_hvsegm_Run_3_2016H_v2.txt for 2016 data as 
                            an example).
   File bad_hvsegm_nodata.txt has no bad segments. Use it instead of file 
   with bad segments if there are no bad HV segments...
= list of cuts for HV correction and gas gain ratio outliers. 
  Example -  outliercuts_2016H_v2.txt. If the cuts are not needed, use
  outliercuts_nodata.txt

- output
= sumqtrimmean.log -  printout of gas gains and HV corrections for each 
  CSC/layer/HV segment (say where are HV corrections....
  In  sumqtrimmean.log see: 
  == In Step 3 - "Non ME11 HV corrections with inner and outer
        global reference gas gains G0"
  == In step 4 - "HV corrections with station/ring G0 gas gain references" see
     lines starting from "dhvME11" for ME11 HV corrections.
= sumqtrimmean.root
= Examples of output are in $WORK folder for input files given in 
script_sumqtrimmean:
DATA=/raid/raid9/terentiev/GasGain_Run_3/SingleMuonReco/2016H_v2/ALL_hadd.root
BADHVSGM=bad_hvsegm_Run_3_2016H_v2.txt
CUTS=outliercuts_2016H_v2.txt

View the output root file by ROOT in $WORK folder:
root view.C where view.C is
{
gStyle->SetPalette(1,0);
gROOT->SetStyle("Plain");

TFile f("sumqtrimmean.root");
f.GetListOfKeys()->Print();
f.ls();
TBrowser browser;
}
---------------------------------------------------------------------------
