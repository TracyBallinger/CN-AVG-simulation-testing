# CN-AVG-simulation-testing

#This is a description of the commands used to generate simulated evolutionary histories, run the CN-AVG pipeline on the known histories, and then calculate accuracy statistics for those predictions. 
# $cnbin is the location of the cnavg_post repository. 
## Files to include for reproducibility. ### 
specs.txt
precision_recall_f1_vs_connectivity_plots.R
likelihood_score_histogram_plot.R
allevents.dat
simevntsp50.txt
simedgesp00.txt

#The commands are the commands that I used to create the simulations used in the paper. 
> mkdir intsims
> cd intsims
## 1.  Generate simulated evolutionary histories. 
> $cnbin/make_simulations_jobtree.py --specfile specs.txt --reps 5 --outputdir cnavgout --integer --id 1 --steps 2000 --runs 10
# This will create subdirectories in cnavgout, one for each simulation. The files necessary for the next step are the HISTORY_STATS_* files and the HISTORIES_*.braney files.  

## 2. Merge the CN-AVG output to a single file of primary flows.  
> ls -d ./cnavout/sim* | awk '{split($1, a, "/"); split(a[2], b, ".");print $1"\t"a[2]"\t"b[1]"\t"b[2]}' | sed 's/sim//3' > simdirs.list

> awk '{print "cnavg_post_analysis_jobtree.py --logInfo --logFile "$1".log --jobTree "$1".jobtree --maxThreads 14 --cnavgout "$1" --outputdir "$1" --simulation --trueID 0 --sampleid "$2" --numruns 10 --refhistoryid 2000 --numsteps 2000 --stepsize -1"}' simdirs.list > postjobs.sh

# make postjobs.sh executable 
> chmod 744 postjobs.sh
> mkdir cnavgpost
> ./postjobs.sh
# This will create merged events in cnavgpost, again creating one directory for each simulated evolutionary history. 

# Each directory has the following files. 
# 1. sampleID.pevnts : A pickled file of the merged events (primary flows) 
# 2. sampleID.pedgs : A pickle file of the merged edges (segments and adjacencies that make up the primary flows). 
# 3. historystats.txt : a tsv file containing rearrangement costs (min and max bounds) for each of the histories predicted by the CN-AVG MCMC sampling.  Also, the penalty costs, and length (number of primary cycles or events) in each history.  
# 4. breakpoints.txt 
# 5. events.stats : has the number of TP, etc. events in it. 
# 6. events.dat: an unpickled file with human readable information about the events.  
# 7. edges.dat, edges.stats : same files as 5 and 6, but for the edges. 

# create merged data files. 
> cat ../cnavgpost/sim*/events.dat | grep -v ^event_id > tmp
> head -1 ../cnavgpost/sim100.10.1.0/events.dat | cat - tmp > allevents.dat
> $cnbin/simulations/get_simulations_data.py --datatype edges --pvalcutoff 0 simdirs.list simedgesp00.txt
> $cnbin/simulations/get_simulations_data.py --datatype events --pvalcutoff 0.5 simdirs.list simevntsp50.txt

# The figures were created using the following R scripts. 
 likelihood_score_histogram_plot.R
 precision_recall_f1_vs_connectivity_plots.R
