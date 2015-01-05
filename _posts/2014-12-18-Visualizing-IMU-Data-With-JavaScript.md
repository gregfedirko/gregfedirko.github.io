---
layout: post
title: Visualizing IMU Data With JavaScript
---

Let's face it, Drones are everywhere.  And if you love to code, you have probably thought at least once about building a quadcopter of your own.  Unfortunately, this tutorial doesn't explain how to do that.  

But it will show you how to build a fun project that utilizes a quadcopter's most fundamental component, an IMU.  We are going to use an IMU to control a virtual cube *in the browser.*

*Note: This tutorial assumes some basic knowledge of Arduino, JavaScript, and Node.js*

* [Introduction to Arduino](http://arduino.cc/en/guide/introduction)
* [JavaScript (Codecademy)](http://www.codecademy.com/en/tracks/javascript)
* [Beginner Node.js Tutorial](https://www.youtube.com/watch?v=ndKRjmA6WNA)

### What is an IMU?

> "An inertial measurement unit, or IMU, is an electronic device that measures and reports a craft's velocity, orientation, and gravitational forces, using a combination of accelerometers and gyroscopes, sometimes also magnetometers."
>
>--[wikipedia](http://en.wikipedia.org/wiki/Inertial_measurement_unit)

The IMU used in this tutorial is called the [MPU6050](http://playground.arduino.cc/Main/MPU-6050). It is a very popular unit and can be purchased off of Amazon or Ebay for under $10.



The library we will use to communicate to the chip is the [I2C lib](https://github.com/jrowberg/i2cdevlib/tree/master/Arduino/MPU6050) created by Jeff Rowberg.  This tutorial is based off of his excellent [teapot demo](https://github.com/jrowberg/i2cdevlib/tree/master/Arduino/MPU6050/Examples/MPU6050_DMP6).  We will be using the exact same arduino sketch, but will be writing our own JavaScript application to render the data.

###Required Materials:
* Arduino
* MPU6050 IMU
* Bread board
* Jumper wires

If you don't already own an arduino I recommend picking up a starter kit, that will include an Arduino as well as a some starting components to get you going with your initial projects.



## Wiring The Circuit

Before writing any code, we need to wire up the IMU to the Arduino.  This is pretty straight foreward and does not require any resistors, just connect the appropriate pins.



![Diagram]({{ site.baseurl }}/images/mpu6050.png)



After everything is hooked up, we will need some code.  This tutorial uses the sketch from Jeff Rowberg un-modified -- [grab it from GitHub here](https://github.com/jrowberg/i2cdevlib/blob/master/Arduino/MPU6050/Examples/MPU6050_DMP6/MPU6050_DMP6.ino).

Basically, this code utilizes the I2Cdev library to communicate with the sensor.  There are several options for sending data, but the default is yaw, pitch, and roll in Euler angles.  In the main loop this code block is executed because the variable, ```OUTPUT_READABLE_YAWPITCHROLL``` is definied, to change the output simply comment in a different variable.  We will come back to this later.

{% highlight C++ %}
#define OUTPUT_READABLE_YAWPITCHROLL
{% endhighlight %}

{% highlight C++ %}
#ifdef OUTPUT_READABLE_YAWPITCHROLL
  // display Euler angles in degrees
  mpu.dmpGetQuaternion(&q, fifoBuffer);
  mpu.dmpGetGravity(&gravity, &q);
  mpu.dmpGetYawPitchRoll(ypr, &q, &gravity);
  Serial.print("ypr\t");
  Serial.print(ypr[0] * 180/M_PI);
  Serial.print("\t");
  Serial.print(ypr[1] * 180/M_PI);
  Serial.print("\t");
  Serial.println(ypr[2] * 180/M_PI);
#endif
{% endhighlight %}

Upload the code to Arduino and open up the serial monitor. If everything is working correctly, you should see an output like this:

![Arduino Serial Monitor]({{ site.baseurl }}/images/serial-monitor.png)


Now it's time to write some javascript.

## Recieving Data in Node.js

At this point we are successfully recieving imu data on our machine. In this section we will write some code to pick it up in Node.  

the first module we will need is [serialport](https://www.npmjs.com/package/serialport)

```npm install serialport```

In a new file, server.js:

{% highlight javascript %}
var serialPortModule = require('serialport');
var SerialPort = serialPortModule.SerialPort;

var serialPortId = "/dev/tty.usbmodem1421";

var sp = new SerialPort(serialPortId, {
  baudrate: 115200,
  parser: serialPortModule.parsers.readline("\n")
});

sp.on("open", function() {
  console.log("serialport opened");

  sp.on("data", function(data) {
    console.log(data);
  });

});
{% endhighlight %}



All this code does is connect, listen for a data event, and then log the data to the console.

*Note: ```serialPortId``` can be found in the Arduino IDE under ``` Tools > Serial Port```*

####Run the app:

* Plug in the Arduino
* Open the serial monitor
* Send a Key
* In terminal execute ```node server.js```


Your terminal should now be printing out IMU data, feel free to move around the chip and watch the yaw, pitch, and roll values change!

![Terminal Data]({{ site.baseurl }}/images/terminal.jpg)


## Sending the Data to the Browser

Ultimately, we will be rendering this data in the browser but before that can happen, we will need to write a server to ship that data to the browser.

For this we will be using Express.js and Socket.io

###Express & Socket.io

```npm install express socket.io```


The code now looks like this:

{% highlight javascript %}
var express = require('express');
var serialPortModule = require('serialport');
var SerialPort = serialPortModule.SerialPort;

var app = express();
var http = require('http').Server(app);
var io = require('socket.io')(http);

io.on('connection', function(socket){
  console.log('Socket.io connected');
});


app.get('/', function(req, res) {
  res.sendfile('index.html');
});

var serialPortId = "/dev/tty.usbmodem1421";

var sp = new SerialPort(serialPortId, {
  baudrate: 115200,
  parser: serialPortModule.parsers.readline("\n")
});

sp.on("open", function() {
  console.log("serialport opened");


  sp.on("data", function(data) {
    io.emit('data', {data: data});
  });

});

http.listen(3000, function(){
  console.log('listening on *:3000');
});

{% endhighlight %}


the express server will serve up our index.html file, so lets create that now.  For simplicity, the scripts will be written in-line.

{% highlight html %}
<!doctype html>
<html>
<body>
  <style type="text/css">
    body {
      background-color: black;
    }
  </style>
  <script src="/socket.io/socket.io.js"></script>
  <script type="text/javascript">
    var socket = io();
    socket.on('data', function(data){
      console.log(data);
    });
  </script>
</body>
</html>
{% endhighlight %}


Now when our server receives data over the serial port, we tell socket.io to emit that data to the browser.

In our client script we listen for that event, and log the data to the console.

####Run the app:
* Plug in the Arduino
* Open the serial monitor
* Send a Key
* In terminal execute ```node server.js```
* Navigate to localhost:3000
* Open up the JavaScript console!

##Three.js Magic

Now the fun part, some Three.js magic.  

The script in our client is going to get a bit long, so for brevity I am only going to discuss the essential parts.  Take a look at the Github repo for the finished code.

Now that our string of data is in the browser, lets write a function to get the actual numerical values.

{% highlight javascript %}
function getEulerNumbers(eulerString) {
  var stringArray = eulerString.split("\t");
  var numberArray;
  for (var i = 0; i < stringArray.length; i++) {
    numberArray.push(Number(stringArray[i]);
  };
  return numberArray;
}
{% endhighlight %}

This data can now be used in Three.js.  If you recall from earlier, the teapot demo sketch outputs euler angles to represent yaw pitch and roll.

We will be instantiating a Three.js cube, which has methods for rotation which accept euler numbers as arguments, which is perfect for us!

Now lets write a function that will rotate our cube:
{% highlight javascript %}
function rotateCube(cube, yaw, pitch, roll) {
  cube.rotation.x = roll * (Math.PI/180);
  cube.rotation.y = yaw * (Math.PI/180);
  cube.rotation.z = pitch * (Math.PI/180);
}
{% endhighlight %}

Check out the source code for the complete index.html file on [GitHub](https://github.com/gregfedirko/node-imu-cube)

####Run the app:
* Plug in the Arduino
* Open the serial monitor
* Send a Key
* In terminal execute ```node server.js```
* Navigate to localhost:3000

And thats all there is to it! we can now control the orientation of this cube using the MPU6050 module.


![Cube]({{ site.baseurl }}/images/cube.gif)

[GitHub Repo](https://github.com/gregfedirko/node-imu-cube)
