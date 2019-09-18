#Generators

#MadGraph Setup

#clone madgraph setup

git clone https://github.com/muhammadgul/schools.git madgraph

`#for backup`

git clone https://github.com/IPNL-CMS/HTTMadgraphDocumentation.git madgraph
cd madgraph

# install the latest PDF
curl -O -L https://www.hepforge.org/archive/lhapdf/LHAPDF-6.1.6.tar.gz
tar xf LHAPDF-6.1.6.tar.gz
rm LHAPDF-6.1.6.tar.gz
cd LHAPDF-6.1.6
./configure --prefix=$PWD/..
# if you have boost problem, then use
#./configure --with-boost=/cvmfs/cms.cern.ch/slc7_amd64_gcc530/external/boost/1.63.0/include --prefix=$PWD/..

make -j8
make install
cd ..
#in case of lhapdf path problem use this export PATH=$PWD/bin:$PATH
# or this export LD_LIBRARY_PATH=$PWD/lib:$LD_LIBRARY_PATH
#download the proper pdf sets
./install_PDF_set.sh
#copy madgraph recent version
wget http://madgraph.physics.illinois.edu/Downloads/MG5_aMC_v2.6.0.tar.gz
tar xf MG5_aMC_v2.6.0.tar.gz
rm MG5_aMC_v2.6.0.tar.gz
###################################################################

#set the pdf path
path=$PWD/bin/lhapdf-config
sed "s@# lhapdf = lhapdf-config@$path@" MG5_aMC_v2_6_0/input/mg5_configuration.txt > MG5_aMC_v2_6_0/input/newmg5_configuration.txt
mv MG5_aMC_v2_6_0/input/newmg5_configuration.txt MG5_aMC_v2_6_0/input/mg5_configuration.txt

`Install pythia-pgs and Delphes`
run ./bin/mg5_amc 
run install pythia-pgs
run install Delphes

########################################################################################################
Running An example of W-boson mass: https://cp3.irmp.ucl.ac.be/projects/madgraph/wiki/TOPMassMeasurmentExample

import model sm
define p  = g u c d s u~ c~ d~ s~ b b~
define w = w- w+
define top = t t~
generate p p > top w z
output template_twz_madevents
launch
delphes=ON
>
>

Your root events file should be place at: template_twz_madevents/Events/run_01/tag_1_delphes_events.root
### change .root file to lhco manually
./root2lhco  template_twz_madevents/Events/run_01/tag_1_delphes_events.root  template_twz_madevents/Events/run_01/tag_1_delphes_events.lhco
## If you can't see root2lhco command, then made a symbolic link to it.
ln -s /ehep/ehep_data/mgul/test/madgraph/MG5_aMC_v2_6_0/Delphes/root2lhco

####### Preparing MadWeight Run:
run ./bin/mg5_aMC 
import model sm
define p  = g u c d s u~ c~ d~ s~ b b~
define w = w- w+
define top = t t~
generate p p > top w z
output madweight template_twz_madweight
exit
cd template_twz_madweight

# launch the madweight
./bin/madweight.py
## Sometimes it makes problem as in my case, but I solved it. You can see the problem and solution in this bug.
## https://bugs.launchpad.net/mg5amcnlo/+bug/1844041

# you can make changes in cards
# which transfer function to be used, just type:
change_tf
## different TF appears, so you can choose one of them.

## Further options with input .lhco file can be given as input
Set
nb_exp_events 2
Set
nb_event_by_node 2
Set
MW_int_points 1000
Set
MW_parameter 13 165 170 175 180 185
Set
precision 0.01
Set
inputfile ../template_twz_madevents/Events/run_01/tag_1_delphes_events.lhco
>

