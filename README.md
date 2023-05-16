# Bullet

Bullet is a heavily modified Ender3. All mods and software is tracked on this repository.

Bullet runs Klipper now! It also has two different control boards - the main control board, which is an SKR Mini E3 V1.2 and a toolhead control board connected over CAN, which is a BIQU EBB42. Here are the compiling + deployment instructions moving forward.

## Main Control Board - SKR Mini E3 V1.2

### Building and Deploying Manually

Run the following commands on the raspberry pi
``` bash
# Build the klipper firmware for the main control board
cd ~/klipper
make clean
make KCONFIG_CONFIG=$HOME/Bullet/control-board/BTT_SKR_Mini_E3_v3/klipper-make.cfg

# Stop the klipper service to upload to the control board
sudo service klipper stop

# Run the flashing script over the SD card
./scripts/flash-sd-card.sh /dev/serial/by-id/usb-Klipper_stm32g0b1xx_4F0047001250415833323520-if00 btt-skr-mini-e3-v3

# Restart the klipper service
sudo service klipper start
```

## Toolhead Board - Biqu EBB42 with MAX31856

### Initialization Steps
1. Set up CAN on Raspberry Pi. See https://maz0r.github.io/klipper_canbus/controller/rs485.html

2. Install CANBoot to allow for installing klipper over CAN

``` bash
# Clone CanBoot
cd ~
git clone https://github.com/Arksine/CanBoot

# Build CanBoot
cd CanBoot
make clean
make KCONFIG_CONFIG=$HOME/Bullet/control-board/BIQU_EBB42_v1_2/canboot-make.config

# Upload CanBoot
# Hold reset and boot buttons while plugged in over USB
# Verify the device is in bootloader mode with:
lsusb
# Flash the CanBoot firmware, make sure you replace the address at the end with what showed up in lsusb
sudo dfu-util -a 0 -D ~/CanBoot/out/canboot.bin --dfuse-address 0x08000000:force:mass-erase:leave -d 0483:df11
# For more information, see source below
```

Source: https://maz0r.github.io/klipper_canbus/toolhead/ebb36-42_v1.1.html

3. Flash Klipper over CAN

``` bash
# Make sure you see the device on the CAN network
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

# Build the Klipper firmware for the toolhead board
cd ~/klipper
make clean
make KCONFIG_CONFIG=$HOME/Bullet/control-board/BIQU_EBB42_v1_2/klipper-make.config

# Flash the Klipper firmware on the toolhead board over CAN
# Replace the last parameter with the UUID from the canbus_query command above
python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u 4c6771e6c940
```

Source: https://maz0r.github.io/klipper_canbus/toolhead/ebb36-42_v1.1.html

## Raspberry Pi 3B+
Making sure the Raspberry Pi won't overheat or undervolt due to too little power is super important. Use the following commands to run a worst-case scenaro CPU stress test, each for 10 minutes:

```bash
while true; do vcgencmd measure_clock arm; vcgencmd measure_temp; sleep 10; done& stress -c 4 -t 900s
while true; do vcgencmd measure_clock arm; vcgencmd measure_temp; sleep 10; done& ./cpuburn-a53
```
Source: https://core-electronics.com.au/guides/stress-testing-your-raspberry-pi/#:~:text=Stress%20testing%20is%20an%20important,and%20stability%20of%20the%20system.

## Resources

* Connecting RPI to control board over UART: https://www.youtube.com/watch?v=AtW3GqkKUz8
* CANBUS board + RPI Hat instructions: https://www.youtube.com/watch?v=jgE3XMM9PBk
* Updating klipper on MCU automatically with a script: https://www.youtube.com/watch?v=uNW8phiV83w
* Voron Design documentation for skr mini e3 v1.2: https://docs.vorondesign.com/build/software/miniE3_v12_klipper.html
* Debugging CANBus communication issues: https://www.teamfdm.com/forums/topic/1524-debugging-canbus-and-communication-timeout-while-homingbytes_invalid/
* Sweet KlackEnder probe macros: https://github.com/Harrypulvirenti/Klack-Probe-Macros/tree/main

### Board Documentation

* [BigTreeTech SKR E3 Mini V1.2 documentation](https://github.com/bigtreetech/BIGTREETECH-SKR-mini-E3), also found in `control-board/BTT_SKR_MINI_E3_V1_2`
* RS485 CANbus Raspberry Pi Hat: https://www.waveshare.com/rs485-can-hat.htm
