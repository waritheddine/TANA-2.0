# TANA-2.0
TANA (Transferring Annotation via Network AlignmenT) is a tool for inferring protein function which is based on our approach MAPPIN for GNA (Global
Network Alignment).

Written By: Warith Eddine DJEDDI (waritheddine@yahoo.fr),
            Sadok BEN YAHIA (sadok.ben@taltech.ee) and
            Engelbert MEPHU NGUIFO (engelbert.mephu_nguifo@uca.fr)

This README describes the usage of the command line interface of TANA. 
The executable TANA is compiled for Linux x86_64 platform.

The program TANA finds a global alignment of multiple input
networks. Given multiple networks with N1, N2,..., Nk nodes each,
it returns a matching between the input networks, each match corresponding to
best-matching nodes from the multiple networks.

To understand how to use the algorithm, let's start with an example of
pairwise alignment of two input networks. The multiple case is similar.


(1) Suppose the species are named 'A' and 'B'

(2) Create the graph files and results from the BLAST runs (between the
    nodes) in a single directory. You'll need 5 files:

  (2.1) Network files:
     You'll need A.pin and B.pin , tab-separated files
     where each line contains an interaction. For example, the first 5 lines of
     A.pin are:

     ====== BEGIN ========
     INTERACTOR_A INTERACTOR_B
     a0 a1
     a0 a2
     a0 a3
     a0 a4
     ====== END ========

      Columns are separated by tabs. The first line is a header line of the
     form as shown above. All other lines describe an interaction, one per
     line.

     There may be a third column which contains edge weights (0 < wt <= 1) 
     and in that case the header line should've a third column titled Weight_VAL.

  (2.2) Results of BLAST runs.
     You'll need to perform an the all-against-all run
     of BLAST between all the nodes of the two networks. You should store the results in 3 files:

        A-B.sim,  A-A.sim, B-B.sim

     The files contain, as their names indicate, the results of BLAST runs
     between species A & B, A & A, and B & B, respectively. IMPORTANT: for
     files containing Blast scores between two species, the filename should
     have the species names in lexicographic order, i.e., A-B.sim is
     expected, not B-A.sim.

     The first 5 lines of the A-B.sim file are:

     ====== BEGIN ========
     a0      b0      1
     a1      b1      1
     a2      b2      1
     a3      b3      1
     a4      b4      1
     ====== END ========

     Each line is of the form:
     <id1>  <id2>   <Bit-Score>



  (2.3)  Gene ontology (GO) File
 
  (2.4) GO annotation file which contains GO annotations for proteins in the input networks. The format of this GO annotation file should be compliant with the GO consortium.
     
    A) File: goa_uniprot_gcrp.gaf → This set contains all GO annotations for canonical accessions
      from the UniProt reference proteomes for all species, which provide one protein per gene.


(3) Create a file that specifies the file locations, species names etc.
   In the folder config/, the file "policy.input".

   For example, suppose we have three PPI networks: a.pin, b.pin, c.pin, GO association file for the 3 species: a.gaf, b.gaf, c.gaf,
   six blast e-value (or bitscore) files: a-a.sim, a-b.sim, ..., c-c.sim, then the policy file should looks like:

          --------------BEGIN OF POLICY--------------
                    PARAMETER VALUE
                    network   dataset/a.pin
                    network   dataset/b.pin
                    network   dataset/c.pin
                    goafile   data/goa/a.gaf
                    goafile   data/goa/b.gaf
                    goafile   data/goa/c.gaf
                    efile     dataset/a-a.sim
                    efile     dataset/a-b.sim
                    efile     dataset/a-c.sim
                    efile     dataset/b-b.sim
                    efile     dataset/b-c.sim
                    efile     dataset/c-c.sim
                    scorefile     dataset/score_composit.model
                    alignmentfile ./result/alignment_TANA_rnd1.data
                    logfile       ./result/measure_time.txt
          --------------END OF POLICY--------------

 

(4) Call the code. Here are samples:
    
   (4.1) Network files using blast e-value:
      
      ./tana -alignment -alpha 0.3 -nmax 1000 -temp 50 -thr 0.3 -numspecies 3 -numthreads 8 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/
    
   (4.2) Network files using blast bitscore:
      
      ./tana -alignment -alpha 0.3 -nmax 1000 -temp 50 -thr 0.3 -numspecies 3 -bscore true -numthreads 8 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/
      
   (4.3) Using the network neighbour and the shared GOA during the prediction process:
      
      ./tana -alignment -out -alpha 0.3 -nmax 3000 -bscore true -neighbour true -thrNt 0.9 -numspecies 9 -numthreads 4 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/
      
   (4.4) Using only the shared GOA during the prediction process:
           
      ./tana -alignment -out -alpha 0.3 -nmax 3000 -bscore true -numspecies 9 -numthreads 4 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/

   The options are as follows (you can also use the "-h" or "--help" flag):

     Usage: 
     Usage:
       ./tana -version|-records|-alignment|-analyse|-format [--help|-h|-help]
     [-alignmentfile str] [-alpha num] [-avefunsimfile str] [-beta num]
     [-bscore] [-distributionfile str] [-edgefactor num] [-eta num]
     [-formatfile str] [-neighbour] [-nmax int] [-numspecies int]
     [-numthreads int] [-orthologyfile str] [-out] [-randomfile str]
     [-recordsfile str] [-resultfolder str] [-task int] [-thrNt num]
        Where:
              --help|-h|-help
             Print a short help message
     -alignment
        Execute the alignment algorithm.
     -alignmentfile str
        The filename of alignment which is required to either create or analyse.
     -alpha num
        Parameter controlling how much topology score contributes to the alignment score. Default is 0.5.
     -analyse
        Make analysis on alignment result.
     -avefunsimfile str
        The filename for functional similarity of alignment.
     -beta num
        Probability for randomly picking out the alignment records. Default is 1.0
     -bscore
        Use bitscore as the similarity of edges.
     -distributionfile str
        Output file for log ratio distribution.
     -edgefactor num
        The factor of the power law normalization. Default is 0.1.
     -eta num
        Threshold imfactor which is used to exclude match-sets from a single species. Default is 1.0.
      -format
        Process input or output file into proper format.
      -formatfile str
        Profile of input parameters.
      -neighbour
        Use neighborhood similarity for each unannotated protein during the prediction process.
      -nmax int
        The parameter for SA algorithm N.
      -numspecies int
        Number of the species compared. Default is 4.
      -numthreads int
        Number of threads running in parallel.
      -orthologyfile str
        Training data for orthology model.
      -out
        Print the alignment result into file.
      -randomfile str
        Training data for random model.
      -records
        Generate alignment records file.
      -recordsfile str
        Records file for writing and reading. It is used to store the triplet edges.
      -resultfolder str
        The folder which was used as active folder in the data process.
      -task int
        Complete different tasks. The task was determined with other options such as -alignment together. Default is 1.
      -thrNt num
        Threshold factor which is used to exclude functional similarity below a given threshold between the unannotated protein its neighbours. Default is 0.7.
      -version
        Show the version number.

(5)  The output

      The main files are :
       (5.1) The output of the alignment is located in the file "result/alignment_TANA_rnd1.data". Each protein is represented 
       by a string (separated by a tab), and each interaction is on a single line.
       (5.2) The affectation from UniprotGOA for for each protein belonging to the N input networks is located 
       in the file "result/affectProtToGene.txt".
       (5.3) The number of unkown functional protein is located in the file "result/alignUnknown_rnd1.result".
       (5.4) The predictions for the unannotated protein using the "CAFA id" are in the file "Predictions/CAFA-Predicted-GOTerms.txt".
       (5.5) The predictions for the unannotated protein using the "Uniprot id" are in the file "Predictions/List-Predicted-GOTerms_rnd1".
       (5.6) The file "Predictions/scoreGOTerms_rnd1.txt" contains the scoring of GO Terms by transferring shared annotation to unannotated protein (marked with *) in each cluster belonging to the file "result/alignUnknown_rnd1.result".
       (5.7) The required time to align the input networks is located in the file "measure_time.txt".
       (5.8) The sequence and functional similarities for each pair of compared protein 
       is located in the file "result/scoreRecords.txt".
       (5.9) The informations : Alignment edges conserved for each species, the number of alignment nodes, 
       the Alpha parameter, nmax and the Alignment score, are located in the file "result/alignment_statistics.data".
       (6.1) The number of unknown alignment records, and the number of proteins annotated with MF,BP and CC in alginment graph, are located in the file "result/statistics.result".
       (6.2) The Mean Entropy and Mean Normalized Entropy value are located in the file "result/Metrics.result".
       (6.3) The similarity between each GO Terms for a given cluster (resulted from the alignment of PPI) using the relevance metric is located in the file "result/scoreGOTerms_rnd1.txt"
       (6.4) The fnctional similarity between each unannotated protein and its neighbours is loacated in the file "result/ComplexProteique.result".
       (6.5) The prediction of GO Terms for each unannotated protein (with uniprot ID) is located in the files "Predictions/List-Predicted-GOTerms.txt" or in "Predictions/List-Predicted-GOTerms_rnd1.txt".
       (6.6) The prediction of GO Terms for each unannotated protein (with CAFA ID) is located in the file "CAFA-Predicted-GOTerms_rnd1".
(7)  EXAMPLE
       
       (7.1) Download TANA freely available at Github website: https://github.com/waritheddine/TANA-2.0

       (7.2) Run TANA on our test dataset with command:
       
       A) Prediction process based on transferring GOA from the shared GO terms and from the direct network neighbours:
       ./tana -alignment -out -alpha 0.3 -nmax 3000 -bscore true -neighbour true -thrNt 0.9 -numspecies 9 -numthreads 4 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/
       
       B)Prediction process based on only on transferring GOA from the shared GO terms :
       ./tana -alignment -out -alpha 0.3 -nmax 3000 -bscore true -numspecies 9 -numthreads 4 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/
       
       (7.3)Then you can find the all the involved output files in ./result/ and prediction files in ./Predictions/. There are many other functions which you can see with "-help" option.
       (7.4) We note that checking the format of the files in the data folder after reading the execution instructions above might be quite helpful.
       
