BCBIO Workflow
==============

Derived from the GA4GH DREAM Challenge workflow: `https://www.synapse.org/#!Synapse:syn9725771` originally submitted by [Brad Chapman](https://github.com/chapmanb).

These are directions for running bcbio in toil.

**Workflow was run with a fresh AWS EC2 (Ubuntu 16.04 Server) instance of type t2.2xlarge (8 CPU; 32 Gb RAM) with one node, mounted with a 200 Gb EFS volume (EC2 wizard in firefox).**

Update the instance::

    sudo apt-get update
    sudo apt-get -y upgrade
    sudo apt-get -y dist-upgrade

Install docker::

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update
    apt-cache policy docker-ce
    sudo apt-get install -y docker-ce

Modify docker so that sudo is not needed for root::

    sudo usermod -aG docker ${USER}
    sudo su - ${USER}

Install pip, a virtualenv, the dev version of toil, and the requisite pip installs::

    sudo apt install -y python-pip
    sudo pip install --upgrade pip
    sudo pip install virtualenv
    git clone https://github.com/BD2KGenomics/toil.git
    cd toil
    virtualenv --python /usr/bin/python2.7 dev
    source /home/ubuntu/toil/dev/bin/activate
    make prepare
    make develop extras=[aws,mesos,azure,google,encryption,cwl]
    pip install cwl-runner schema-salad==2.6.20170630075932 
    pip install avro==1.8.1 cwltool==1.0.20170822192924 ruamel.yaml==0.14.12 --no-cache-dir
    pip install synapseclient --no-cache-dir
    pip install html5lib cwltest
    cd ..

I created a copy of my Synapse credentials in .synapseConfig (synapse hosts the data files needed to run the workflow)::

    nano .synapseConfig

After pasting a username and password into `.synapseConfig`::

    [authentication]
    username: email@ucsc.edu
    password: password

Download the data from Synapse::

    synapse get -r syn9725771

Run the main bcbio workflow::

    cd NA12878-platinum-chr20-workflow
    toil-cwl-runner NA12878-platinum-chr20-workflow/main-NA12878-platinum-chr20.cwl NA12878-platinum-chr20-workflow/main-NA12878-platinum-chr20-samples.json
