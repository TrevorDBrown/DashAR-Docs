---
id: 'install-ics-setup'
title: 'ICS Setup'
sidebar_position: 2
slug: '/install/ics-setup'
---

Once all prerequisites have been satisfied, you can set up the Intermediate Computing Subsystem (ICS). This document covers the process for setting up the Data Aggregator and Server (DAS) API, and associated components on a Raspberry Pi 5.

## Raspberry Pi Setup

1. Image the USB mass storage device with the Ubuntu Server 24.04 installation media.
    - Visit [Raspberry Pi's official Imager documentation](https://www.raspberrypi.com/documentation/computers/getting-started.html#raspberry-pi-imager) for additional information on this process.
2. Connect the Raspberry Pi to power, a keyboard, and an external display.
3. Insert the microSD Card into the Raspberry Pi.
4. Connect the USB mass storage device to the Raspberry Pi.
5. Run the OS installation.
    - Visit [Canonical's official installation tutorial](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi) for additional information on this process.

After the base OS installation is completed, additional software is required in order to run the ICS subsystem.

## Additional Software Setup

1. Install Python 3.12
    - Visit [the Python Foundation's official Beginner's Guide]( https://wiki.python.org/moin/BeginnersGuide/Download) for additional information on this process.
2. Configure Python 3.12
    - A virtual environment will be required. Visit [the Python 3.12 documentation](https://docs.python.org/3.12/library/venv.html) for additional information.
3. Install Apache
    - Visit [Canonical's official Apache installation tutorial](https://ubuntu.com/tutorials/install-and-configure-apache) for additional information on this process.
4. Configure Apache
    - Visit [Canonical's official manpage on Apache](https://manpages.ubuntu.com/manpages/focal/en/man8/a2ensite.8.html) for additional information on this process.
    - The following configuration must be applied and enabled:

        ```apache showLineNumbers
        <VirtualHost *:80>
            ProxyPreserveHost On
            ProxyPass / http://127.0.0.1:3000/
            ProxyPassReverse / http://127.0.0.1:3000/
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

        <Proxy />
            Require host localhost
            Require ip 127.0.0.1
            Require ip 192.168.2
            Require ip 192.168.3
        </Proxy>
        ```

5. Configure Git
    - Not required, if the source code is fetched directly from the repository.
    - Visit [GitHub's official Git Basics documentation](https://docs.github.com/en/get-started/git-basics/set-up-git) for more information on this process.

## DAS API Deployment

1. Verify that the Raspberry Pi is connected to the Internet via an ethernet connection. This will allow direct access to the Raspberry Pi after the WiFi configuration is changed.
2. Either through direct access or over a remote interface (i.e. SSH, VNC, etc.), clone the [DashAR repository](https://github.com/TrevorDBrown/DashAR) onto the device.
3. Navigate to DashAR/Source/Miscellaneous
4. Run the Bash shell script entitled "Setup-DashAR.sh". This will:
    - Set up the necessary directories and populate them.
    - Establish an ad-hoc network connection, by configuring the Wi-Fi to be an access point (AP). This is how the Augmented Reality Subsystem (ARS) connects to the ICS.
5. Schedule the Bash shell script "Start-DashAR.sh" to run at every reboot of the device.
    - It is recommended to utilize crontab for this, as cron is built into most common Linux distributions.
        - Visit [Canonical's official manpage on crontab](https://manpages.ubuntu.com/manpages/focal/man5/crontab.5.html)
    - If using crontab, use this configuration as a template:

        ```sh showLineNumbers
            # Add this to the bottom of your crontab file.
            # To edit, use "crontab -e".
            @reboot [Replace this with the path to the DashAR Project]/Source/Miscellaneous/Start-DashAR.sh
        ```

6. Active the Python virtual environment.
7. Install all Python requirements.
    - A requirements.txt is included in the source, in the Miscellaneous directory:

        ```python showLineNumbers
            # Not required, but useful for testing, as this package can emulate the ELM327 adapter.
            # Usage: python -m elm -s car
            ELM327-emulator==3.0.3
            pyserial==3.5

            # The following modules are specifically for das_service.py and files in das_core.
            obd==0.7.2                  # Enables OBDII access.
            tornado==6.4.1              # The HTTP server for the app.
            pythongping==1.1.4          # Not used, but reserved for future use, especially with Extensions.

            lockfile==0.12.2
            Pint==0.20.1
            python-daemon==3.1.2
            PyYAML==6.0.2
        ```

## Wi-Fi setup

Before the DashAR system can be used, it is important to verify the Raspberry Pi’s IPv4 address, so the ARS will function correctly.

1. Using a secondary, Wi-Fi enabled device, connect to the SSID emitted by the Raspberry Pi, named “DashAR-Network”.
    - The default password is “ConnectToDashAR”.
2. Once connected, check the assigned gateway’s IPv4 address (i.e. the Raspberry Pi's assigned IPv4 address).
    - The subnet mask applied, by default, is 255.255.255.0.
    - Most likely, the gateway IPv4 address will be 192.168.3.1.
3. Once the server’s IPv4 address is retrieved, remember it when building and configuring the ARS.
    - It is planned for a future release of the DashAR system to have autoconfigure functionality once the ARS connects to the network and the app is opened.

## Final Touches

1. Connect the OBDII-to-USB interface device to the Raspberry Pi.
2. Run the command `ls /dev/ttyUSB*`. This will be the access path for the OBDII interface.
3. Edit the config.json file in the DAS data directory, updating the key “OBDII_ELM327_DEVICE_PATH” with the device path retrieved.
    - The path most likely will be /dev/ttyUSB0 or /dev/ttyUSB1.

With the ICS now configured, this portion of the DashAR System is now ready for use!
