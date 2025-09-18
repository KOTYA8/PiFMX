# PiFmX
Original repository: [PiFmRds](github.com/ChristopheJacquet/PiFmRds)  
PiFmX - will support (hope) full RDS functions.  
This is an extended version of PiFmRds, which operates on the latest **Raspberry Pi OS** system and on the **Raspberry Pi 4B** board.  

# Installation 
For continuous operation of the FM transmitter on the Raspberry Pi 4B, the following command is entered:  
```bash
echo "performance"| sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

Installation PiFmX:  
```bash
git clone https://github.com/KOTYA8/PiFmX.git
cd PiFmX/src
make clean
make
```

PIFMX launch:  
```bash
sudo ./pi_fm_rds
```

# General Arguments

```bash
pi_fm_rds [-freq freq] [-audio file] [-ppm ppm_error] [-pi pi_code] [-ps ps_text] [-rt rt_text] [-ecc XX]
```
