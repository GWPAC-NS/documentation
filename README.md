# documentation
This is an outline on the process of adding a new parameter to the input waveforms when running a banksims as well as creating the workflow and submitting the job. For this I’m going to assume you have a working version of lawsuit already set up, the instructions for which can be found at

http://ligo-cbc.github.io/pycbc/latest/html/install.html#installing-lalsuite-into-a-virtual-environment.  
Be sure you have a working version of lalsuite installed in a virtual environment before continuing. All instructions assume you are in this environment.

First, you want to follow instructions here https://help.github.com/articles/fork-a-repo/ to "fork" both the pycbc and pycbc-glue repositories into your git-hub account. We will be making edits to these repositories and eventually checkout these versions of pycbc/pycbc-glue with these instructions http://ligo-cbc.github.io/pycbc/latest/html/install.html#installing-source-from-github-for-development. But first, these steps

For this I’m going to use the example of adding lambda1/lambda2 as I wanted to see the effect of adding in tidal effects. Any mention of L1/L2 should be replaced with the parameter YOU want to add.
1) Tell pycbc-glue that we want the XML files to contain parameters called lambda1 and lambda2. Note here that the name of the parameters should match the exact names that PyCBC parses through already. In this example I double checked the names from https://github.com/ligo-cbc/pycbc/blob/master/pycbc/waveform/waveform.py#L97 . If you know the name of the parameter(s) you want to add, go into your pycbc-glue directory and find the file lsctables.py (/glue/ligow/lsctables.py). Add the parameters you want to the SimInspiralTable class. An example of this is here https://github.com/torreycullen/pycbc-glue/commit/fa2fcef12e7edf388d33ca3c3fb0849c2a7f16c1 (in this case I added tidal_order, as well as lambda1/lambda2). Take note also that the data type of each parameter is important. “real_4” is a float, “int_4s” is an integer, etc.

2) Next we need to write a code that takes in a XML file *not* containing the lambda parameters (which lalapps_inspinj would produce), and add them with whatever value is appropriate. Here's an example of a code that does that https://github.com/torreycullen/pycbc/blob/master/bin/pycbc_add_to_siminspiral_table (credit Ian Harry). In this I create a polynomial fit for what I know the lambda values to be and add them in this for loop https://github.com/torreycullen/pycbc/blob/master/bin/pycbc_add_to_siminspiral_table#L87. 

3) Check if your python script works. Run the command:
lalapps_inspinj \
--m-distr componentMass \
--min-mass1 1 \
--max-mass1 3 \
--min-mass2 1 \
--max-mass2 3 \
--min-mtotal 2 \
--max-mtotal 6 \
--gps-start-time 1126051217 \
--gps-end-time 1127271617 \
--enable-spin \
--min-spin1 0 \
--max-spin1 0.4 \
--min-spin2 0 \
--max-spin2 0.4 \
--dchirp-distr uniform \
--min-distance 10 \
--max-distance 1000 \
--waveform SpinTaylorT2threePointFivePN \
--f-lower 15 \
--i-distr uniform \
--l-distr random \
--t-distr uniform \
--time-step 100 \
--time-interval 1 \
--taper-injection startend \
--seed 1231432 \
--output OUTPUT_FILE.xml
to create an example xml file. Use this as the -i option for the script you just created. For example,
python pycbc_add_to_siminspiral_table.py -i OUTPUT_FILE.xml -o new_output_file.xml 
Open the new_output_file.xml and make sure you see your parameters have been added to the columns of existing parameters, for example: 
                <Column Type="real_4" Name="sim_inspiral:lambda1"/>
                <Column Type="real_4" Name="sim_inspiral:lambda2"/>
                <Column Type="int_4s" Name="sim_inspiral:tidal_order"/>
and that the values you specified in the script are there. If this is working lets go to step 4.

4) To give a very short overview of what needs doing. Here is the codethat is used to create the "banksim" workflow:

 https://github.com/ligo-cbc/pycbc/blob/master/bin/workflows/pycbc_create_bank_verifier_workflow

 Each executable that it runs is declared as a class at the top e.g.:

 https://github.com/ligo-cbc/pycbc/blob/master/bin/workflows/pycbc_create_bank_verifier_workflow#L46

 A new entry would need to be added here for the new code.

 Then the function here:

 https://github.com/ligo-cbc/pycbc/blob/master/bin/workflows/pycbc_create_bank_verifier_workflow#L189

 Is responsible for running a specific set of injections. After this
 line the injection file is handed off to the actual banksim code:

 https://github.com/ligo-cbc/pycbc/blob/master/bin/workflows/pycbc_create_bank_verifier_workflow#L196

 so between lines 196 and 197 we would need to add a call to the new executable and then replace inj_file with the new inj_file which would be made by adding the new executable in place. 
 HOWEVER, I’ve created a patch to implement the implied changes so you shouldn’t need to do much. Before you do anything though make sure your python scrip from step 2 is called add_to_siminspiral_table.py. Download the patch here  https://github.com/torreycullen/pycbc/blob/master/bin/0001-Changes-to-run-add_to_siminspiral_table-in-banksim.patch. A link to applying a patch in git is here https://www.devroom.io/2009/10/26/how-to-create-and-apply-a-patch-with-git/. Download and apply the patch, if you want to see the actual changes that will be made they are here https://github.com/torreycullen/pycbc/commit/5b5d3162955ee96e977828992b368910e101b85e. 

5) After this we just need to install your forked version of pycbc and pycbc-glue. Using this link http://ligo-cbc.github.io/pycbc/latest/html/install.html#installing-source-from-github-for-development (the main command being pip install -e git+git@github.com:your-username-here/pycbc.git#egg=pycbc changing the path where necessary). Next install your edited version of pycbc-glue by following these instructions https://ligo-cbc.github.io/pycbc/latest/html/install.html#modifying-pycbc-glue. 

And that’s it! 


Other notes:
One problem I found during this process that you may encounter depending what cluster you are working on, is that while installing pycbc it updates scipy to version 0.19.0. Some processes for this framework to work correctly require a version not as up to date as this (something about moving spicy.weave into a different library). A version that should be working correctly is 0.12.1. To download this version of scipy run the following commands:

pip uninstall scipy
pip install scipy==0.12.1
