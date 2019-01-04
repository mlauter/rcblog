---
layout: post
title: "How to Make Your Own Smart AC"
description: Or, the Internet of Things comes to my room. 
date: 2014-09-10T19:56:51-04:00
tags: internet of things, home automation, DIY, instructable, raspberry pi, flask
---

<img align="right" src="http://techli.com/wp-content/uploads/2011/10/415.jpeg">
Remember that late-90s Disney Channel Original Movie [*Smart House*](http://en.wikipedia.org/wiki/Smart_House_(film)) in which a young, motherless, computer-genius boy wins a fully automated robot-house, only to have the house-software come alive as an overbearing 50s-esque mother who locks the boy and his family indoors 'for their own good'? 
<p><br /></p>
...maybe you don't remember. But *I* do, and I have always wanted to build that. Except without the sexist/robot-takeover part (jeez Disney, what is your deal with mothers?).

<p><br /></p>
<!-- <p><br /></p>  -->

Well, I didn't quite do that. But, I did build a pretty neat device and web app for controlling my air conditioner. 
<!-- <p><br /></p> -->
<!-- how to phrase?? -->

# Here's a step-by-step guide to what I did.
*Note, before starting this project, I'd never used a raspberry pi (I'd done just a tiny bit of Arduino), I'd never written a server, and I'd never made a web app of any kind. So, if you're coming to this with just a bit of programming experience, take heart! This is a totally doable project.*

## Parts and Tools

![toolkit](../images/ac_project_toolkit.jpg)

### Components of the finished project
1. Raspberry Pi (I used [this](https://www.adafruit.com/products/1914) one)
1. [Power Switch (or just a relay)](https://www.adafruit.com/products/268)
1. [USB wifi for raspberry pi](https://www.adafruit.com/products/814)
1. [A micro SD card](https://www.adafruit.com/products/102) (if you are using RPi B+, otherwise check your specs)
1. [Digital temperature sensor](https://www.adafruit.com/products/381)
1. [Micro USB charger for the raspberry pi](https://www.adafruit.com/products/592) + [wall adapter](https://www.adafruit.com/products/501)
1. 4.7K resistor
1. Jumper wires, both [male](http://www.ebay.com/itm/40-Pin-20cm-Dupont-Wire-Connector-Cable-2-54mm-Male-to-Male-1P-1P-For-Arduino-/221455439425?ssPageName=ADME:X:RTQ:US:1123) and [female](http://www.ebay.com/itm/Dupont-Wire-20cm-Cable-Line-Color-40P-40P-Test-Lines-Connector-/221402540531)
  *Tip! this ebay seller is reliable and cheap*
1. [Prototype Paper PCB ](http://www.ebay.com/itm/10pcs-DIY-Prototype-Paper-PCB-Universal-Experiment-Matrix-Circuit-Board-5-x-7cm-/221542018120]) (DIY circuit board)
1. Your computer

### Tools
1. Soldering iron and solder
1. A very very skinny phillips head screwdriver
1. A hot glue gun

### For testing/setup only
1. A [breadboard](https://www.adafruit.com/products/64)
1. An [LED](https://www.sparkfun.com/products/9590)
1. A monitor, keyboard, and ethernet cable (for setting up the pi)

## Set up the Pi
I cannot stress this strongly enough: __[Adafruit](https://learn.adafruit.com/) tutorials for allll the things!__ Their tutorials are super informative and easy to follow. Plus they also sell all the parts. It's a one stop super shop for all things hardware.

If you already have a Raspberry Pi set up, or you feel comfortable getting started by going directly to Adafruit, downloading an OS image to your SD card, configuring, setting up wifi, etc, just continue with this post.

If you want some more help getting started, check out my [companion blog post](../getting-started-with-raspberry-pi) on getting your Raspberry Pi set up as a first-time user.

# Overview

![diagram](../images/acproject_schematic.jpg)

_If you don't want to read this whole post and just want to see the code I used, check out [my github repo](https://github.com/mlauter/RemoteAC). `ac_ping.py` contains the code running on the Pi, while the app folder contains everthing for the server and UI, including the Procfile for deploying on Heroku. You'll have to change the URLs, secret key, and username password, I've put in place-holders._

# Part I: The Hardware: building your circuit

Before I get started with this section of the post, I want to thank [Dana Sniezko](http://polkapolka.net/), a fellow Hacker Schooler and hardware genius, for helping me SO much with this project. Seriously, Dana, if you're reading this, thank you for letting me bug you with questions for weeks and for teaching me all this awesome stuff. 

## Raspberry Pi -- PowerSwitch (relay) -- Air conditioner

As you can see in the diagram, this is the only part of the entire system that interacts with the air conditioner. Instead of connecting your air conditioner's power plug directly into the wall socket, you will plug it into the PowerSwitch Tail II, and plug the other end of that into the wall. (Make sure to check your air conditioner's electrical specifications, because the PowerSwitch can only handle up to 15 amps. Most ACs will draw far less than that, though, mine only draws about 5). 

Now the cool part. Inside the black box that is now between your air conditioner and the wall socket there is a relay, which is just an electrically controlled switch. When you send power to the switch from one of your Raspberry Pi's GPIO (general purpose input/output) pins, the switch will close and power will flow between the wall and the AC. When no power is flowing from the Pi to the switch, the switch will be open, and no power will flow between the wall and the AC. 
![powerswitch](../images/powerswitch.jpg)
Make sure that your AC is always set to be 'on', we're bypassing the air conditioner's normal on/off control system. 

Let's set that up. Consult the [pinout map](http://www.raspberry-projects.com/pi/wp-content/uploads/2014/09/rpi_model_b_plus_io_pinouts.jpg) for your Raspberry Pi. I find it useful to look at one that shows a picture of the Pi, so you can make sure you're oriented correctly.

![pinoutmap](../images/rpi_model_b_plus_io_pinouts.jpg)

1. Make sure your Raspberry Pi is not plugged into power while you are working on this.
1. Choose any pin labeled GPIO except GPIO4 (which we need to reserve for our temperature sensor). 
  For example, on the map above, pin number 11, the 6th pin down in the left column is GPIO17. 
1. Take one of your jumper wires (a red one would be a good choice), and connect it to that pin. 
1. Now connect one of the male to male jumper wires to that wire. 
1. Find a pin labeled ground (GND) and connect another wire (black or grey would be a good choice), and again connect that to one of the male to male wires.
1. Connect the power (wire from the GPIO pin) to _1: +in_ on the PowerSwitch, and your ground wire to _2: -in_. Ignore the spot labeled _3:Ground_ on the PowerSwitch. 
1. You'll notice that on the face of the power switch with labels, there are some holes with screws in them. Use a small screw driver to adjust these screws such that your jumper wires are secured in the powerswitch and cannot be tugged out. 
1. *Note: while you are testing, you don't need the power switch at all, just plug the wires into a breadboard and [use an LED](https://projects.drogon.net/raspberry-pi/gpio-examples/tux-crossing/gpio-examples-1-a-single-led/). If you can turn the light on and off, you can turn your AC on and off.*

![power switch plugged in](../images/powerswitch_connected.jpg)

### RPi-GPIO library
[Install RPi-GPIO on your Pi](https://learn.adafruit.com/playing-sounds-and-using-buttons-with-raspberry-pi/install-python-module-rpi-dot-gpio)

~~~~~~~~~
import RPi.GPIO as io
import time

io.setmode(io.BCM)
switch_pin = 17
io.setup(switch_pin, io.OUT)

#blink 10 times
for i in range(10):  
    io.output(switch_pin, False)   # set output pin LOW (off)
    time.sleep(5)   # sleep for 5 seconds
    io.output(switch_pin, True)    # set out put pin HIGH (on)
    time.sleep(5)
io.cleanup() #reset the pins
~~~~~~~~~~
{: .language-python}

We will adapt this blinking light for controlling the relay later on. Note, in order to use GPIO you have to run as root! 

## Digital Temperature sensor

Checkout [this tutorial](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-11-ds18b20-temperature-sensing/hardware). The circuit diagram is slightly confusing so here is another I find easier to read:

![temp sensor circuit](../images/RasPi_DS18B20.jpg)

For testing, set this up on the circuit board. You can send power and ground to the bus along the side.

<p><br /></p>
When you set this up for real: 

1. Take a sheet of PCB (it doesn't need to be large, so you can break off a piece if you want)
1.. Cut off the connecter end of the jumper wires coming from the pi and strip off the covering
1. Put the wire through a hole of the pc.
1. Strip down the leads from the temperature sensor.
1. Solder red to red, yellow to yellow, black to black.
1. Solder your resistor between red and yellow.
1. Use your hot glue gun to put a blob of glue over the blobs of solder. (Not strictly necessary, but a good thing to do).
1. Use something to secure the cord of the temperature sensor to your circuit board (so that you don't end up putting a lot of strain on individual wires). I used a hairband.

Here are some pictures of my circuit, both top and bottom sides.
![my temp sensor circuit](../images/tempsensorcircuit_frontback.jpg)

### The code
Continue following the [adafruit tutorial from above](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-11-ds18b20-temperature-sensing/ds18b20) to set up the directory for the temperature sensor for the first time. 

Once you've done that, here is the code you will use. I copied it directly into my own program. *Note: I noticed that for some reason, every time I wanted to restart my program, I had to cd into my device folder first, otherwise I'd get an error saying that the folder didn't exist.*

~~~~~~
import os
import glob
import time
 
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')
 
base_dir = '/sys/bus/w1/devices/'
device_folder = glob.glob(base_dir + '28*')[0]
device_file = device_folder + '/w1_slave'
 
def read_temp_raw():
    f = open(device_file, 'r')
    lines = f.readlines()
    f.close()
    return lines
 
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        temp_f = temp_c * 9.0 / 5.0 + 32.0
        return temp_c, temp_f
~~~~~~
{: .language-python}

That's it for the hardware side of this project. I'll return to the full program I have running on the Pi in Part III. 

# Part II: The Server

If this is not new to you, write your server however you want, just make sure that your pi can make POST requests to some route that will add information to a database.

If this is new to you, I'll do a quick overview of my Flask server, and you can check out the code for yourself on [github](https://github.com/mlauter/RemoteAC/blob/master/app/app.py). Note, this will not be a Flask tutorial. If you've never used Flask or written a server, I highly recommend [Miguel Grinberg's Flask Mega-Tutorial](http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world). Though it is much more information than you need for this project. I also recommend the [realpython tutorial](https://realpython.com/blog/python/python-web-applications-with-flask-part-i/) and their ['discover flask'](https://github.com/realpython/discover-flask) video series.

The server I've written is intended to be used by just two clients, you, and your Raspberry Pi. It is not intended for a large scale app.

## The three main routes

My server has three main routes, `/index`, `/ac_status`, and `/switch_state`.Here is what they do.

### /Index : The Homepage
This route (which can also be accessed with the bare `/`), runs the homepage function which returns the homepage, `index.html`. 

~~~~~~
@app.route('/', methods = ['GET','POST'])
@app.route('/index', methods = ['GET','POST'])
def homepage():
    current_log = db.get_last_ac_state()
    room_temp = current_log[2]
    is_running = bool(current_log[3])
    return render_template('index.html', room_temp = room_temp, 
                            is_running = is_running, 
                            time=datetime.datetime.now())
~~~~~~
{: .language-python}

### /ac_status: The route for the Raspberry Pi

This route only accepts POST requests. We will run code on the Pi to POST to this route every second with its most up-to-date state information. 

The function called in this route `update()` will take the info from the Pi, and store it in a sqlite database (using the `sqlite3` module in python). Check out `db.py` to see my database functions.

It will then return the information stored in a global variable called `desired-state` to the Pi. 

~~~~~~
AcState = namedtuple('AcState', 
                    ['timestamp','room_temp','is_running', 
                    'state_num','goal_temp'])

@app.route('/ac_status', methods=['POST'])
def update():
    response = request.json
    
    room_temp = response['room_temp']
    is_running = response['is_running']
    state_num = response['state_num'] #1, 2, or 3
    #an empty string or an integer
    goal_temp = str(response['goal_temp']) 
    print room_temp
    print "I heard from the AC"
    db.add_ac_state(AcState(datetime.datetime.now(),
                            room_temp,is_running,
                            state_num,goal_temp))

    global desired_state
    return jsonify(desired_state)
~~~~~~
{: .language-python} 

I will discuss how I handle the `desired-state` shortly.

### /switch_state: The route that the UI will hit

This route takes both GET and POST requests, and it calls the function, `switch_state`. 

First, it gets the latest information from the database. If the request is just a GET, the function will simply return the latest information from the database to the browser. If it's a POST request, however, we'll first call another function `statify`, to translate what the user has asked for on the browser end, into something easy to convey to the air conditioner, and we'll update the global `desired_state` variable.

~~~~~~
@app.route('/switch_state', methods=['GET','POST'])
def switch_state():
    current_log = db.get_last_ac_state()
    current_state = (current_log[4], current_log[5])

    #if we have a post request, do this stuff, 
    #otherwise just populate the page 
    #with the latest database state
    if request.method == 'POST':
        print request.json
        global desired_state
        desired_state = statify(request.json)
        desired_state_tup = (desired_state['state_num'],
                             desired_state['goal_temp'])
        current_log = db.get_last_ac_state()
        current_state = (current_log[4], current_log[5])

    #need to return stuff that the browser will then use
    return jsonify(is_running = current_log[3], 
                   state_num=current_state[0],
                   goal_temp=current_state[1])
~~~~~~
{: .language-python} 

### Representing the state of the AC: desired_state and statify

In order to make the program on the Pi as simple as possible, I've decided to represent what I want the air conditioner to be doing in three different states. **1: Completely and totally OFF. 2: Completely and totally ON. 3: Trying to maintain a particular temperature**

Because state 3 requires a temperature, I've represented all the states as a dictionary of `{state number:?, temperature:?}`, where the temperature is the empty string if we have state 1 or 2, or an integer (converted to a string) in fahrenheit for state 3. 

~~~~~~
#initialize desired_state to a dictionary 
#representing the off state
desired_state={'state_num':1,'goal_temp':''}

def statify(ui_state):
    #takes in inputs from the browser and returns 
    #an allowable state to give the AC as a dictionary. 
    allowed_states = {'OFF':{'state_num':1,'goal_temp':''},
                      'ON':{'state_num':2,'goal_temp':''},
                      'MANAGE_TEMP':{'state_num':3,
                                     'goal_temp':str(ui_state['desired_temp'])}}
    cleaned_state = {}
    if ui_state['desired_power_state'] == False:
        cleaned_state=allowed_states['OFF']
    elif ui_state['desired_mode_is_home']:
        cleaned_state=allowed_states['MANAGE_TEMP']
    else:
        cleaned_state=allowed_states['ON']
    return cleaned_state
~~~~~~
{: .language-python}

### Login and logout routes

I've added login and logout routes to my final server program. I'm not going to explain how to do that here, but you can look at my code, or go to this [realpython tutorial](https://realpython.com/blog/python/introduction-to-flask-part-2-creating-a-login-page/). I recommend watching the video. Make sure to add the @login_required decorator to the appropriate functions.

### HTML/User interface

You'll need some pages for these routes to return. To start out, I recommend just using some plain HTML, and later, you can add onto it with some javascript and CSS to make the UI better. I'm using some jQuery and CSS Bootstrap. My HTML templates are [here](https://github.com/mlauter/RemoteAC/tree/master/app/templates), js is [here](https://github.com/mlauter/RemoteAC/blob/master/app/static/on_off.js).

UI design was decidedly *not* the focus of this project, and I'd love suggestions for making it better:

![UI](../images/ac_website.png) 
 
# Part III: The Raspberry Pi program

Now that we have our server, we can flesh out the code that's going to run on the pi. 

So far, we have some functions that allow us to read from the temperature sensor (I've added one that just returns the temperature in fahrenheit), and we've imported the `RPi-GPIO` library and set up our switch pin. I've also added a function to read off the value of the switch pin, so we know whether power is flowing to our AC or not.

~~~~~~
import RPi.GPIO as io
import time
import os
import glob

#setup for GPIO
io.setmode(io.BCM)
switch_pin = 17
io.setup(switch_pin, io.OUT)
io.output(switch_pin, False)

#setup for the temperature sensor
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')
 
base_dir = '/sys/bus/w1/devices/'
device_folder = glob.glob(base_dir + '28*')
device_file = device_folder[0] + '/w1_slave'

#functions to read from temp sensor
def read_temp_raw():
    f = open(device_file, 'r')
    lines = f.readlines()
    f.close()
    return lines
 
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        temp_f = temp_c * 9.0 / 5.0 + 32.0
        return temp_c, temp_f

def current_temperature():
    temperature = read_temp()[1]
    return int(temperature)

#Check the whether switch_pin is HIGH or LOW
def is_running():
    return io.input(switch_pin)
~~~~~~
{: .language-python}

Now, we'll just add two more main functions to this program. One to make POST requests to the server, and one to set the state of the air conditioner.

## POST requests to the server

This is the 'hey, server!' function. In the main loop of our program, we'll call it every second. The Raspberry pi will POST its current state to the server, and the server will respond with the desired state from the user. We can then pass this information off to a different function that will decide what to do with the information.

First, install and import the python [Requests](http://docs.python-requests.org/en/latest/) library.

~~~~~
import requests

def send_current_state():
    r = requests.post(url, 
                      data=some data, 
                      headers=some header, 
                      timeout=5)
~~~~~
{: .language-python}

Now we'll define the parameters for the post request to use. The url should be either the IP address of your computer, if you are testing on localhost, or the address of your site, wherever you are hosting it, followed by the name of the route to hit (ac_status).

The headers I've set allow requests to send data as json. 

~~~~~
import requests

def send_current_state():
    url = "your.url.here/ac_status"
    headers = {'Content-type': 'application/json', 
               'Accept': 'text/plain'}
    r = requests.post(url, 
                      data=some data, 
                      headers=some header, 
                      timeout=5)
~~~~~
{: .language-python}

We also need to decide what data to send to the server. Looking back at the code for our server, we need to send the current temperature in the room and  whether the air conditioner is on or off (We can get these values from the functions we just wrote). We also need to send the state number and goal temperature that together represent the air conditioner's current state. *Note: I have a separate value for on or off so I can always know whether the air conditioner is on or off, even when the AC is acting as a thermostat.*

In order to keep track of the state of the air conditioner, I'm going to initialize a variable in the main loop of the program called `state`.

~~~~~
import requests

def send_current_state():

    data_to_be_sent = {
        "is_running": is_running(),
        "room_temp": current_temperature(),
        "state_num": state['state_num'],
        "goal_temp": state['goal_temp']
    }

    url = "your.url.here/ac_status"
    headers = {'Content-type': 'application/json', 
               'Accept': 'text/plain'}
    r = requests.post(url, 
                      data=some data, 
                      headers=some header, 
                      timeout=5)

if __name__ == "__main__":
    state={'state_num':1,'goal_temp':''}
    while 1:
        time.sleep(1)
        send_current_state()
~~~~~
{: .language-python}

The last thing this function needs to do is get the server's response and send off the user's desired state to another function. We've called the request object `r`, and when the POST request is made successfully, `r.json()` will give you the server's response as json.

~~~~~
import requests

def send_current_state():

    data_to_be_sent = {
        "is_running": is_running(),
        "room_temp": current_temperature(),
        "state_num": state['state_num'],
        "goal_temp": state['goal_temp']
    }

    url = "your.url.here/ac_status"
    headers = {'Content-type': 'application/json', 
               'Accept': 'text/plain'}
    r = requests.post(url, 
                      data=some data, 
                      headers=some header, 
                      timeout=5)
    state_num = r.json()['state_num']
    goal_temp = r.json()['goal_temp']

    #The function we have to write next
    set_state(state_num,goal_temp)                      

~~~~~
{: .language-python}

## Setting the air conditioner's state

We'll define a function, `set_state` that will take two arguments, a state number (1, 2, or 3) and a goal temperature, which, as we defined in our server code will be the empty string if the state number is 1 or 2. 

~~~~~
def set_state(state_num, goal_temp):
    # get the room temperature again
    room_temp = current_temperature()

    #handle states
    if state_num == 1:
        pass
    elif state_num == 2:
        pass
    else:
        pass

    #set the new state
    state['state_num'] = state_num
    state['goal_temp'] = goal_temp
~~~~~
{: .language-python}

**State 1** means the air conditioner should be off no matter what, and **State 2** means the AC should be on no matter what, so these conditions are quite simple. We'll either set the switch pin HIGH or LOW.

~~~~~
    #handle states
    if state_num == 1:
        io.output(switch_pin, False)
        goal_temp = ''
    elif state_num == 2:
        io.output(switch_pin, True)
        goal_temp = ''
    else:
        pass
~~~~~
{: .language-python}

**State 3** is slightly more complicated. The air conditioner should compare the temperature requested by the user to the room temperature, turn or stay on if the room is too hot, and turn or stay off if the room is at the right temperature or too cold. 

In an effort to save energy and keep the air conditioner from turning on and off in rapid succession right on the edge of the goal temperature, I set a temperature range. The lower-bound is the goal temperature and the upper-bound is two degrees warmer. (Using it in my own room, I found this to be a reasonable and comfortable range). Of course, the air conditioner still vacillates rapidly around the borders of this range, an issue I have yet to fix. A better way to handle this might be to disallow changes for a certain amount of time after the air conditioner has turned on or off. I would love suggestions! I've made it an open issue on the github repo. 

~~~~~
    else:
        # don't get warmer than 2 degrees above goal
        goal_temp_range_max = int(goal_temp) + 2 
        if room_temp > goal_temp_range_max:
            # if we're hotter than the max, turn on
            io.output(switch_pin, True)
        elif room_temp <= goal_temp:
            # if we're at or under the temp range, turn off
            io.output(switch_pin, False)
        #otherwise do nothing
~~~~~
{: .language-python} 

## Cleanup and error handling

My final program has a little bit more code in it to handle problems. The goal of this code is to a) keep the program alive even if the server doesn't respond, and b) make sure to turn the air conditioner off if something goes seriously wrong or the program is going to exit. 

My strategy for waiting out periods of server unresponsiveness is: 

1. Keep track of the time of the last successful communication
1. If the server doesn't respond and it's been less than five minutes since the last successful request, just keep trying.
1. If it's been more than five minutes, turn the AC off and keep trying.

~~~~~
def signal_handler(signal, frame):
    cleanup()

def cleanup():
    io.cleanup()
    sys.exit(0)

if __name__ == "__main__":
    state={'state_num':1,'goal_temp':''}
    last_connect = None
    try:
        while True:
            time.sleep(1)
            try:
                pre_connect = datetime.now()
                send_current_state()
                last_connect = datetime.now()
            except IOError as e:
                print e
                t_delt = timedelta(minutes=5)
                if last_connect:
                    if datetime.now()>(last_connect+t_delt):
                        io.output(switch_pin,False)
                else:
                    if datetime.now()>(pre_connect+t_delt):
                        io.output(switch_pin,False) 
    except Exception as e:
        print e
    finally:
        cleanup()
~~~~~
{: .language-python} 
  
# Final notes

## Host your server somewhere or deploy it to Heroku

I followed [these instructions](https://devcenter.heroku.com/articles/python-gunicorn) to deploy my server to Heroku. You will need a Procfile. Mine is in the repo in the app directory, and it's just one line:

`web: gunicorn app:app --log-file=-`

## Keep your Pi program running without an ssh session

Once your server is deployed, make sure you put in the correct URL into the program that will run on the Raspberry Pi. 

In order to keep a process running on the Pi even after you've closed your ssh session, just put `nohup` in front of the command you would normally run. Also, keep in mind that the GPIO library requires root privileges! 

`nohup sudo python ac_ping.py`

<p><br /></p>

# Now you have your own Smart AC!

Go make cool stuff!





