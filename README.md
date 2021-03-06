---
**Ocean Observatories Initiative Cyberinfrastructure** 
**Integrated Observatory Network (ION)** 

eoi-services - Classes and services for acquiring and publishing data from external sources

(C) UC Regents, 2010-2012

---

# Description
This project contains modules, utilities, and services for acquisition of data from external sources within the OOICI-ION infrastructure.  The repository is first and foremost a framework, designed to be extended with concrete implementations that facilitate data acquisition from specific data sources.  However, it also contains concrete External Data Handler implementations for the services listed below:

- DAP
- WaterOneFlow *(not yet, but very soon!)*

Additional concrete implementations may be added to this repository over time.  The repository can also be used as a library to allow implementation of concrete External Data Handlers by other projects.

**References**  
[EOI Development Page](https://confluence.oceanobservatories.org/display/CIDev/External+Observatory+Integration+Development)  
[EOI Architecture Page](https://confluence.oceanobservatories.org/display/syseng/CIAD+EOI+External+Observatory+Integration)


#Prerequisites

This assumes basic development environment setup (git, directory structure). Please follow the
"New Developers Tutorial" for basic steps.

Pyon: The main dependency of this repository is the pyon Capability Container. Follow the listed
steps to install the minimal needed dependencies to run pyon on a Mac. For more details and Linux
install instructions, check out the [pyon README](https://github.com/ooici/pyon/blob/master/README)



Install the following if not yet present:

- git 1.7.7: Download the Mac or Linux installer and run it

- OS Packages and package management:
For Mac, use homebrew
    > /usr/bin/ruby -e "$(curl -fsSL https://raw.github.com/gist/323731)"
- python 2.7
*see below for installation*

* couchdb 1.1.0 (optional if memory mockdb is used)
*see below for installation*

- rabbitmq 2.6.1 or later (recommended, but can use rabbitmq on amoeba)
    *Download generic Linux version and unpack into a suitable directory.
    **Note:** May need erl (Erlang) and some dependencies installed before*

- **Install** libevent, libyaml, zeromq, couchdb, python, and rabbitmq with Homebrew
    > brew install libevent libyaml zeromq couchdb python rabbitmq

    You can even reinstall git using brew to clean up your /usr/local directory
    Be sure to read the pyon README for platform specific guidance to installing
    dependent libraries and packages.
    Linux: Note that many installs have much older versions installed by default.
    You will need to upgrade couchdb to at least 1.1.0.

Python packages and environment management:

- pip
    > easy_install pip

- virtualenv and virtualenvwrapper modules for your python 2.7 installation
    > easy_install --upgrade virtualenv
    > easy_install --upgrade virtualenvwrapper
    Note: This may require Mac's XCode (use XCode 3.3 free version)

- Setup a virtualenv to run COI-services (use any name you like):
    > mkvirtualenv --no-site-packages --python=python2.7 eoi
    Note: Do not use the pyon virtualenv if you are a pyon developer

###Compiled Library Dependencies
**hdf5 library**

    brew install hdf5

**netcdf**

    brew install netcdf
 

###Other OOI-CI Project Dependencies

This project requires that both the pyon and coi-services projects are installed in the same directory as the pydap-handlers-ion project (typically the "Dev/code" directory if the OOI-CI development directory structure is used).  These projects can be obtained using the following commands:

    git clone git@github.com:ooici/coi-services.git
    git clone git@github.com:ooici/pyon.git

#Source

Obtain the eoi-agents project by running:  

    git clone git@github.com:ooici-eoi/eoi-services.git
    cd eoi-services

#Buildout

Build the project using buildout (*don't miss the **-d** flag on the first command*):

    python bootstrap.py -d
    bin/buildout

#Generate Interfaces and Submodules

Before unit tests will pass, the following commands must be run (*See below for further information about submodules*):

    git submodule update
    bin/generate-interfaces

#Unit and Integration Tests
The unit and integration tests for this project can be run as follows.  Note that the *-v* flag can be used to get verbose output (prints the name and result of each test that is run).  The *--with-coverage* flag can also be added to view a test coverage report.

##Running the Unit Tests

The unit tests for this project only require that a rabbitmq broker be running:

    bin/nosetests -a UNIT,group=eoi

##Running the Integration Tests

Currently, the integration tests are only "psuedo" integration tests - they do not rely on any other ION services at this point, but they do reach out to external data servers for data:

    bin/nosetests -a INT,group=eoi


#Development

You can develop services locally in this repository. Use this repository until subsystem
specific repositories are available.

Please follow the following steps as long as you are new:

Get the latest code before you start editing, or anytime you want:

    git pull
    git submodule update  # Do NOT forget. This does not happen automatically

**See below for an automated approach to git-submodules.**

Once in a while, service interfaces change. Generate interfaces frequently (especially in case of error):

    bin/generate-interfaces

Before defining objects, services in ./obj, or defining app and deploy files in ./res, checkout the **master** branch:

    cd extern/ion-definitions
    git status              # Just to see what's going on
    git checkout master     # To track the master branch (enables update and later push)
    git pull origin master  # To get latest from the server

***Note:** The res/ and obj/ dirs are symlinks to a subdirectory in a git submodule. Beware of the pitfalls
of git submodule. You need to treat it as a separate GIT module. In case of changes, both GIT modules
must be pushed, submodule first:*

    cd extern/ion-definitions
    git status            # Just to see what's going on
    git commit -am "Something smart"
    git push origin master
    cd ../..              # To the root of coi-services
    git commit -am "Something smarter"
    git push

Put your services in ion/services/<subsystem>/... (subdirectories are allowed).


#Git Submodule Hooks

A git hook is a script that executes during various points of using git. Some simple hooks have been written
to help automate dealing with submodules for most people. See the steps here:
http://blog.chaitanyagupta.com/2009/08/couple-of-hooks-to-make-life-easy-with.html

They do require an initial setup. Simple instructions:

Clone this repository:

    cd /some/tmp/directory
    git clone https://github.com/chaitanyagupta/gitutils.git

Use the provided install script:

    sh gitutils/submodule-hooks/install.sh /path/to/your/pyon/or/coi-services/dir


The install script does the following (you can also do it manually):

    cd /your/pyon/.git/hooks
    cp /your/tmp/gitutils/submodule-hooks/pre-commit pre-commit
    cp /your/tmp/gitutils/submodule-hooks/post-merge-commit post-merge
    ln -s post-merge post-checkout
    chmod +x post-merge post-checkout pre-commit

Now, when checking out a branch, pulling, merging etc, git will prompt you to automatically update
if it notices a change to the commit that your supermodule points to.

The pre-commit script is so you don't forget to push changes to the submodule **BEFORE** you push changes
to the supermodule.