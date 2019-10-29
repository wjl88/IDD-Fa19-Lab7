# Video Doorbell, Lab 7

*A lab report by William J Leon*

### In This Report

1. Upload a video of your version of the camera lab to your lab Github repository
1. As usual, update your class Hub repository to add your [forked IDD-Fa18-Lab7](/FAR-Lab/IDD-Fa18-Lab7) repository.
1. Answer the questions in-line below on your README.md.

## Part A. HelloYou from the Raspberry Pi

**a. Link to a video of your HelloYou sketch running.**

[Hello You and MotionTracked Project](https://youtu.be/jyNu1Knw9kU)

## Part B. Web Camera

**a. Compare `helloYou/server.js` and `IDD-Fa18-Lab7/pictureServer.js`. What elements had to be added or changed to enable the web camera? (Hint: It might be good to know that there is a UNIX command called `diff` that compares files.)**

```
pi@ixe137:~$ diff -d IDD-Fa19-Lab7/pictureServer.js helloYou/server.js 
1,30c1,6
< /*
< server.js
< 
< Authors:David Goedicke (da.goedicke@gmail.com) & Nikolas Martelaro (nmartelaro@gmail.com)
< 
< This code is heavily based on Nikolas Martelaroes interaction-engine code (hence his authorship).
< The  original purpose was:
< This is the server that runs the web application and the serial
< communication with the micro controller. Messaging to the micro controller is done
< using serial. Messaging to the webapp is done using WebSocket.
< 
< //-- Additions:
< This was extended by adding webcam functionality that takes images remotely.
< 
< Usage: node server.js SERIAL_PORT (Ex: node server.js /dev/ttyUSB0)
< 
< Notes: You will need to specify what port you would like the webapp to be
< served from. You will also need to include the serial port address as a command
< line input.
< */
< 
< var express = require('express'); // web server application
< var app = express(); // webapp
< var http = require('http').Server(app); // connects http library to server
< var io = require('socket.io')(http); // connect websocket library to server
< var serverPort = 8000;
< var SerialPort = require('serialport'); // serial library
< var Readline = SerialPort.parsers.Readline; // read serial data as lines
< //-- Addition:
< var NodeWebcam = require( "node-webcam" );// load the webcam module
---
> const express = require('express'); // web server application
> const http = require('http');       // http basics
> const app = express();				// instantiate express server
> const server = http.Server(app);	// connects http library to server
> const io = require('socket.io')(server);	// connect websocket library to server	// serial library
> const serverPort = 8000;            // port (ixe##.local:PORT)
32c8,9
< //---------------------- WEBAPP SERVER SETUP ---------------------------------//
---
> const SerialPort = require('serialport')
> const Readline = require('@serialport/parser-readline')
34c11
< app.use(express.static('public')); // find pages in public directory
---
> app.use(express.static('public'));	// find pages in public directory
36,40c13,17
< // check to make sure that the user provides the serial port for the Arduino
< // when running the server
< if (!process.argv[2]) {
<   console.error('Usage: node ' + process.argv[1] + ' SERIAL_PORT');
<   process.exit(1);
---
> // check to make sure that the user calls the serial port for the arduino when
> // running the server
> if(!process.argv[2]) {
>     console.error('Usage: node '+process.argv[1]+' SERIAL_PORT');
>     process.exit(1);
43,77c20,21
< // start the server and say what port it is on
< http.listen(serverPort, function() {
<   console.log('listening on *:%s', serverPort);
< });
< //----------------------------------------------------------------------------//
< 
< //--Additions:
< //----------------------------WEBCAM SETUP------------------------------------//
< //Default options
< var opts = { //These Options define how the webcam is operated.
<     //Picture related
<     width: 1280, //size
<     height: 720,
<     quality: 100,
<     //Delay to take shot
<     delay: 0,
<     //Save shots in memory
<     saveShots: true,
<     // [jpeg, png] support varies
<     // Webcam.OutputTypes
<     output: "jpeg",
<     //Which camera to use
<     //Use Webcam.list() for results
<     //false for default device
<     device: false,
<     // [location, buffer, base64]
<     // Webcam.CallbackReturnTypes
<     callbackReturn: "location",
<     //Logging
<     verbose: false
< };
< var Webcam = NodeWebcam.create( opts ); //starting up the webcam
< //----------------------------------------------------------------------------//
< 
< 
---
> // initialize the serial port based on the user input
> const port = new SerialPort(process.argv[2])
79,84c23,26
< //---------------------- SERIAL COMMUNICATION (Arduino) ----------------------//
< // start the serial port connection and read on newlines
< const serial = new SerialPort(process.argv[2], {});
< const parser = new Readline({
<   delimiter: '\r\n'
< });
---
> // create a parser so that we can easily handle the incoming data by reading the line
> const parser = port.pipe(new Readline({
>     delimiter: '\r\n'
> }))
86,92d27
< // Read data that is available on the serial port and send it to the websocket
< serial.pipe(parser);
< parser.on('data', function(data) {
<   console.log('Data:', data);
<   io.emit('server-msg', data);
< });
< //----------------------------------------------------------------------------//
95d29
< //---------------------- WEBSOCKET COMMUNICATION (web browser)----------------//
98,105c32,33
< io.on('connect', function(socket) {
<   console.log('a user connected');
< 
<   // if you get the 'ledON' msg, send an 'H' to the Arduino
<   socket.on('ledON', function() {
<     console.log('ledON');
<     serial.write('H');
<   });
---
> io.on('connect', function (socket) {
>     console.log('a user connected');
107,111c35,39
<   // if you get the 'ledOFF' msg, send an 'L' to the Arduino
<   socket.on('ledOFF', function() {
<     console.log('ledOFF');
<     serial.write('L');
<   });
---
>     // if you get the 'ledON' msg, send an 'H' to the arduino
>     socket.on('ledON', function () {
>         console.log('ledON');
>         port.write('H');
>     });
113,118c41,45
<   //-- Addition: This function is called when the client clicks on the `Take a picture` button.
<   socket.on('takePicture', function() {
<     /// First, we create a name for the new picture.
<     /// The .replace() function removes all special characters from the date.
<     /// This way we can use it as the filename.
<     var imageName = new Date().toString().replace(/[&\/\\#,+()$~%.'":*?<>{}\s-]/g, '');
---
>     // if you get the 'ledOFF' msg, send an 'L' to the arduino
>     socket.on('ledOFF', function () {
>         console.log('ledOFF');
>         port.write('L');
>     });
120c47,51
<     console.log('making a making a picture at'+ imageName); // Second, the name is logged to the console.
---
>     // if you get the 'disconnect' message, say the user disconnected
>     socket.on('disconnect', function () {
>         console.log('user disconnected');
>     });
> });
122,126c53,60
<     //Third, the picture is  taken and saved to the `public/`` folder
<     NodeWebcam.capture('public/'+imageName, opts, function( err, data ) {
<     io.emit('newPicture',(imageName+'.jpg')); ///Lastly, the new name is send to the client web browser.
<     /// The browser will take this new name and load the picture from the public folder.
<   });
---
> // this is the serial port event handler
> // note that we are using the parser
> // read the serial data coming from arduino - you must use 'data' as the first argument
> // and send it off to the client using a socket message
> parser.on('data', function(data) {
>     console.log('data:', data);
>     io.emit('server-msg', data);
> })
128,134c62,65
<   });
<   // if you get the 'disconnect' message, say the user disconnected
<   socket.on('disconnect', function() {
<     console.log('user disconnected');
<   });
< });
< //----------------------------------------------------------------------------//
---
> // start the server and say what port it is on
> server.listen(serverPort, function () {
>     console.log('listening on *:%s', serverPort);
> });
\ No newline at end of file
```


**b. Include a video of your working video doorbell**

>> Video Recorded

## Part C. Make it your own

**a. Find, install, and try out a node-based library and try to incorporate into your lab. Document your successes and failures (totally okay!) for your writeup. This will help others in class figure out cool new tools and capabilities.**

Used Motion to monitor the Raspberry Pi bench in the MakerLAB. The webserver shows a live view of the bench. When a motion event is detected it calls sendEmail.py, a script to automatically ping my email alerting me of the intrustion.

**b. Upload a video of your working modified project**

[Hello You and Modified Project](https://youtu.be/jyNu1Knw9kU)
