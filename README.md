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
By default the PS changes back and forth between Pi-FmRds and a sequence number, starting at 00000000. The PS changes around one time per second.  
```bash
sudo ./pi_fm_rds [-freq freq] [-audio file] [-ppm ppm_error] [-pi pi_code] [-ps ps_text] [-rt rt_text] [-ecc XX]
```
All arguments are optional:  

* `-freq` specifies the carrier frequency (in MHz). Example: `-freq 107.9`.  
* `-audio` specifies an audio file to play as audio. The sample rate does not matter: Pi-FM-RDS will resample and filter it. If a stereo file is provided, Pi-FM-RDS will produce an FM-Stereo signal. Example: `-audio sound.wav`. The supported formats depend on libsndfile. This includes WAV and Ogg/Vorbis (among others) but not MP3. Specify - as the file name to read audio data on standard input (useful for piping audio into Pi-FM-RDS, see below).  
* `-pi` specifies the PI-code of the RDS broadcast. 4 hexadecimal digits. Example: `-pi FFFF`.  
* `-ps` specifies the station name (Program Service name, PS) of the RDS broadcast. Limit: 8 characters. Example: `-ps RASP-PI`.  
* `-rt` specifies the radiotext (RT) to be transmitted. Limit: 64 characters. Example: `-rt 'Hello, world!'`.  
* `-ctl` specifies a named pipe (FIFO) to use as a control channel to change PS and RT at run-time (see below).  
* `-ppm` specifies your Raspberry Pi's oscillator error in parts per million (ppm), see below.  
* `-ecc` specifies the country for the transmitter (Extended Country Code, ECC). Displayed through 2 characters, example: `-ecc E0`.  
