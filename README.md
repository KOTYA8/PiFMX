# PiFmX
PiFmX - will support (hope) full RDS functions.  
This is an extended version of PiFmRds, which operates on the latest **Raspberry Pi OS** system and on the **Raspberry Pi 4B** board.  
Original repository: [PiFmRds](github.com/ChristopheJacquet/PiFmRds) 

# Supported functions RDS (global and rds_ctl)
* **PI** - Programme Identification (4 characters: XXXX)  
* **PTY** - Programme Type (0 - 31)
* **PS** - Programme Service Name (8 characters: XXXXXXXX)  
* **RT** - Radio Text (64 characters)
* **RT(A/B)** - Radio Text (A/B Switches). Modes: only A, only B, A/B.  
* **RT+** - Radio Text+. (Tags: 0 - 63). Example (tags.first symbol.last symbol): XX.XX.XX|XX.XX.XX
* **TP** - Traffic Programme identification (ON/OFF)   
* **TA** - Traffic Announcement identification (ON/OFF) 
* **AF(A)** - Alternative Frequencies List (A method). Example: 87.6 87.8 91.1  
* **AF(B)** - Alternative Frequencies List (B method). Example (main|same|regional): 87.6|90.1 95.5|90.5 90.6  
* **M/S** - Music Speech switch (M/S)  
* **ECC** - Extended Country Code (2 characters: XX)  
* **LIC** - Language Identification Code (2 characters: XX)  
* **PIN** - Programme Item Number. (Date: 01-31, Hours: 00-23, Minutes: 00-59). Example (date|hours|minutes): XX|XX|XX  
* **PTYN** - Programme Type Name. (8 characters: XXXXXXXX)  
* **Long PS** - Long Programme Service Name (32 characters)  
* **DI(A,C,D)** - Decoder Identification (Artifical Head, Compressed, Dynamic PTY). Example: A, AC, AD, C, CA, CD, D, ACD  
* **EON** - Enhanced Other Networks Information. (PI,PS,AF,LI,PTY,TP,TA,PIN). Example: D392|WDR 2   |102.1 88.5|0000|10|ON|OFF|022254|    

# Installation 
For continuous operation of the FM transmitter on the Raspberry Pi 4B, the following command is entered:  
```bash
echo "performance"| sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

Module installation (before installing PiFmX):
```
sudo apt install libsndfile1-dev
```

Installation PiFmX:  
```bash
git clone https://github.com/KOTYA8/PiFmX.git
cd PiFmX/src
make clean
make
```

PiFmX launch:  
```
sudo ./pi_fm_rds
```

# General Arguments
By default the PS changes back and forth between `RPi-Live` and a sequence number, starting at `00000000`. The PS changes around one time per second.  
```bash
sudo ./pi_fm_rds [-freq freq] [-audio file] [-ppm ppm_error] [-pi pi_code] [-ps ps_text] [-rt rt_text] [-ecc XX]
```
All arguments are optional:  

* `-freq` specifies the carrier frequency (76 - 108 MHz). Example: `-freq 107.9`.  
* `-audio` specifies an audio file to play as audio. The sample rate does not matter: Pi-FM-RDS will resample and filter it. If a stereo file is provided, Pi-FM-RDS will produce an FM-Stereo signal. Example: `-audio sound.wav`. The supported formats depend on libsndfile. This includes WAV and Ogg/Vorbis (among others) but not MP3. Specify - as the file name to read audio data on standard input (useful for piping audio into Pi-FM-RDS, see below).  
* `-pi` specifies the PI-code of the RDS broadcast. 4 hexadecimal digits. Example: `-pi FFFF`.  
* `-ps` specifies the station name (Program Service name, PS) of the RDS broadcast. Limit: 8 characters. Example: `-ps RASP-PI`.  
* `-rt` specifies the radiotext (RT) to be transmitted. Limit: 64 characters. Example: `-rt 'Hello, world!'`.  
* `-ctl` specifies a named pipe (FIFO) to use as a control channel to change PS and RT at run-time (see below).  
* `-ppm` specifies your Raspberry Pi's oscillator error in parts per million (ppm), see below.  
* `-ecc` specifies the country for the transmitter (Extended Country Code, ECC). Displayed through 2 characters, example: `-ecc E0`.

### Clock calibration (only if experiencing difficulties)

The RDS standards states that the error for the 57 kHz subcarrier must be less than ± 6 Hz, i.e. less than 105 ppm (parts per million). The Raspberry Pi's oscillator error may be above this figure. That is where the `-ppm` parameter comes into play: you specify your Pi's error and Pi-FM-RDS adjusts the clock dividers accordingly.

In practice, I found that Pi-FM-RDS works okay even without using the `-ppm` parameter. I suppose the receivers are more tolerant than stated in the RDS spec.

One way to measure the ppm error is to play the `pulses.wav` file: it will play a pulse for precisely 1 second, then play a 1-second silence, and so on. Record the audio output from a radio with a good audio card. Say you sample at 44.1 kHz. Measure 10 intervals. Using [Audacity](https://www.audacityteam.org/) for example determine the number of samples of these 10 intervals: in the absence of clock error, it should be 441,000 samples. With my Pi, I found 441,132 samples. Therefore, my ppm error is (441132-441000)/441000 * 1e6 = 299 ppm, **assuming that my sampling device (audio card) has no clock error...**

### Piping audio into Pi-FM-RDS

If you use the argument `-audio -`, Pi-FM-RDS reads audio data on standard input. This allows you to pipe the output of a program into Pi-FM-RDS. For instance, this can be used to read MP3 files using Sox:

```
sox -t mp3 http://www.linuxvoice.com/episodes/lv_s02e01.mp3 -t wav -  | sudo ./pi_fm_rds -audio -
```

Or to pipe the AUX input of a sound card into Pi-FM-RDS:

```
sudo arecord -fS16_LE -r 44100 -Dplughw:1,0 -c 2 -  | sudo ./pi_fm_rds -audio -
```

### Non-ASCII characters

You can use the full range of characters supported by the RDS protocol. Pi-FM-RDS decodes
the input strings based on the system's locale variables. As of early 2024, Raspberry Pi
OS uses by default UTF-8 and the `LANG` variable is set to `en_GB.UTF-8`. With this setup,
it should work out of the box.

If it does not work, look at the first message that Pi-FM-RDS prints out. It should be
something sensible, like:

```
Locale set to en_GB.UTF-8.
```

If it is not consistent with your setup, or if the locale appears to be set to `(null)`,
then your locale variables are not set correctly and Pi-FM-RDS is incapable of working
with non-ASCII characters.

### Control RDS (rds_ctl)

You can control RDS at run-time using a named pipe (FIFO). For this run Pi-FM-RDS with the -ctl argument.
```bash
mkfifo rds_ctl
sudo ./pi_fm_rds -ctl rds_ctl
```

At this point, Pi-FM-RDS waits until another program opens the named pipe in write mode (for example cat >rds_ctl in the example below) before it starts transmitting.

You can use the named pipe to send “commands” to change PS, RT and TA. For instance, in another terminal:
```
cat >rds_ctl
PS MyText
RT A text to be sent as radiotext
TA ON
PS OtherTxt
TA OFF
```

### PS and RT modes (rds_ctl)
I also have a special script that allows you to use different PS and RT modes:
[PiFmPSRT](https://github.com/KOTYA8/PiFmPSRT)

# Changelog

* 18.09.2025 - Support **ECC**
