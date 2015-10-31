---
layout: post
title:  "[Raspberry Pi] Smarter Plants Tutorial"
date:   2015-10-21 18:34:14
categories: projects
---

This is a small side project I’ve been working on in my free time. The main reason why I started working on this was to learn more Python and Javascript, but to also get better with hardware related projects with the Raspberry Pi. And what better way to learn is there, than actually doing it and creating a step-by-step guide on how it works? So here we are, with this tutorial about a Soil Moisture Control through your Raspberry Pi!

At the end of this tutorial you will have a working Raspberry Pi with Soil Moisture Sensors wired up that is able to take data from your plant, store it in a CSV file and visualize it on a nice dashboard online. In addition to that, you will also be able to control your plant via Twitter. That means your plant will be able to tell you if it needs to be watered via a Tweet.

Overall, this project is work in progress and I intend to add more features to it and improvise the existing code. Most importantly, adding an automatic irrigation system will be the right step in creating a completely autonomous plant. A plant that is able to automatically satisfy its needs, that sounds pretty cool, right? One step closer to AI (sort of).

I’m mostly still a beginner with Raspberry Pi related development, so I did take help from people that have already created similar projects and I would like to publicly acknowledge some of them here: Jeremy Blythe with his amazing tutorial on setting up the Pi with the moistures sensors, seempie who wrote a similar tutorial and the people who created some amazing D3.js visualizations (named later). This tutorial takes pieces from all of these tutorials and combines them with new ones to create a perfect tutorial for beginners to get started.


# Overview

This tutorial is divided into 5 sections. First we are going to get the Pi’s GPIO properly wired up with the sensor on a breadboard, then we are going to write the Python script that takes the data and stores it in our CSV file, following with actually enabling the Pi to act as a Server (LAMP), then we will display the data on our website and visualize it through D3.js and lastly, we will add some tweaks through the Twitter integration.

This is what the final product will look like:

![alt tag](http://i.imgur.com/c60yPD5.png)


### What you will need

In total, this setup has cost around $20 without the actual Raspberry Pi. You will obviously need all the prerequisites, which are a working Raspberry Pi, with a keyboard, mouse and a monitor. Here is a list of the other things you will need:

* Soil Moisture Sensor: $4.50. I recommend getting this one from elecfreaks [http://www.elecfreaks.com/store/octopus-soil-moisture-sensor-brick-p-422.html](http://www.elecfreaks.com/store/octopus-soil-moisture-sensor-brick-p-422.html). As they doesn’t ship globally, I’ve had to use a different Moisture Sensor [http://www.play-zone.ch/de/erd-feuchtigkeitssensor-mit-digital-und-analogausgang.html](http://www.play-zone.ch/de/erd-feuchtigkeitssensor-mit-digital-und-analogausgang.html), but it works the same.
* MCP3008: $3.75 [http://www.adafruit.com/products/856](http://www.adafruit.com/products/856) This one is required to convert the analog signal to digital.
* Breadboard: $5
* Breakout Cable or around six female-to-male jumper wires. Cost: $5


## Step 1: Wiring the Sensor

In order to interact with our sensor, we need to connect it to the Raspberry Pi's GPIO (General Purpose Input Output). Here is a scheme that displays what each of the pins is used forward

![alt tag](/assets/rpi-smart-plant/Raspberry-Pi-GPIO-pinouts.png)

First we will connect the respective GPIO pins to the breadboard. Connect Pin 1 (which is 3.3V) to the Power Rail and Pin 6 (Ground) to the Ground Rail. Then place the MCP3008 chip in the middle of the breadboard.

![alt tag](/assets/rpi-smart-plant/mcp3008.gif)

Now we need to connect the pins to the MCP3008.

* MCP3008 VDD -> 3.3V (red)
* MCP3008 VREF -> 3.3V (red)
* MCP3008 AGND -> GND (black)
* MCP3008 CLK -> pin 23 (orange)
* MCP3008 DOUT -> pin 21(yellow)
* MCP3008 DIN -> pin 19 (blue)
* MCP3008 CS -> pin 24 (violet)
* MCP3008 DGND -> GND (black)

In the end, it should look like this.

![alt tag](/assets/rpi-smart-plant/20150804_181411.jpg)
![alt tag](/assets/rpi-smart-plant/20150804_181543.jpg)

Now the only thing missing is to connect the actual moisture sensor to the MCP3008. Connect the analog output of the sensor (yellow jumper cable in my case) to `CH5` on the MCP3008, then connect VCC (which is power) to the Power Rail and GND (which is Ground) to the Ground Rail. If all is fine, you should be ready to go! This is what it should look like:

![alt tag](/assets/rpi-smart-plant/20150804_171514.jpg)
![alt tag](/assets/rpi-smart-plant/20150804_171350.jpg)

## Step 2: Installing Apache Web Server

Apache is the most used web server software that helps server our HTTP requests. The setup for Apache is pretty straight forward:

```
$ sudo apt-get install apache2
```

If it isn’t running already, type in
```
$ sudo service apache2 start
```

Go to `127.0.0.1` and check if it’s working, if not the installation may have been faulty and you have to do some troubleshooting by Googling.

### Enabling Python for Apache

Now for this tutorial all the code is in Python, so we need to prepare the web server to run Python. We basically have to enable CGI Programming, described by [TutorialsPoint](http://www.tutorialspoint.com/python/python_cgi_programming.htm) “Common Gateway Interface, or CGI, is a set of standards that define how information is exchanged between the web server and a custom script.”.

As Python is already fully installed on your Raspberry Pi, you have to install the `python-setuptools helper package` and the `mod-wsgi` Apache module to enable us to run Python applications on our Apache server.

```
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
```

Now we need to edit the config file, so that the Apache server actually executes our .py files.
Type in:

```
$ sudo nano /etc/apache2/sites-available/default
```

Then change the current *<Directory /var/www/>* to:

```javascript
<Directory /var/www/>
	Options ExecCGI Indexes FollowSymLinks MultiViews
	AllowOverride None
	Order allow, deny
	allow from all
	AddHandler cgi-script.py
<Directory>
```

Now save and close the file. Then restart the server

```
$ sudo service apache2 restart
```

Now our Apache server should be able to run Python scripts. Lets make a small test just to be sure.

```
$ cd /var/www/
$ sudo nano testpython.py
```

Then type in the following test code:

```python

#!/usr/bin/python

print "Content-type:text/html\r\n\r\n"
print “<html>”
print “<body>”
print “<h1>Lets see if our Python works!</h1>”
print “<br>”
for i in range(6):
    print “<h3>
    print “Test: “ + str(i)
    print “</h3>”
print “</body>”
print “</html>”
```

If all is well, you should see the following page, all working well and displaying the code as intended.
![alt tag](http://i.imgur.com/1xHUtm7.png)


### Storing Data with CSV

Now some of you may wonder why we are not completing the full LAMP setup by using MySQL to store the Data. The reason for that is because of D3.js, which we are using for visualizing the data. Since D3.js works with CSV, TSV or JSON formatted data, if we were to choose MySQL we would first have to enter the data into our MySQL table and then later convert the data into CSV/TSV/JSON, which is a cumbersome process and makes things complicated.

That’s why we instead write to a CSV file directly and retrieve the data later when we visualize it on the website (3 steps turned into 2). As for the reason why we chose CSV, this is mainly because with JSON we cannot append data to the existing file. We would first have to load the entire data into a local array, append our data to it and then save it back into our JSON file. Such a process will certainly slow down our program as we accumulate more data from our plant. That’s why we choose a CSV, where we can append the data at the end of the file without having to load the entire data in first.

For this you now simply have to create a data.csv file with the following header:
`moisture,perc,date,time,weekday`

This is the description of the data we are retrieving and will make it possible for us to later make use of them in Javascript.


### Installing SpiDev and Python-Dev

Since we are using the SPI (Serial Peripheral Interface) bus for this tutorial, we are going to use the SpiDev wrapper in order to interact with our sensor. Python-dev is another library we require.

Run the following command:
```
$ sudo apt-get install python-dev
```

Now we need to enable the SPI communication on our Raspberry Pi, which is pretty straight forward. Just head to the following link and follow its steps: http://www.raspberrypi-spy.co.uk/2014/08/enabling-the-spi-interface-on-the-raspberry-pi/

Afterwards you have successfully restarted, type in:
```
$ cd ~
$ git clone https://github.com/Gadgetoid/py-spidev
$ cd py-spidev/
$ sudo python setup.py install
```

With this in hand we should be read to go and can now finally test our sensor on a plant! Congratz on making it so far, you can be sure that your plant will appreciate it.

## Step 3: Coding

Now the simplest program for us is to display the data into the terminal window. This program is great for testing if all the wiring is correct and your sensor is actually getting, converting and displaying the right data. This code was written by seempie from Instructables.com and it works as a base for us to add features as we proceed.

```python
#! /usr/bin/python
# python program to communicate with an MCP3008.
# Base Code written by seempie from instructables.com

# Import SpiDev wrapper and our sleep function
import spidev
from time import sleep

# Establish SPI device on Bus 0,Device 0
spi = spidev.SpiDev()
spi.open(0,0)

def getAdc(channel):
    #check valid channel
    if ((channel>7)or(channel<0)):
        return -1

    while True:
        # Perform SPI transaction and store returned bits in 'r'
        r = spi.xfer([1, (8+channel) << 4, 0])

        #Filter data bits from retruned bits
        adcOut = ((r[1]&3) << 8) + r[2]
        percent = int(round(adcOut/10.24))

        #print out 0-1023 value and percentage
        print "ADC Output: {0:4d} Percentage: {1:3}%".format (adcOut,percent)
        sleep(10)

if __name__ == '__main__':
    getAdc(0)
```



This should display the moisture of our plant and the percentage of it relative to the maxium (which happens to be 1024). Now before we actually get to more coding, we need to start collecting more data to be able to better understand it.

### Preparing the Data


Now the data here will vary depending on which sensor you have. What I highly suggest is that you run a few tests and determine what the upper bound and lower bound of the data is, meaning when the plant is completely wet or completely dry and what the average is. This way we can more better determine what the data should be and how we should properly visualize it on our website later on.

For determining the upper bound, meaning when the plant is completely dry, we simply place the sensor outside (not in the plant) and then run a few tests. In my case, the upper bound limit happens to be 1024.
The lower bound is determined by placing the sensor in a glass of water, where I got around 300. For detecting the average we simply place the sensor in the plant when we think it is in a “normal” state (normal defined that it wasn’t watered just recently and is also not too dry). Here the data can vary quite dramatically, and I have gotten between 600 - 850 depending on where in the pot I inserted the sensor.

![alt tag](http://i.imgur.com/FkEJxza.jpg)

With the data we gathered we can now more easily characterize the data we got and actually understand what it means.


### Fixing Permissions

Now we need to fix the permissions so that we have access from our /www folder directly to the SPI:

```
sudo adduser www-data spi
sudo adduser <username> www-data
sudo chown -R :www-data /var/www
sudo chmod -R g+rw /var/www
```

Replace <username> with your username.


### Getting the Code


For this tutorial I have prepared all the code already for you. You can check out the repo here: [https://github.com/domschiener/smart-plant-raspberry](https://github.com/domschiener/smart-plant-raspberry) So all you have to do is clone the repo and change a few settings to be fully operational.

```
$ git clone https://github.com/domschiener/smart-plant-raspberry.git
$ cd smart-plant-raspberry
```

Feel free to look around and learn more how everything's works together!


## Final Step: Enabling automated Twitter messaging


First of all we will need to create a new Twitter account for our plant. Since you actually care about your plant (else you wouldn’t follow this tutorial), take a profile picture for it as well and add some details (you can check out my plant over here: [https://twitter.com/domsplant](https://twitter.com/domsplant).

Once that is done, we will need to setup an application so that our plant (or rather the Raspberry Pi) can login to the Twitter account and communicate through it with OAuth.

```
$ git clone https://github.com/tweepy/tweepy.git
$ cd tweepy
$ python setup.py install
```

If you see an SSL error, run
```
$ sudo pip install requests[security]
```

Now you need to go to the **twitter.py** file and add your consumer key, consumer secret, as well as access token and access token secret which was provided to you by Twitter.


# Running the Application


Now everything should be ready to function and we can start testing it all. Simply run:

```
$ sudo python main.py
```

This in turn will run twitter.py and getdata.py in the background. You can now go to *http://localhost* and everything should be operational :)
