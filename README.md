# documentation
This is an outline on the process of adding a new parameter to the input waveforms when running a banksims as well as creating the workflow and submitting the job. For this I’m going to assume you have a working version of lawsuit already set up, the instructions for which can be found at

http://ligo-cbc.github.io/pycbc/latest/html/install.html#installing-lalsuite-into-a-virtual-environment.  
Be sure you have a working version of lalsuite installed in a virtual environment before continuing. All instructions assume you are in this environment.

First, you want to follow instructions here https://help.github.com/articles/fork-a-repo/ to "fork" both the pycbc and pycbc-glue repositories into your git-hub account. We will be making edits to these repositories and eventually checkout these versions of pycbc/pycbc-glue with these instructions http://ligo-cbc.github.io/pycbc/latest/html/install.html#installing-source-from-github-for-development. But first, these steps

For this I’m going to use the example of adding lambda1/lambda2 as I wanted to see the effect of adding in tidal effects. Any mention of L1/L2 should be replaced with the parameter YOU want to add.
