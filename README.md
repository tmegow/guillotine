# Guillotine - using Raspberry Pi 3, motion, ffmpeg, sshfs, and autofs to capture motion video events to an offsite server.
## Prerequisites
## - Raspberry Pi 3
## - Raspberry Pi Camera Module
- Download Raspbian Jessie Lite and install it on an SD card (e.g. OS X example commands shown below)
  - diskutil unmountDisk /dev/disk2
  - sudo dd bs=1 if=raspian_image.dmg -of /dev/disk2
- Boot raspbian and append this to the bottom of /boot/config.txt
  - gpu_mem=880
  - start_x=1
  - disable_camera_led=1
- Edit /etc/defaults/keyboard
  - XKBLAYOUT="us"
- (reboot to apply above changes)
- Connect pi to wifi
  - iwlist wlan0 scan
  - wpa_passphrase "your_ssid" "your_passphrase" > wifi_config
  - cat wifi_config >> /etc/wpa_supplicant/wpa_supplicant.conf
  - wpa_cli reconfigure
  - #Check if wlan0 connects and acquires an address via DHCP
- Update and install git
  - apt-get update && apt-get upgrade
  - apt-get install git
- Install ffmpeg
  - Follow the instructions specified here -> https://www.jeffreythompson.org/blog/2014/11/13/installing-ffmpeg-for-raspberry-pi/ (summarized below)
    - Install H264 Support
      - cd /usr/src
      - sudo git clone git://git.videolan.org/x264
      - cd x264
      - sudo ./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
      - sudo make
      - sudo make install
    - Compile and install ffmpeg
      - cd /usr/src
      - sudo git clone https://github.com/FFmpeg/FFmpeg.git
      - cd ffmpeg
      - sudo ./configure --arch=armel --target-os=linux --enable-gpl --enable-libx264 --enable-nonfree
      - sudo make
      - sudo make install
- Install motion			
  - wget https://github.com/Motion-Project/motion/releases/download/release-4.0.1/pi_jessie_motion_4.0.1-1_armhf.deb
  - sudo dpkg -i pi_jessie_motion_4.0.1-1_armhf.deb
  - sudo apt-get -f intall #install missing dependencies for motion
- Configure Motion (optimized motion config for raspberry pi)
  - #this patch file sets the motion target_dir to /opt/Video/ir
  - sudo patch < motion.patch
- Copy ssh config file and private ssh key to /root/.ssh
  - Connect to the video server once to add the ssh public key (id_rsa.pub) to /home/your_user/.ssh/known_hosts
- Enable sshfs & autofs
  - sudo apt-get install sshfs autofs
  - sudo mkdir -p /opt/Video/noir
  - #copy auto.master & auto.sshfs to /etc (ensure neither are executable (chmod 0600).
  - #The auto.sshfs file uses this bit at the end of the line to configure the remote sshfs mount: "sshfs\#User@your_server_fqdn\:/home/User/Video/evidence/ir"
  - #Be sure to create the remote dir on the target server, owned by the sshfs user, with appropriate permissions (I used mode 777)
  - sudo systemctl restart autofs 
  - #check mount point is working as intended (can you see the contents of the sshfs mount in /opt/Video ?)
- Enable motion auto-start systemd service
  - #copy start_motion.service to /etc/systemd/system
  - sudo systemctl enable start_motion.service
  - (start raspivid -p -f once if you get this error (mmal_vc_component_enable failed to enable component ENOSPC)
	
Motion now should be capturing motion video events to your offsite server 


Optional Steps:
- Setup SSH Server:
  - sudo raspi-config
    - select "Interfacing Options"
    - Navigate to SSH and enable
    - (headless activation)
    - sudo touch /boot/ssh
- Review timezone config
    - sudo dpkg-reconfigure tzdata
- Adjust mmalcam_control_params options such as -hf and -vf if the picam picture needs tweaking
- Adjust motion framerate and threshold config values (to increase/decrease frames captured per second, and the motion threshold value [the motion config file can be found at /etc/motion/motion.conf])	
