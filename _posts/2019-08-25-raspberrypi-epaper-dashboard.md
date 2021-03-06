---
title: "Raspberry Pi: Waveshare e-paper dashboard"
description: "Raspberry Pi dashboard with an e-paper display from waveshare"
categories: 
  - raspberrypi
tags: 
  - raspberrypi
  - epaper
  - dashboard
gallery:
  - url: /assets/images/raspberrypi-epaper-dashboard/001.jpg
    image_path: /assets/images/raspberrypi-epaper-dashboard/001thumb.png
    alt: "In a picture frame"
    title: "In a picture frame"
  - url: /assets/images/raspberrypi-epaper-dashboard/002.png
    image_path: /assets/images/raspberrypi-epaper-dashboard/002.png
    alt: "Generated image"
    title: "Generated image"

gallery2:
  - url: /assets/images/raspberrypi-epaper-dashboard/003.png
    image_path: /assets/images/raspberrypi-epaper-dashboard/003.png
    alt: "Cutout for e-paper connector"
    title: "Cutout for e-paper connector"
  - url: /assets/images/raspberrypi-epaper-dashboard/004.png
    image_path: /assets/images/raspberrypi-epaper-dashboard/004.png
    alt: "Topdown view or Raspberry Pi attached to picture frame stand"
    title: "Topdown view or Raspberry Pi attached to picture frame stand"    
  
---

Instructions on setting up a Raspberry Pi Zero WH with a Waveshare ePaper 7.5 Inch HAT. 
The screen will display: 

* Date and time
* Weather icon with high and low temperature
* Google Calendar entries
* Pihole stats

Here it is in action

{% include gallery caption="epaper dashboard" %}


## Shopping list

### E-Paper Display

The most important component is the Waveshare display, which is a [7.5 inch e-paper HAT](https://www.waveshare.com/wiki/7.5inch_e-Paper_HAT) with `SKU: 13504` and `UPC: 614961951068`.  A quick search will also show similar displays available, with a single additional color. As tempting as they may be, the problem with those displays is the refresh rate, in part due to the way the third color is 'pushed' to the surface when displaying a color.  While the black and white display isn't very fast, the colored ones are much, much slower and are only suitable for frequently-refreshing dashboards.  

[E-Paper Display](https://smile.amazon.co.uk/gp/product/B075R4QY3L/){: .btn .btn--info}

### Raspberry Pi

Although any Raspberry Pi can be used, the best one to get here is the Raspberry Pi Zero W - it's thinner and more portable.  Since it's a HAT (Hardware Attached on Top), you can save some time by buying it with the GPIO presoldered.  Of course you'll also need a microSD card.  

[Raspberry Pi Zero WH](https://smile.amazon.co.uk/gp/product/B07BHMRTTY/){: .btn .btn--info}
[microSDHC card](https://smile.amazon.co.uk/gp/product/B073K14CVB){: .btn .btn--info}

### Picture frame

You'll need a 6"x4" picture frame to hold everything together.  This is the best size just larger than the e-paper display.  The back needs to be made of cheap material so that it can be cut out for the e-paper display's connection mechanism.  

[Picture frame](https://www.wilko.com/wilko-6-x-4-inch-black-easy-photo-frame/p/0242493){: .btn .btn--info}


## Setup the PI

### SD Card

Use [Etcher](https://etcher.io) to write the SD card with the [Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/) image, no need for the desktop version. 


### Enable SSH 

After the image has been written create a file called `ssh` in the boot partition of the card.

```bash
sudo touch /media/mendhak/boot/ssh
```

### Enable WiFi

Create a file called `wpa_supplicant.conf` in the boot partition 

```bash
sudo nano /media/mendhak/boot/wpa_supplicant.conf
```

with these contents, substituting your Wifi's SSID and password.  You'll also need to change the `country` to match yours.  


    update_config=1
    country=GB

    network={
        ssid="yourwifi"
        psk="wifipasswd"
        key_mgmt=WPA-PSK
    }


### Connect the display

Put the HAT on top of the Pi's GPIO pins.  

Connect the ribbon from the epaper display to the extension.  To do this you will need to lift the black latch at the back of the connector, insert the ribbon slowly, then push the latch down. 


### Start the Pi

Insert the SD card.  Connect the Pi to power, let it boot up.  In your router devices page, a new connected device should appear.  If all goes correctly then the pi should be available with its FQDN even.  You can now SSH in. 

```bash
ssh pi@raspberrypi.lan
```

Login with the default password of raspberry and change it using the `passwd` command.


## Setup dependencies

The display as well as the code that talks to the display has quite a few dependencies.  Here we're installing a few fonts, some Python dependencies.

```bash
sudo apt install git ttf-wqy-zenhei ttf-wqy-microhei python3-pip python-imaging libopenjp2-7-dev libjpeg8-dev inkscape figlet
```

### Enable SPI

[SPI](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi/all) is how the Raspberry Pi will send data over the ribbon to the e-paper display.   You will need to enable it and reboot the device.  

```bash
sudo sed -i s/#dtparam=spi=on/dtparam=spi=on/ /boot/config.txt  #This enables SPI
sudo reboot
```

### Get the BCM2835 library

The BCM2835 library provides access to the GPIO pins and it needs to be manually installed.  

```bash
wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.58.tar.gz
sudo tar zxvf bcm2835-1.58.tar.gz
cd bcm2835-1.58/
sudo ./configure
sudo make
sudo make check
sudo make install
```

### Get the WiringPi library

WiringPi also provides access to the GPIO, and also needs to be manually installed. 

```bash
sudo git clone git://git.drogon.net/wiringPi
cd wiringPi
sudo ./build
```

### Get the Python3 libraries

Installing Pillow took a few attempts as it always seemed to be missing some dependencies.  

```bash
sudo pip3 install spidev RPi.GPIO Pillow  
sudo pip3 install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib
```
    


## Using this application

You can now use the code from this repo


### Clone it

Clone this repository to your Raspberry Pi

```bash
git clone git@github.com:mendhak/waveshare-epaper-display.git
```


### (Optional) Build the displayer

The `display/display` executable does the work of sending the image over to the screen and is already included in the repo.  If you need to rebuild it however, 

```bash
cd waveshare-epaper-display
cd display
make
```    

### DarkSky API key

The DarkSky API will be used to get the weather conditions.  Modify the `env.sh` file and put [your DarkSky API key](https://darksky.net/dev) in there. 

```bash
export DARKSKY_APIKEY=xxxxxx
```


### PiHole info

If you have a [Pi-Hole](https://pi-hole.net/) set up, you can display stats from its [API](http://pi.hole/admin/api.php) on screen.   Modify the `env.sh` and add the domain of the PiHole in there, eg `pi.hole` or `192.168.0.111`

```bash
export PIHOLE_ADDR=192.168.0.111
```

### Google Calendar info

The script will by default get its info from your primary Google Calendar.  If you need to pick a specific calendar you will need its ID.  To get its ID, open up [Google Calendar](https://calendar.google.com) and go to the settings for your preferred calendar.  Under the 'Integrate Calendar' section you will see a Calendar ID which looks like `xyz12345@group.calendar.google.com`.  Set that value in `env.sh`

```bash
export GOOGLE_CALENDAR_ID=xyz12345@group.calendar.google.com
```

### Google Calendar token

In order to display Google Calendar events, we'll need an OAuth Token.  However Google have not made the process of obtaining one simple at all.  The Oauth process needs to complete once manually in order to allow the Python code to then continuously query Google Calendar for information. 

Go to the [Python Quickstart](https://developers.google.com/calendar/quickstart/python) page and enable Google Calendar API.  When presented, download or copy the `credentials.json` file and add it to this directory. 

Next, on the Raspberry Pi, run: 

```bash
python3 screen-calendar-get.py
```

The script will prompt you to visit a URL in your browser and then wait.  Copy the URL, open it in a browser on another PC, and you will go through the login process.  When the OAuth workflow tries to redirect back (and fails), copy the URL it was trying to go to (eg: `http://localhost:8080/...`).  Open up a brand new SSH session to the Raspberry Pi, and curl that URL. 

```bash
curl "http://localhost:8080/..." 
```

On the first SSH session, you should now see the auth flow complete, and a new `token.pickle` file appears.  The Python script should now be able to run in the future without prompting required.  





## Run it

Run `./run.sh` which should query DarkSky, PiHole, Google Calendar.  It will then create a png, convert to a 1-bit black and white bmp, then display the bmp on screen. 
After a few runs, if everything is working well, you should then make this a cron job. 

```bash
*/1 * * * * cd /home/pi/waveshare-epaper-display && bash run.sh > run.log 2>&1
```

## Putting it in a picture frame

The picture frame I got had a cheap backing.  Using a box cutter (Stanley knife) I was able to remove a square portion from the bottom.  This allowed me to put the e-paper display inside the picture frame while its connector hung outside.  

The ribbon from the connector loops upwards and over to the picture frame's stand.  The Raspberry Pi Zero WH is light enough that it could be taped right to the stand.  

The only bit of wire in the whole setup is the USB to power the Raspberry Pi.  

{% include gallery id="gallery2" caption="Picture frame details" %}


## How it works

Everything starts with the `screen-template.svg` which holds the labels and layout for the final image to be produced. SVGs are simply XML files which are understood by renderers.  Being text files makes them easy to work with from dynamic scripts.  



### API Calls

The first part of `run.sh` calls on the `screen-weather.get.py` script which queries DarkSky API, gets the weather info and substitutes icons and temperatures in the SVG.  It also sets the date and time.  The SVG is then written out to `screen-output-weather.svg`.  The API response is stored in 


Next is the Pihole summary, queried using `screen-pihole-get.py` which invokes the Pihole API and gets the summary, again substituting it in the `screen-output-weather.svg` 

The last API call is to Google Calendar, the upcoming 2 calendar entries are written to the same SVG.  

Due to API rate limits, you will see various `.pickle` files which store the Google Calendar and Dark Sky API responses for a few hours.  This means that any new entries in your target calendar won't show up immediately.  Similarly weather info will be up to a few hours delayed.  
{: .notice--info}

### Image conversion and display

The image is converted from the intermediate SVG to PNG, and then converted from a PNG to a 1-bit bitmap.  Using a 1-bit, low grade BMP is what allows the screen to refresh relatively quickly. In my example this refresh takes about 6 seconds.  

It's possible to use Python libraries to display images directly to screen, however it's slow, really slow.  Rendering a 'high quality' JPG/PNG to screen, using Python, took about 35 seconds.  



## Learn more: Waveshare documentation and sample code


Waveshare have a [user manual](https://www.waveshare.com/w/upload/7/74/7.5inch-e-paper-hat-user-manual-en.pdf) which you can get to from [their Wiki](https://www.waveshare.com/wiki/7.5inch_e-Paper_HAT)


The [Waveshare demo repo is here](https://github.com/waveshare/e-Paper).  Assuming all dependencies are installed, these demos should work.  

    git clone https://github.com/waveshare/e-Paper waveshare-epaper-sample
    cd waveshare-epaper-sample





### Run the BCM2835 demo


    cd ~/waveshare-epaper-sample/7.5inch_e-paper_code/RaspberryPi/bcm2835/
    make
    sudo ./epd


### Run the WiringPI demo

    cd ~/waveshare-epaper-sample/7.5inch_e-paper_code/RaspberryPi/wiringpi/
    make
    sudo ./epd

### Run the Python3 demo

    cd ~/waveshare-epaper-sample/7.5inch_e-paper_code/RaspberryPi/python3/
    sudo python3 main.py
