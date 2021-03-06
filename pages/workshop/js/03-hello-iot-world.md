---
layout: page-fullwidth
title: "Hello, IoT World!"
subheadline: "Node.js - Connected Weather Station"
teaser: "In this lab you will use one of the digital output pins to send a signal to an LED."
show_meta: true
comments: true
header: no
breadcrumb: true
categories: [arduino, photon, iot, maker, javascript, node.js, johnny-five]
permalink: /workshop/js/weather/hello-iot-world/
---
# Table of Contents
*  Auto generated table of contents
{:toc}

In this lab you will use one of the digital output pins to send a signal to an LED. You will use a digital pin that has an LED on the board connected to it (so you will be able to see it turn on and off).

# Write the Code
Since you are using Node.js and Johnny-Five for this lab you can take advantage of the dependency management capabilities that Node.js provides. You need to let the application know that it has a dependency on the Johnny-Five framework as well as the Particle-IO plugin so that when the application is prepared for execution, it can fetch the required dependencies for us. In Node.js this is done with a package.json file. This file provides some basic meta-data about the application, including any dependencies on packages that can be retrieved using NPM (according to [npmjs.com](https://www.npmjs.com){:target="_blank"} today, NPM stands for _Nonometer Process Machine_ and not _Node Package Manager_ like you may have thought).

1. Using your favorite/preferred text/code editor, create a file in your labs folder named __package.json__
2. Add the following code (select the tab for the type of board you are using).

<div id="json-tabs">
  <ul>
    <li><a href="#arduino"><span>Arduino</span></a></li>
    <li><a href="#photon"><span>Photon</span></a></li>
  </ul>
  <div id="arduino">
{% highlight javascript %}
{
  "name": "IoT-Labs",
  "repository": {
    "type": "git",
    "url": "https://github.com/ThingLabsIo/IoTLabs/tree/master/Arduino/Blinky"
  },
  "version": "0.1.2",
  "private":true,
  "description": "Sample app that connects a device to Azure using Node.js",
  "main": "blinky.js",
    "author": "YOUR NAME HERE",
  "license": "MIT",
  "dependencies": {
    "johnny-five": "0.9.19"
  }
}
{% endhighlight %}
  </div>
  <div id="photon">
{% highlight javascript %}
{
  "name": "IoT-Labs",
  "repository": {
    "type": "git",
    "url": "https://github.com/ThingLabsIo/IoTLabs/tree/master/Photon/Blinky"
  },
  "version": "0.1.2",
  "private":true,
  "description": "Sample app that connects a device to Azure using Node.js",
  "main": "blinky.js",
    "author": "YOUR NAME HERE",
  "license": "MIT",
  "dependencies": {
    "johnny-five": "0.9.19",
    "particle-io": "0.12.0"
  }
}
{% endhighlight %}
  </div>
</div>

<script>
$( "#json-tabs" ).tabs();
</script>

With the package.json file created you can use NPM to pull down the necessary Node modules.

1. Open a terminal window (Mac OS X) or Node.js command prompt (Windows) 
2. Execute the following commands (replace _C:\Development\IoTLabs_ with the path that leads to your labs folder):

<pre>
  cd C:\Development\IoTLabs
  npm install
</pre>

Next you will create the application code to make the LED turn on and off.

1. Create another file in the same directory named __blinky.js__.

The first thing you need to do is define the objects you will be working with in the application. The three things that matter are a Johnny-Five framework object, an object to represent the board, and the output pin the LED will be connected to. 

1. Add the following code to the __blinky.js__ file (select the tab for the type of board you are using).

<div id="definitions-tabs">
  <ul>
    <li><a href="#arduino"><span>Arduino</span></a></li>
    <li><a href="#photon"><span>Photon</span></a></li>
  </ul>
  <div id="arduino">
{% highlight javascript %}
// https://github.com/ThingLabsIo/IoTLabs/tree/master/Arduino/Blinky
'use strict';
var five = require ("johnny-five"); 

// Define the pin that is connected to the LED 
var LEDPIN = 13;

// Create a Johnny Five board instance to represent your Arduino.
// Board is simply an abstraction of the physical hardware, whether it is 
// a Photon, Arduino, Raspberry Pi or other boards. 
var board = new five.Board();

/*
// You may optionally specify the port by providing it as a property
// of the options object parameter. * Denotes system specific 
// enumeration value (ie. a number)
// OSX
new five.Board({ port: "/dev/tty.usbmodem****" });
// Linux
new five.Board({ port: "/dev/ttyUSB*" });
// Windows
new five.Board({ port: "COM*" });
*/
{% endhighlight %}
  </div>
  <div id="photon">
In order to complete the next step, you will need the device ID you copied earlier when you were claiming the Photon (or the name/alias you gave the Photon when you updated the firmware to VoodooSpark) and your Particle Cloud access token. To get the access token, open a terminal window (Mac OS X) or command prompt (Windows) and execute the following command (you may be prompted to login or provide your Particle Cloud password again): 

<pre>
  particle token list
</pre>
  
Find the token for <i>user</i> (make sure if you see more than one that you choose the one that is not expired).
  
Optionally you can use a browser to navigate to <a target="_blank" href="https://build.particle.io/">Particle Build</a> and find your Access Token on the setting page (click on the gear icon in the lower-left part of the screen).

{% highlight javascript %}
// https://github.com/ThingLabsIo/IoTLabs/tree/master/Photon/Blinky
'use strict';
var five = require ("johnny-five"); 
var Particle = require("particle-io");

// Set up the access credentials for Particle Build
var token = process.env.PARTICLE_KEY || 'YOUR PARTICLE ACCESS TOKEN HERE'; 
var deviceId = process.env.PHOTON_ID || 'YOUR PARTICLE PHOTON DEVICE ID/ALIAS HERE'; 

// Define the pin that is connected to the LED 
var LEDPIN = 'D7';

// Create a Johnny Five board instance to represent your Particle Photon.
// Board is simply an abstraction of the physical hardware, whether it is 
// a Photon, Arduino, Raspberry Pi or other boards. 
var board = new five.Board({ 
	io: new Particle({ 
		token: token, 
		deviceId: deviceId 
	}) 
});
{% endhighlight %}
  </div>
</div>

<script>
$( "#definitions-tabs" ).tabs();
</script>


In this code you define a few variables that you will be working with:

- <code>five</code> - represents the Johnny Five framework capabilities, which provide a type of object model for working with boards like Arduino and Particle.
- <code>LEDPIN</code> - a variable that references pin D7, which the LED is connected to.
- <code>board</code> - a representation of the physical board you are using.

Now that the objects are created, you can get to the meat of the application. Johnny-Five provides a board 'ready' construct that makes a callback when the board is on, initialized and ready for action. Inside the anonymous callback function is where your application code executes (this function is invoked when the board is ready for use).

Johnny-Five provides a collection of objects that represent the board, the pins on the board, and various types of sensors and devices that could be connected to the board. For this lab you are going to write code that is fairly true to the base Arduino C/sketch or Particle Build programming model (we'll get into the abstractions that Johnny-Five provides you later). This will help you understand some of the basic concepts for how an Arduino or Particle board works.

In the following code you will create a callback function that is invoked when the board is initialized and ready (this is a Johnny-Five concept). 

- You will set digital pin (the <code>LEDPIN</code> variable above) as an output pin (vs. an input pin), meaning the application is expecting to send voltage out from the pin as opposed to read the voltage coming in to the pin.
- You will create a loop that runs once per second.
- Inside that loop you will write out to the pin either LOW or HIGH voltage. Since the pin is a digital pin, its only options are 0 and 1 - in the world of Arduino (and similar) boards that is LOW and HIGH. When you send 0 (or LOW) to the pin, that is equivalent to off (sending no voltage). When you send 1 (or HIGH) to the pin that is equivalent to on (sending full voltage).

1. Add the following code to __blinky.js__

{% highlight javascript %}
// The board.on() executes the anonymous function when the
// board reports back that it is initialized and ready. 
board.on("ready", function() { 
	console.log("Board connected..."); 
    
	// Set the pin you connected to the LED to OUTPUT mode  
	this.pinMode(LEDPIN, five.Pin.OUTPUT); 

	// Create a loop to "flash/blink/strobe" an led  
	var val = 0;
	this.loop( 1000, function() {
		this.digitalWrite(LEDPIN, (val = val ? 0 : 1));
	});
});
{% endhighlight %}
  
As an option to code above, Johnny-Five actually has an object model for an LED and we could also have simply done the following, but one of the goals was for you to see how the __digitalWrite()__ function works before abstracting it away.

{% highlight javascript %}
board.on("ready", function() {
	console.log("Board connected...");

	var led = new five.Led(LEDPIN);
	led.blink(1000);
});
{% endhighlight %}

# Run the App
When you run the application it will execute on your computer, and thanks to Johnny-Five, it will connect with your board and work directly with it. Basically, your computer is acting as a hub and communicating via USB or Wi-Fi with the micro-controller on the prototyping board as one of potentially many devices (or spokes). This application will run on your development machine during this workshop, but it could also be deployed to a small micro-processor device like the Raspberry Pi or Intel Edison.

1. Open a terminal window (Mac OS X) or command prompt (Windows)
2. Execute the following commands (replace c:\Development\IoTLabs with the path that leads to your labs folder).

<pre>
  cd C:\Development\IoTLabs
  node blinky.js
</pre>

You should the indicator LED on the board blink a little as the app is initialized, and then the on-board LED should start blinking in unison at one blink per second.

<blockquote>
  PHOTON USERS - When you power on the Photon and it establishes a Wi-Fi connection, the first thing it does is a 'phone home' to the Particle Cloud where it registers itself as online. When it does that, it also registers its local IP address. When you run the Node.js application, thanks to the Johnny-Five framework and the Particle-IO plugin, the Node app pings the Particle Cloud and requests the IP address for the device name you specified (that is why the Particle Token and Device ID/Alias are needed). Once the application has the local IP address for the Photon, all communications with the device are over local TCP (which is why your development machine and the Photon have to be on the same network). Since the communication from the Node.js app to the Photon is over TCP and not USB, the Photon doesn't need to be plugged into your USB port - it simply needs to be powered on and on the same Wi-Fi you configured it for (and the machine running the Node.js app has to be on the same Wi-Fi network). 
</blockquote>
  
When you want to quit the application, press <kbd>CTRL</kbd> + <kbd>C</kbd> twice to exit the program without closing the window (you may also have to press <kbd>Enter</kbd>). 

<blockquote>
  PHOTON USERS - After stopping the application press the __Reset__ button (the right button) on the Photon to prepare it for the next run.
</blockquote>

# Conclusion &amp; Next Steps
In this lab you learned how to write a Node.js/Johnny-Five application that writes LOW and HIGH signals to a digital pin (designated for output) to make an LED blink. In itself this may not be very exciting, but the core concept is necessary - writing to a digital output pin.

In the [next lab][nextlab] you will set up a Microsoft Azure IoT Hub that will act as the cloud backend for your IoT devices. In the labs after that you will build a new _Thing_ that will collect environment data and send it to your IoT hub.

<a class="radius button small" href="../setup-azure-iot-hub/">Go to 'Setting Up Azure IoT Hub' ›</a>

[nextlab]: ../setup-azure-iot-hub/