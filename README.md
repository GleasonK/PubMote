###Turn Your Smartphone into a Wiimote Using Polymer:

With all this technology around us, it's surprising that most devices do not interact with each other. Take your smartphone and your laptop for example, they probably sit less than a foot apart on your desk, but they rarely talk. I am going to show you how to use PubNub and Polymer to bridge that gap by turning your smartphone into a [Material Design][MaterialDesign] gamepad.

The [live demo of PubMote][PubMote] can be seen here. Navigate to the [controller][PMController] on your smartphone and to one of the [demos][PubMote] on your desktop. In this blog post I will show you how to create your PubMote and how to use it for HTML5 games. Your gamepad can have any design you want, with any features. Trackpad, buttons, accelerometer, anything goes. The PubMote I cover in this guide will have four arrow keys, a bar that tilts with the accelerometer, and three action buttons. If you need a reference at any time, use the [full code](https://github.com/GleasonK/PubMote/blob/master/controller_lite.html).

<img src="https://raw.githubusercontent.com/GleasonK/PubMote/master/img/controller.png" width="250" title="The PubMote controller from the live demo." alt="PubMote">

####Prerequisites 
- Basic knowledge of Polymer 
- Basic understanding of [Bower](https://www.polymer-project.org/0.5/docs/start/getting-the-code.html) package management tools ro install and manage dependencies of Polymer component files is helpful, but I will walk you through it

### Polymer and Material Design

[Polymer] is provides an interesting way to go about web development. It is build on top of a set of new W3C platform called [Web Components](http://www.w3.org/wiki/WebComponents/). This standard allows you to use both premade and custom blocks of HTML to build a larger webapp. 

Before one of these building block elements can be used, first you must import it. A typical use will look as follows:

```
The import:
<link rel="import" href="paper-fab"> 
... 
The use:
<paper-fab icon="send"></paper-fab>
```

Polymer comes with many premade elements, all of which follow the [Material Design][MaterialDesign] standards that were created for Android 5.0 Lollipop. The material design elements that come with Polymer are called _Paper Elements_, and they are responsive, adapting across different screen sizes and devices.

### 1. Creating Your Material Design Gamepad

Let's begin designing and developing our gamepad. First, we need to install Polymer so we can use its Material Design UI components.

#### 1.1 Install Polymer
The first step to creating your gamepad is [Installing Polymer](https://www.polymer-project.org/0.5/docs/start/getting-the-code.html) in your project. The easiest to do this is using Bower. To see other possibilities, check the [Polymer documentation](https://www.polymer-project.org/0.5/docs/start/getting-the-code.html).

	$ cd <Project-Root>
	$ bower init
	$ bower install --save Polymer/polymer#^0.5

Running those commands should result in the following file structure:

<img src="https://raw.githubusercontent.com/GleasonK/PubMote/master/img/install-polymer.png" width="700" title="File structure after a bower install of Polymer." alt="bower-install">

To get started using Polymer elements, include `webcomponents.min.js` in the `<head>` tag of your `index.html`.

```
<!DOCTYPE html> 
<html> 
	<head> 
		<script src="bower_components/webcomponentsjs/webcomponents.min.js"></script> 
	</head> 
<body> 
...
```


#### 1.2 Import UI Compnenets

The PubMote UI is created solely using [Polymer Core and Paper elements](https://www.polymer-project.org/0.5/docs/start/usingelements.html).

Elements Used:
1. paper-button, a responsive and elegant button UI
2. paper-fab, a Material Design action button
3. core-icons, used for gamepad icons
4. hardware-icons, also used for gamepad icons

    bower install Polymer/core-icon
    bower install Polymer/core-icons
    bower install Polymer/core-iconset
    bower install Polymer/core-iconset-svg
    bower install Polymer/paper-fab#^0.5
    bower install Polymer/paper-button#^0.5	

This will install all necessary components, for our Polymer project. Now, before elements can be used they must be imported in the `<head>` tag.

```
 <script src="bower_components/webcomponentsjs/webcomponents.min.js"></script>

<link rel="import" href="bower_components/paper-button/paper-button.html">
<link rel="import" href="bower_components/paper-fab/paper-fab.html">
<link rel="import" href="bower_components/core-icons/core-icons.html">
<link rel="import" href="bower_components/core-icons/hardware-icons.html">
```

Now you can use all those Polymer elements you just downloaded and imported.

#### 1.3 Basic Gamepad UI

First, we will make our body tag to house all Polymer elements. Inside that, we use `<paper-shadow>` which we will later style to have the look of a Material Design [Card](http://www.google.com/design/spec/components/cards.html). Inside that we create arrow keys using a `<paper-button>	` and a `<core-icon>`. I have an HTML5 `<canvas>` that we will use to later show accelerometer tilt. Our action buttons are `<paper-fab>` with `<core-icon>` icons.

<img src="https://raw.githubusercontent.com/GleasonK/PubMote/master/img/controller_labeled.png" width="250" title="Labeled Polymer PubMote" alt="PubMote_labeled">

```
<body fullbreed unresolved>	
	<template is="auto-binding" id="controller-template">
		<paper-shadow id="gamepad" z="1" class="card">
			<template if="{{isConfigured}}" id=channel>
				<p style="text-align: center;"><i>Channel: {{channel}}</i></p>
			</template>
		
			<div id="arrow-btns"> 
		    	<paper-button data-key="UP" on-tap="{{handleButton}}">
					<core-icon data-key="UP" icon="hardware:keyboard-arrow-up"></core-icon>
				</paper-button> <br>

				
				<!--... Left and Right Buttons are the same.-->

				<paper-button data-key="DOWN" on-tap="{{handleButton}}">
					<core-icon data-key="DOWN" icon="hardware:keyboard-arrow-down"></core-icon>
				</paper-button> <br>
				<img src="controller_logo.png" style="width:100px; margin: 50px;">	
			</div>
			
			<div id="action-btns">
				<paper-fab icon="close" data-key="X" on-tap="{{handleButton}}"></paper-fab>
				<paper-fab icon="radio-button-off" data-key="O" on-tap="{{handleButton}}"></paper-fab>
				<br>
				<paper-fab mini icon="polymer" style="background:black" on-tap="{{changeChannel}}"></paper-fab>
			</div>
		</paper-shadow>
		...
	</template>
</body>
```

The `fullbreed` attribute causes the body to fill the viewport, and the `unresolved` attribute will allow all Polymer elements to be loaded before being displayed, avoiding any flash of unstyled content issues (FUOC). Note that there is an `<img>` for your gamepad's logo, make it your own!

The `on-tap={{exp}}` syntax is an event handler (on-tap), and an expression (exp). Values in `{{}}` can be bound to data in a template. Therefore, we wrap our controller in a `<template is="auto-binding">` tag. To set the values or expressions contained in the curly braces, we must first get the template in javascript, and attach the values to it.

```
var controller = document.querySelector('#controller-template');
controller.isFullscreen = false;
controller.handleButton = function(e) { ... };
...
```

The `<template if="{{isConfigured}}">` tag is a child of the main template, and it will only display when the value of isConfigured is true. We will change this value when the user types in a channel to join. The last potentially unfamiliar bit in this HTML is `data-key="<KEY>"`. HTML5 allows you to attach custom data to any tags. We will access this data later.

Finally, to give this the look of a floating material design card, a little styling was used on the elements, you can [see the CSS here](https://gist.github.com/GleasonK/cf3b70692c3628b32d31#file-pubmote_style-css). Elements also have attributes that you can find in the [documentation](https://www.polymer-project.org/0.5/docs/elements/paper-fab.html). I used `<paper-fab mini>` to create the smaller botom button. Basic CSS can also be used on an element's tag. For example, if you want to change the color of your `<paper-fab>` buttons you could do the following:

```
paper-fab {
	background:red;
}
```

### 2. Implementing Our Gamepad

#### 2.1 Installing and Importing PubNub

PubNub has a Polymer element pre-made. The `<pubnub-element>` does not have any UI, but it can be used to publish messages from our gamepad. It requires your own API keys, so to continue, you need to [sign up for a PubNub account](http://www.pubnub.com/get-started/) to get your publish and subscribe keys. Your publish/subscribe keys are in the [Developer’s Admin Dashboard](https://admin.pubnub.com/).

Once you have completed that, `<pubnub-element>` can be downloaded using Bower.

	$ bower install --save pubnub-element
	
To import this element in your `index.html`, use the following line:

	<link rel="import" href="bower_components/pubnub-polymer/pubnub-element.html">

#### 2.2 Using the PubNub Element

You can initialize the PubNub client using the `<core-pubnub>` element with your keys. Inside the `<core-pubnub>` we will use a `<core-pubnub-publish>` element to send keypresses from our gamepad. 

_Note: this element should be the last tag in your `<template>` from earlier._

```
<template>
...
	<core-pubnub 
		publish_key="your_pub_key" 
		subscribe_key="your_sub_key">
		<core-pubnub-publish id="pub" channel="{{channel}}" message="Hello"> 
		</core-pubnub-publish>
	</core-pubnub>
</template>
```

`core-pubnub-publish>` is used to send messages to all subscribers of a channel. Also, since we use `{{channel}}` means that our template will need an attribute `channel`. Then, to publish a message, we need to access the `<core-pubnub-publish>` element through out template

```
var controller = document.querySelector('#controller-template');
...
controller.channel="demo";
controller.$.pub.message = {'type':'button', 'data':'UP'};
controller.$.pub.publish();
```
#### 2.3 Implement Your Gamepad

All of our gamepad will be implemented through the parent `<template>`. Looking back through the code you will notice that so far the attributes that the template needs are:
- isConfigured
- handleButton
- changeChannel
- channel

We can begin my implementing those as follows:

```
var controller = document.querySelector('#controller-template');
controller.isConfigured = false;
controller.channel="demo";
// Pulls the data-key attribute from the HTML and then publishes a button press.
controller.handleButton = function(e) {
	this.$.pub.message = {"type":"button", "data":e.target.getAttribute('data-key') };
	this.$.pub.publish();
};
// Prompt for the channel to join, then update the controllers channel.
controller.changeChannel = function(e){
	this.isConfigured = true;
	var channel = prompt("Enter a channel: ");
	if (channel == "" || channel == null) channel="demo";
	this.channel = channel;
}
...
```

Take a deep breath. Currently, all of our gamepad buttons are functional. If I press the `<paper-button data-key="UP" on-tap="{{handleButton}}">` button, it will call the template's handleButton function. That function pulls the data-key field ("UP"), and then publishes the value to the current channel. 

<img src="https://github.com/GleasonK/PubMote/raw/master/img/Nintendo_Gameboy.jpg" width="250" alt="Gameboy">

Welcome to 1989, we now have a functional Gameboy! We can do better than that though. The final step for us to catch the 2006 Wii is tracking the smartphone's accelerometer values so we can add tilts and shakes to our games.

First, add the accelerometer fields to your controller.

```
controller.aX=0;     // Used for tilts
controller.aY=0;
controller.aZ=0;
controller.speed=0;  // Used for shakes
```

Now, we will implement the ready function for our `<template>`. I'm sure most of us know the jQuery $(document).ready() function. Polymer's ready is similar. When any `<polymer-element>` is loaded and ready to be rendered, it will then call its own `ready` function. Since we want to wait for our controller to be loaded before attempting it to run any code, we will begin to implement `ready` as follows:

```
controller.ready = function(){
	var iOS = /(iPad|iPhone|iPod)/g.test( navigator.userAgent );
	var lastUpdate = Date.now();
	var SHAKE_THRESHOLD = 1250;
	...
}
```

This may come as a shock, but (from my tests so far) iOS and Android implemented their accelerometer in opposite directions. A left tilt on android produces the value of a right tilt on iOS. To make these values uniform, we store the boolean `iOS` which tells whether the controller is an apple product. The variable `lastUpdate` will be used to set a time threshold, so we arent publishing a value every millisecond, that would be unnecessary and probably slow our game down. You may have to play around with the value of `SHAKE_THRESHOLD` a litte bit, depending on what you consider a shake. We will get there though.

Not for the fun part. We only want out gamepad to register an accelerometer if the device viewing the page has one.

```
if (window.DeviceMotionEvent == undefined) {
    // No accelerometer is present. Use buttons. 
    alert("no accelerometer");
}
else {
	// Hooray, we have an accelerometer! Attach a decivemotion listener.
    alert("accelerometer found");
    window.addEventListener("devicemotion", accelerometerUpdate, true);
}    
```

If an accelerometer is found, we attach a `devicemotion` listener to our `window`. The second parameter, `accelerometerUpdate` is a function that will be called whenever device motion is seen.

```
function accelerometerUpdate(e) {
    if (Date.now() - lastUpdate > 100) {  // Only checks the accelerometer value ever 100ms
	    lastUpdate = timeNow;
		var aX = event.accelerationIncludingGravity.x*1;  // Used for tilt
		var aY = event.accelerationIncludingGravity.y*1; 
		var aZ = event.accelerationIncludingGravity.z*1;
		
		// Since iOS flips the value, this will flip them to an Android standard
		aX = iOS ? -1*aX : aX;
		
		// Calculate the change in X, Y, and Z. Used to check if there was a shake.
		var dX = Math.abs(aX - controller.aX);
		var dY = Math.abs(aY - controller.aY); 
		var dZ = Math.abs(aZ - controller.aZ);

		// Only change if the dX was large enough, avoids noise. Publish if there was a change.
		if (dX > 0.45) { 
			controller.aX = Math.floor(aX*2)/2.0;  // Floor to the nearest half.
			controller.$.pub.message = {'type':'aX', 'data':controller.aX};
			controller.$.pub.publish();
		}
		if (dY > 0.45) controller.aY = Math.floor(aY*2)/2.0;
		if (dZ > 0.45) controller.aZ = Math.floor(aZ*2)/2.0;
		
		// Calculate the controller's total speed. This will tell whether there was a shake.
		controller.speed = Math.abs(aX + aY + aZ - controller.aX - controller.aY - controller.aZ)/ timeDiff * 10000;
		
		// Play around with SHAKE_THRESHOLD. If speed is larger than ST, publish a shake.
		if (controller.speed > SHAKE_THRESHOLD) { 
			controller.$.pub.message = {'type':'shake', 'data':controller.speed};
			controller.$.pub.publish();
		}
	}
}
```

Read the code comments to see what is happening at each step, but for the most part this code just checks and publishes the aX tilt and device shakes. If you are holding the phone in portrait mode, `aX` is the tilt is left and right. A shake is based off how large the change in X,Y,Z coordinates was over the span of 100ms, a large change probably means a shake.

![Accelerometer](https://raw.githubusercontent.com/GleasonK/PubMote/master/img/coordinates.png)

Congratulations, you made your very own PubMote! Now we can go use it in some javascript games.

### 3. Using your PubMote

I am going to show you how to use your gamepad with a javascript game that uses the keyboard. To see an example of using the accelerometer, see [this code with comments](https://github.com/GleasonK/PubMote/blob/master/demos/wiinub.html).

<img src="https://raw.githubusercontent.com/GleasonK/PubMote/master/img/PubMan.png" width="250" alt="Pac-Man">

I found this [javascript version of Pac-Man](https://github.com/GleasonK/PubMote/blob/master/demos/pacman/pacman.js) open source on github. It uses the N key to start the game and the arrow keys to control Pac-Man. To use the PubMote, I simply created a function in Pac-Man's `index.html` to simulate keypresses.

```
// Simulates clicking a key
function fireKey(el, key) {
    if (document.createEventObject) {
        var eventObj = document.createEventObject();
        eventObj.keyCode = key;
        el.fireEvent("onkeydown", eventObj);   
    } else if (document.createEvent) {
        var eventObj = document.createEvent("Events");
        eventObj.initEvent("keydown", true, true);
        eventObj.which = key;
        eventObj.keyCode = key;
        el.dispatchEvent(eventObj);
    }
} 
```

Then, in your PubNub subscribe function, handle the controller button presses as keyboard events.

```
pubnub.subscribe({                                      
	channel : channelName,
	message : function(message){
		if (message.type == "button") {
			switch(message.data){
				case "UP":
					fireKey(el, 38);	// Up Arrow Key
					break;
				case "DOWN":
					fireKey(el, 40);	// Down Arrow Key
					break;
				case "LEFT":
					fireKey(el, 37);	// Left Arrow Key
					break;
				case "RIGHT":
					fireKey(el, 39);	// Right Arrow Key
					break;
				case "X":
					fireKey(el, 78);	// N Key
					break;
				case "O":
					fireKey(el, 78);	// N Key
					break;
				default:
					break;
			}
		}
	}
});
``` 

That's it. Now you can use your PubMote with JavaScript games!

[MaterialDesign]:http://www.google.com/design/spec/material-design/introduction.html

[PubMote]:http://kevingleason.me/PubMote/

[PMController]:http://kevingleason.me/PubMote/controller.html

[Polymer]:https://www.polymer-project.org/0.5/

[PolymerIcons]:https://www.polymer-project.org/0.5/components/core-icons/demo.html

[AndroidPhoneGap]:http://www.pubnub.com/blog/how-to-convert-your-javascript-app-into-an-android-app-with-phonegap/

[iOSPhoneGap]:http://www.pubnub.com/blog/converting-your-javascript-app-to-an-ios-app-w-phonegap/