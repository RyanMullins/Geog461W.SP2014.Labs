Animation and JavaScript
========================

Animation is a well used tool in the web. In class, we have looked at several projects and papers where animation is a key component. Many of you have expressed interest, or even a requirement, for animation in your final projects. This demo gives a bit of background on animation in web pages and how it works. 

## Basics of Animation

Most of the applications we have looked at in class make animation a universal thing. Starting the animation causes all of the UI to update in some way, be it a step through time or a step through some story. To make this happen on a web page you need to have four things:

* Something to keep track of the current time step... a counter
* Something to update the UI... an updater
* Something to tell the UI that it's time to update... a timer
* Some controls to start/stop/progress the animation... buttons

Let's tackle the purely code parts first...

A counter is a variable, typically a Number, used to track progress through an animation. You increase it to go forward and decrease it to go backwards. 

An updater is a function. While this is not a requirement, I like to define an updater function to take in a single parameter, _increment_, that tells it how far forward or backward to jump through the animation. Inside the function, the first thing you do is increment the counter. Then you update the different UI components to show the data for that step in the animation. 

A timer is another variable, created by making a call to the [setTimeout()][timeout] function. The setTimeout() function has two parameters, _function_ and _duration_. The _duration_ parameter allows you to define the frequency that the timer will go off in milliseconds. The _function_ parameter is a reference to the function that should be called every time the timer goes off. The function returns a unique identifier (UID), a Number, for the timer you just created. You should always store the UID for a timer in a variable, otherwise there will be no way to turn it off. Below is a complete example of how to initialize a timer that goes off every half-second and increments updates the UI by one step. 

```JavaScript

var counter = 0;
var timer   = setTimeout(function () { updateUI(1); }, 500);

function updateUI (increment) {
    counter = counter + increment;

    // Update the UI Components...
}

```

## Creating Controls

Now that we have an understanding of the code to create a timer, let's take a look at the controls. 

We are going to make use of [jQuery][jquery] and [jQuery UI][jqueryui] again in this demo (see Lab 3 for a primer of these two libraries). In particular, we are going to borrow the controls from the [Button Toolbar demo][jqueryuibutton].

Much like Lab 3, we make a _div_ element in the _body_ of our page and give that _div_ the _id_ "controls". Inside the controls div we add three _button_ elements, with their _id_ parameters set to "prev", "play", and "next", respectively. 

```HTML

<div id="controls" class="ui-widget-header ui-corner-all">
    <button id="prev">previous</button>
    <button id="play">play</button>
    <button id="next">next</button>
</div>

```

jQuery UI provides a really convenient method, $().**button**(), for controlling the styling of a button. The function takes an optional _options_ parameter. This demo takes advantage of the _text_ and _icon_. The _text_ option is set to false, which basically means that the text in the tag won't be displayed if the icon isn't available. The _icon_ property is actually an object with several properties within it, of which we will set the _primary_ icon to be "ui-icon-seek-prev", "ui-icon-play", and "ui-icon-seek-next", respectively. 

Unfortunately, this kind of code can get a bit long, so the example below only covers the code for the step back button. 

```JavaScript

$("#prev")
    .button({
        text: false,
        icons: {
            primary: "ui-icon-seek-prev"
        }
    })

```

## Linking Controls and Timers

We've got some code and we've got some controls. Now we just need to link to two together. 

You will probably remember the $().**click**() function from Lab 3. We will use this to make the buttons trigger the animation. There are three basic functions that we can define to pass as the parameter to $().click().

First is a supplemental function to stop the timer from going off. This function uses the [clearTimeout()][timeoutstop] function to stop the timeout, sets the timer variable to null, and sets the play button's icon to be the play icon (this button's icon changes, we will discuss it later). The code for this function is shown below.

```JavaScript

function stopAnimation () {
    clearInterval(timer);
    timer = null;
    $("#play").button("option", "icons", {"primary": "ui-icon-play"});
}

```

Second, is the function to simply step the animation in a direction. This is the function you will use for the previous and next buttons. The function first checks to see if the timer is going, and if it is, stops it. Then it increments the counter using the **updateUI**() function. The example below shows the code for the previous button. 

```JavaScript

function () {
    if (timer) { stopAnimation(); }
    updateUI(-1);
}

```

Finally, we have the function to make the play button work. This builds on the last function, but adds a bit of code to initialize the timer. The example code below shows how to initialize the timer in this function, and set the icon of the play button to look like a pause button. 

```JavaScript

function () {
    if (timer) {
        stopAnimation();
    } else {
        timer = setTimeout(function () { updateUI(1); }, 500);
        $("#play").button("option", "icons", {"primary": "ui-icon-pause"});
    }
}

```

## All Together Now

In practice the code would look something like this.

```JavaScript

var counter = 0,
    timer = null;

function updateUI (increment) {
    counter = counter + increment;

    // Update the UI Components...
}

function stopAnimation () {
    clearInterval(timer);
    timer = null;
    $("#play").button("option", "icons", {"primary": "ui-icon-play"});
}

$("#prev")
    .button({
        text: false,
        icons: {
            primary: "ui-icon-seek-prev"
        }
    })
    .click(function () {
        if (timer) { stopAnimation(); }
        updateUI(-1);
    });

$("#play")
    .button({
        text: false,
        icons: {
            primary: "ui-icon-play"
        }
    })
    .click(function () {
        if (timer) {
            stopAnimation();
        } else {
            timer = setTimeout(function () { updateUI(1); }, 500);
            $("#play").button("option", "icons", {"primary": "ui-icon-pause"});
        }
    });

// Code for Next button... 

```

One thing I have not covered is how to handling looping in your animation. This is a slightly difficult task, as it depends greatly on the data you use in you app. Generally, the easiest way to handle this is to create another variable, _maxCounter_, and set that variable's value to the highest allowable time step for the counter. Then you change the code in the **updateUI**() function to account for the looping conditions. A simple example: 

```JavaScript

var counter = 0,
    maxCounter = 10;

function updateUI (increment) {
    if (counter >= maxCounter && increment > 0) {   // Loop to the front if you're at the end
        counter = 0;
    } else if (counter <= 0 && increment < 0) {     // Loop to the end if you're at the front
        counter = maxCounter;
    } else {                                        // Just keep swimming...
        counter = counter + increment;
    }

    // Update the UI Components...
}

```

And that's it. Obviously there are some things I left out, like how to actually update various UI components. But you should have enough information about how animation works to start implementing it in your own application. 

<!-- Links -->

[jquery]: http://jquery.com
[jqueryui]: http://jqueryui.com
[jqueryuibutton]: http://jqueryui.com/button/#toolbar
[timeout]: https://developer.mozilla.org/en-US/docs/Web/API/Window.setTimeout
[timeoutstop]: https://developer.mozilla.org/en-US/docs/Web/API/window.clearTimeout
