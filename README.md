# CMSPhase2GCT

This project uses APxL1TAlgoDev from Ales Svetek (https://github.com/APxL1TAlgoDev)


STEP-1: Vivado HLS
SSH into a computer at UW Madison or uwlogin machine:
```bash

mkdir -p /scratch/$USER/CMSPhase2HLS (/data/$USER/CMSPhase2HLS)
cd /scratch/$USER/CMSPhase2HLS (/data/$USER/CMSPhase2HLS)

source /opt/Xilinx/Vivado/2018.2/settings64.sh (/data/tools/Xilinx/Vivado/2018.2/settings64.sh)
(Only for uwlogin: source /data/setup_xilinx_lic.sh)

git lfs install
git version
git-lfs version
git clone --recursive git@github.com:APxL1TAlgoDev/APx_Gen0_Algo.git (Cloning Algo project)
git clone --recursive git@github.com:APxL1TAlgoDev/APx_Gen0_Infra.git (Cloning Infra project)
cd APx_Gen0_Algo
cd VivadoHls/null_algo_unpacked/vivado_hls (Keep all your Vivado HLS projects in this file directory)
mkdir /scratch/$USER/CMSPhase2HLS/APx_Gen0_Algo/build
On how to use one of the existing projects and incorporate your source files into it, see note later.
vivado_hls -f run_hls.tcl (Read vivado_hls command note to understand this better)
```

STEP-2: Using Infra project to generate bitfile
```
change directory to Infra: cd /scratch/$USER/CMSPhase2HLS/APx_Gen0_Infra (/data/$USER/CMSPhase2HLS/APx_Gen0_Infra)
mkdir build (only first time)
cd firmware
Edit ruckus.tcl file and provide correct path to your HLS ruckus.tcl file. 
(If you are following the format of this README, this path should be /scratch/$USER/CMSPhase2HLS/APx_Gen0_Algo/VivadoHls/hls_project_name)
make clean 
make (nohub make &> make.log & - to make it run in background)
```

STEP-3: Bitfile on Gen0
```
Login to ayinger (UW Madison CPU)
Copy the bitfile onto a Gen-0 CTP7 to tmp folder: scp <bitfile> uwguest@eagle<card_ID>:/tmp 
Layer 1: eagle 65 card_ID=0, eagle 54 card_ID=1 and Layer 2: eagle50 card_ID=2
For permission related query, contact Aleš.
git clone https://github.com/APxL1TAlgoDev/APx_Gen0_SW
cd APx_Gen0_SW/pc_client
ln -sf CTP7Map_UW.xml CTP7Map.xml
make
source env.sh
mkdir data
./card_config <card_ID>  <bitfile_name>
please note to drop .bit extension when passing <bitfile_name> to the application

./layer1_config <card_ID>

Copy your input test vector and output reference test vector files to your local data dir
Inject input test vectors: ./upload_inputs  <card_ID> inp.txt
Download outputs: ./download_outputs <card_ID> out.txt
Compare out.txt with out_ref.txt to verify.
```
Using one of demo HLS projects from Algo repository: 
```
For maintaining the file format, it is recommended to use one of the existing demo projects 
and modify it according to your source files.
Following changes need to be made:
1. cp -R one of the existing projects (null_algo/test_algo) and rename it 
2. Copy your source files into vivado_hls/src folder
3. Edit the sources.tcl file in vivado_hls folder and add correct source files
4. (Optional) Edit the input test vector file as required in the vivado_hls/data folder
5. Run the vivado_hls -f run_hls.tcl script
```
vivado_hls command:
```
Internally, the “run_hls.tcl” script uses 5 parameters that steer the build process:

synth: 1-> do C synthesis, 0-> skip this step
csim: 1-> run C simulation, 0-> skip skip this step
cosim: 1-> run C/RTL cosimulation, 0-> skip this step
export: 1-> export algo HDL (previously we were using *.dcp), 0-> skip this step
tv: specify test vector (defaults to test1)

By default if you pass no parameters to the build script, it runs with the following configuration:
vivado_hls -f run_hls.tcl synth=1 csim=1 cosim=1 export=1 tv=test1

If you want to execute only C simulation (useful for quickly iterating on many initial algo  changes), you can do so:
vivado_hls -f run_hls.tcl synth=0 csim=1 cosim=0 export=0 tv=test1

As another example, if you want to skip both simulations steps altogether and just build  the project, you can do that as well:
vivado_hls -f run_hls.tcl csim=0 cosim=0 (this will run synth and export the HLS IP while skipping both sim steps)
```
