# liang-group-fss-geant4
Geant4 Code Installation for FSS Analysis / Detector Response Calibration

## Remote Geant-4 Installation
0. Access the expanse cluster using your ACCESS ID.
```
ssh <ACCESS_ID>@login.expanse.sdsc.edu
```

1. Clone this repository into the home directory at your login node on Expanse.
```
cd $HOME
git clone https://github.com/masonweiss/liang-group-fss-geant4.git
```

2. Download and unpack Geant4 to a subdirectory in the home directory named 'geant4' via:
```
mkdir geant4
cd geant4
wget https://gitlab.cern.ch/geant4/geant4/-/archive/v11.2.2/geant4-v11.2.2.tar.gz
tar -xzvf geant4-v11.2.2.tar.gz
```
Note: When I installed Geant4 during October 2024, the most recent version was 11.2.2. There have since (as of February 2025) been a few minor updates. I can't promise that the compilers on Expanse will work 100% from 

3. Load the Expanse modules from spider for C/C++ compilation. 
```
module load gcc/10.2.0
module load cmake/3.21.4
```

4. Verify the compressed source code has been unpacked and make an installation directory
```
rm geant4-v11.2.2.tar.gz
mkdir geant4-v11.2.2-install
```

There should now only be two objects in the geant4 folder, both directories: geant4-v11.2.2-install and geant4-v11.2.2

5. Prepare build files for Geant4 using the following commands:
Note: this method installs all of the default Geant4 data. The download is completed in subsequent steps, which may take quite a long time! If you're concerned about disconnecting, before running the following code, you can type ```screen``` to start a screen session, run any shell command as if it were a normal terminal, and then detach the screen using Ctrl-A then Ctrl-D. To resume the screen session (assuming you haven't made more than one), you can either type ```screen -r``` from the terminal, or press Ctrl-R. The code will run in a detached screen even if your VM connection fails. Note that you can't view the full output to console from a screen session, though, since it won't let you scroll up.
```
cd geant4-v11.2.2
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/geant4/geant4-v11.2.2-install \
-DGEANT4_BUILD_MULTITHREADED=ON \
-DGEANT4_INSTALL_DATA=ON \
-DGEANT4_BUILD_CXXSTD=17
```


6. If everything goes correctly, the following commands should finish the installation without any fatal errors (some warnings may exist, though)
```
make 
make install
```

7. After installing the data, you may need to specify the path for Geant4 to recognize it. You may also need to specifically add the executable to the path.
```
export G4DATA=$HOME/geant4/geant4-v11.2.2-install/share/Geant4-11.2.2/data
source $HOME/geant4/geant4-v11.2.2-install/bin/geant4.sh
```

8. You can then test your installation by running an example simulation studying the cross-section of hadronic interactions. First, you'll need to locate the directory for the test and build the executable. These tests are contained within the geant4 source code, so first navigate to your ```geant4-v11.2.2``` directory. Then, go into the Hadr03 source code within the extended examples directory.
```
cd $HOME/geant4/geant4-v11.2.2
cd examples/extended/hadronic/Hadr03
```
Within this directory is an example Geant4 project. If you're familiar with C++, you'll recognize that it contains two important directories: ```src``` and ```include```. The ```src``` directory includes the source code for the project with all of the revelant files, such as the ```ActionInitialization```, ```DetectorConstruction``` and ```PrimaryGeneratorAction``` files. Others, such as Run, Event, Stepping, and Tracking Action are present depending on what your simulation needs. Any ```.cc``` files in the ```src``` directory will also need a corresponding ```.hh``` header file in the ```include``` directory. In the main ```Hadr03``` directory, you will also see an ```Hadr03.cc``` file. This is the main entry point for the simulation, and specifies some general parameters, such as what visualization to use. You'll also see some macro files, ending in ```.mac```. These specify external parameters, such as what particles to shoot, what their energy is, etc. 

Note: If you're curious about the other pre-defined macro files, you can look at the README file for descriptions of the other simulations. Some are quite interesting, but just make sure they don't try to initialize some visualization package.

9. To run the simulation, you'll need to build the executable. Make a directory in the main simulation folder ```Hadr03``` named ```build```. Compile the program and build the executable from within this folder. You will also run the executable from within this folder. Before doing this, make sure you've loaded in the compilers from Step 3. Each time you log into Expanse, you'll need to reload them.
```
mkdir build
cd build
cmake ..
make
```

10. Generally, when you run executables on Expanse, you'll need to submit a job to the cluster. For complex jobs, such as those for job arrays or needing particular configuration files, you'll use the web portal, make a ```.sb``` job submission script file including the full paths to each file, and then submit the job from there. For the test files, you can away with just calling ```srun``` to submit the job with its arguments all in one (relatively long) command. The general construction of this command is: ```srun --<options> <executable_path> <macro_path>```. Note that you just provide the path to the exectuable, not the ```./``` command. The options can include how many nodes to request, what partition (compute, shared, or debug) to submit to, how many threads per node, how much time to permit the job to run, and importantly, what project code to charge. Our project code is ```riu122```. Here, we'll specify the ```nFission``` macro, which will launch 10,000 1eV neutrons at a 10-meter thick block of U235, basically guaranteeing all neutrons will induce fission. The text output to the terminal will include a calculation of the cross-section and mean free path, so make sure you don't run this from a ```screen``` command, as suggested from section 5. Try running the simulation with the following command, which will create a job and return a lot of text to your terminal.

```
srun --partition=debug  --pty --account=riu122 --nodes=1 --ntasks-per-node=16 --mem=32G -t 00:05:00 --wait=0 --export=ALL Hadr03 nFission.mac
```

Important: If you don't specify a macro file, Geant4 will usually open interactive mode. The default is usually to try to launch a GUI tool, which will not work on Expanse. If you install Geant4 locally (on your computer), you can configure the visualization to show your particle tracks and detector. This is useful to make sure your detector construction makes sense, but if you're running many particles or events, it is not terribly useful and makes the simulation very slow. So, when you run on Expanse, you should provide the path to the macro file as the first argument when you run the executable. On your local machine, this would look like ``` ./executable <macro_path>```. However, on Expanse, you will usually not be able to run executables from the login node, so this syntax is NOT how you would submit a job.

11. Scroll up to review the output from the simulation. You should find a block of text like the following:

```
Run terminated.
Run Summary
  Number of events processed : 10000
  User=2.940000s Real=13.114699s Sys=0.080000s [Cpu=23.0%]

 The run is 10000 neutron of 1 eV  through 10 m   of U235 (density: 19.05 g/cm3 )

 Process calls frequency:
	nFission= 10000


 MeanFreePath:	2.9762 mm  +- 2.9715 mm 	massic: 5.6696 g/cm2 
 CrossSection:	3.36 cm^-1 		massic: 17.638 mm2/g 
 crossSection per atom:	68.841 barn  

 Verification: crossSections from G4HadronicProcessStore

            nFission = 17.937 mm2/g 	70.006 barn  
               total = 17.937 mm2/g 	70.006 barn  

 List of nuclear reactions: 


 List of generated particles:

   Momentum balance: Pmean =  81.017 MeV	( 286.26 keV --> 350.91 MeV) 

... write file : nFission.root - done
... close file : nFission.root - done
```

12. The created nFission.root file uses Geant4's data storage/analysis software, ROOT. You can send this file back to your local machine and analyze the histograms using either the C++ or Python versions of ROOT.
