
# ===== start of configuration phase ====
# 1. Clone CAPI-SNAP repository
#(from /home/opuser directory)
git clone https://github.com/open-power/snap.git

# 2. Clone CAPI PSLSE repository with proper release
#(from /home/opuser directory)
git clone -b v3.1 https://github.com/ibm-capi/pslse

# 3. Export PSLSE_ROOT (Simulation) and PSL_DCP variable (Board specific hardware)
cd snap
echo "export PSLSE_ROOT=/home/opuser/pslse" > snap_env.sh
echo "#Variable FPGACARD will be automatically set to either ADKU3 or N250S according to your choice in make snap_config step" >>  snap_env.sh


#Ignore following lines kept for memory
#echo "export PSL_DCP=/home/opuser/CAPI_PSL_Checkpoints/\${FPGACARD}_Checkpoint/b_route_design.dcp" >>  snap_env.sh
echo "export PSL_DCP=/home/opuser/CAPI_PSL_Checkpoints/ADKU3_Checkpoint/b_route_design.dcp" >>  snap_env.sh
or 
gedit snap_env.sh
export PSLSE_ROOT=/home/opuser/pslse
export PSL_DCP=/home/opuser/CAPI_PSL_Checkpoints/${FPGACARD}_Checkpoint/b_route_design.dcp
# ===== end of configuration phase ====


# Now start the GUI configurator (from snap directory)

make snap_config

# select ADKU3 board for this Techdays lab

# Execute SNAP Action on CPU
cd ~/snap/actions/hls_helloworld/sw
make
# Prepare the data
echo "HELLO WORLD. I love this new experience with SNAP" > /tmp/t1
# Run the code
SNAP_CONFIG=CPU ./snap_helloworld -i /tmp/t1 -o /tmp/t2
# Check result
cat /tmp/t1
cat /tmp/t2

# Compile the hardware action
cd ~/snap/actions/hls_helloworld/hw
make
# Build the simulation model
cd ~/snap
make model
# Run the simulation
cd hardware/sim
./run_sim
# ==>> New terminal window

# Run the following commands in new simulation terminal
# Setup SNAP Action in simulation
# DO NOT CHANGE DIRECTORY !!
snap_maint -vv
# Set input if not done already
echo "Hello World. I love this new experience with SNAP." > /tmp/t1
snap_helloworld -v -i /tmp/t1 -o /tmp/t2
# Check results
cat /tmp/t1
cat /tmp/t2
# Hardware simulation mode
SNAP_CONFIG=CPU ./snap_helloworld -i /tmp/t1 -o /tmp/t2
SNAP_CONFIG=FPGA ./snap_helloworld -i /tmp/t1 -o /tmp/t3
# Original text file :
cat /tmp/t1
# CPU modified text file :
cat /tmp/t2
# FPGA modified text file :
cat /tmp/t3
# Exit simulation
# Returning to original terminal
exit

# Build FPGA image
cd ~/snap
make image

# ===== start of Power8 instance phase ====
# Upload FPGA Image (.bin file!) to cloud
upload-image <image_file.bin>

# Start new cloud instance with accelerator attached (Use the same file name as
in the upload step)
test-image <image_file.bin>
# ===== end of Power8 instance phase ====


##############################################
# Deployment step
# Run SNAP Action on hardware
# open the created Power8 Machine from your dashboard Instance view