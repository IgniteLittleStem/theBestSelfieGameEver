
All credit goes to Sean M. Tracey
@seanmtracey
Thank you for inspiring me at the works you do at IBM London Meetup



## Prerequisites

To follow this workshop, you will need:

1. Node.js / npm installed `>= 7.0.0`
2. Your favourite IDE
3. A modern web browser (Chrome/Firefox/Safari/Samsung Internet)

Though not strictly necessary to have all of the above to follow the workshop, you will struggle to run the code examples without them.

## Getting started
**NB**: _These following steps are for individuals running macOS/Linux systems. If you're running a Windows device, you may need to adjust some of the CLI commands to better suit your OS._

### Creating an Express.js server to run our code from

1. Either clone or download this repo to your system `https://github.com/IgniteLittleStem/theBestSelfieGameEver.git`
2. Using your CLI (such as terminal, or iTerm) enter the newly downloaded repo with `cd https://github.com/IgniteLittleStem/theBestSelfieGameEver.git/scaffold`. This is the directory that we'll be working from for the remainder of the workshop. It has all of the code that we'll need to deliver the web components that we'll be writing code for in the next little while. In essence, directory is a very simple Express.js web server that uses the Handlebars templating system to deliver the HTML/CSS/JavaScript that  we'll construct (including the required polyfills to help with cross-browser compatibility).
3. Install the dependencies for our Express.js server with `npm i`. This may take a few moments, depending on your speed of your internet connection.


### Creating the `<component-webcam>` tag

Part of the promise of web components is that commonly-used code patterns and structures that are strewn/reimplemented across the web will have a nice, standardised way to write said functionality for easy reuse across sites, services, and apps.

That can sometimes mean writing a great deal of code once so that we never have to worry about it again.

So far, the web-components we've put together have been simple example to demonstrate the techniques, technologies, and methodologies behind creating such things, but for our final component we're going to create a node that can interact with, and capture images with our web cam.

Done well, this normally takes up a few hundred lines of code to ensure cross-browser compatibility (and we're going to write most of that code in this next part of the tutorial, although we won't be examining the bits of the code that handle the webcam itself. We'll only focus on the bits of the code that interact with the web component technologies), but once the component is created, the most anybody should ever have to do is write `<component-webcam>` and a lovely, response, interact UI should appear that **_just gets the job done_**.

1. Just as with the previous components, we need to create a new file to put our web component in. Create a new file called `webcam.hbs` in the same directory that we created `blink.hbs` and `spoilers.hbs` in and open it for editing.

2. We're not going to add any markup in addition to the `<component-webcam>` page this time as we're not affecting content with our nodes. Instead, we're encapsulating complex functionality within a single node. Copy and paste the following into `webcam.hbs`:

```HTML
<h1>Webcam</h1>
<component-webcam></component-webcam>

<!-- SNIPPET THREE -->

<!-- SNIPPET ONE -->

<!-- SNIPPET TWO -->

```

3. And time for another `<template>` node, except this one has _wwwwaaaaaayyyyyy_ more stuff in it. Paste the following code just after the `<!-- SNIPPET ONE -->` line.

```HTML
<template id="component-webcam-template">

    <style>
        :host {
            all: initial;
        }

        main {
            display: flex;
            flex-direction: column;
            width: 300px;
            font-family: sans-serif;
            overflow: hidden;
            justify-content : center;
            align-items: center;
            min-height: 300px;
        }

        main[data-state="inactive"] #preview{
            display: none;
        }

        main #activate, main #preview{
            border: 1px solid #c7c7c7;
        }

        main #activate{
            width: 100%;
            max-width: 300px;
            height: 300px;
            background: #d6d6d6;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
        }

        main #activate p, main #previw p {
            width: 100%;
            text-align: center;
            color: white;
            background: #9c9c9c;
            padding: 1em 0;
            text-shadow: 0 1px 1px black;
            font-weight: 400;
        }

        main[data-state="active"] #activate {
            display: none;
        }

        main #preview{
            display: flex;
            align-items: center;
            min-height: 300px;
            background: #e8e7e7;
            /* padding: 1em; */
            box-sizing: border-box;
            flex-direction: column;
        }

        main #preview video{
            position: fixed;
            left: 100%;
            top: 100%;
        }

        main #preview canvas{
            width: 300px;
        }

        main #preview .controls{
            margin: 1em;
            box-sizing: border-box;
        }

        main #preview button{
            background: #3d70b2;
            border: 1px solid transparent;
            color: white;
            padding: 1em;
            box-sizing: border-box;
            cursor: pointer;
        }

    </style>

    <main data-state="inactive" data-type="still" data-capturing="false">

        <div id="activate">
            <p>click to use camera</p>
        </div>

        <div id="preview">

            <canvas> </canvas>
            <video autoplay playsinline muted> </video>

            <div class="controls still">
                <button id="captureStill">Take Picture</button>
            </div>

        </div>


    </main>

</template>

```

4. Just as before, we're going to create a `<script>` tag to encapsulate our component functionality. On the line just after `<!-- SNIPPET TWO -->` (don't worry, we'll come back to `<!-- SNIPPET THREE -->` later) copy and paste:

```HTML
<script>

    (function(){

        const templateElement = document.body.querySelector('#component-webcam-template');

        class Webcam extends HTMLElement {

            constructor() {

                super();

                const domNode = this;

                domNode.attachShadow({mode: 'open'});
                domNode.shadowRoot.appendChild(document.importNode(templateElement.content, true));

                // SNIPPET FOUR



            }

        }

        window.customElements.define('component-webcam', Webcam);

    }());

</script>
```

Just as before, we're creating a reference to our `<template>` node so that when it comes to populating our shadow DOM, we have a nice easy reference to work with. We've also created a shadow DOM and attached it to our `<component-webcam>` element, which we then populated with the content of our `<template>` node. We've also added the code to register our custom component with the DOM (the line the begins with `window.customElements`).

Now we're going to write some business logic for our web component. This has nothing to so with web components themselves, the code we're about to write (well, copy/paste) is just like the code that you'd normally write to interact with a webcam in a browser, except this time it's working with elements within the scope of the shadow DOM. The elements and code within will not interact with anything in the global scope _except_ the APIs that we'll be using to access the camera.

5. Copy and paste the following on the line just after `// SNIPPET FOUR`:

```JavaScript
const requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;
let activated = false;

const main = domNode.shadowRoot.querySelector('main');
const activate = domNode.shadowRoot.querySelector('#activate');
const video = domNode.shadowRoot.querySelector('video');
const canvas = domNode.shadowRoot.querySelector('canvas');
const ctx = canvas.getContext('2d');

const stillControls = main.querySelector('.controls.still');
const stillCapture = stillControls.querySelector('#captureStill');

// SNIPPET FIVE

```

6. Now we'll add the functions that our component will use to interact with the camera, and that will handled certain elements being clicked (the "activate" button, and the "capture image" button). Again, _none of the code has anything to do with web components_, it just handles how the elements contained within the web component look and behave. Copy and paste the following on the line just after `// SNIPPET FIVE`:

```JavaScript
function drawVideoToCanvas(){
    ctx.drawImage(video, 0, 0);
    requestAnimationFrame(drawVideoToCanvas);
}

function captureImage(){

    const base64 = canvas.toDataURL('image/png');
    const imageData = base64.split(',')[1];

    dispatchEvent('imageavailable', base64);

}

function dispatchEvent(name, data){
    const event = new CustomEvent(name, {
        bubbles: true,
        detail: data,
        composed : true
    });

    domNode.dispatchEvent(event);
}

activate.addEventListener('click', function(){

    if(activated){
        return;
    } else {
        activated = true;
    }

    activate.querySelector('p').textContent = 'attempting to access camera';

    const constraints = {
        video : { facingMode: 'environment' },
        audio : false
    };

    navigator.mediaDevices.getUserMedia(constraints)
        .then(function(stream) {

            const externalStream = stream.clone();

            video.addEventListener('canplay', function(){

                this.play();

                if(main.dataset.state === 'inactive'){
                    main.dataset.state = 'active';
                }

                canvas.width = video.offsetWidth;
                canvas.height = video.offsetHeight;

            });

            try{
                const vidURL = window.URL.createObjectURL(stream);
                video.src = vidURL;
            } catch(err){

                video.srcObject = stream;

                setTimeout(function(){
                    if(video.paused){
                        video.play();
                    }
                }, 1000);

            }

            stillCapture.addEventListener('click', captureImage, false);

            drawVideoToCanvas();

        })
        .catch(function(err) {

            console.log(err);
            dispatchEvent('error', err);

        })
    ;

}, false);

this.addEventListener('cheese', function(){

    if(activated){
        captureImage();
    }

}, false);
```

7. And that's it! If you save the `webcam.hbs` file, restart the Express.js server and head to `http://localhost:3000/component/webcam`, you should see something like the following:

![The functioning <component-webcam> element](images/webcam_first_run.png)

Do as it says! Click to use the camera! You'll be asked if you want to enable to camera, after saying "yes", you should see something a bit like...

![The author of the document, but in a webcam image](images/webcam_first_enabled.png)

_...but with your face, obviously_ 😜

but you may have noticed something: pushing the "Take Picture" button doesn't do anything... or at least, it certainly seems like it isn't doing anything.

Within the code of the web component there's a `dispatchEvent` function. This function enables the custom element to dispatch a custom event to anything listening, and in fact, it already does so when the "Take Picture" button is pressed. It emits an `imageavailable` event with captured image included.

So, while we could have baked in some very specific functionality into our web component to maybe send the image off to a server, or save it to disc, the point of a web component is that it's _reusable_ as much as possible.

By emitting an event with information whenever the "Take Picture" button is pressed, the application developer can decide whether or not they want to do something with that image without ever having to worry about changing the code of the component itself.

So, let's do something with the captured images!

8. Remember when we promised we'd pay attention to `<!--SNIPPET THREE-->` earlier on? Well, now it's time for snippet three to shine! We're going to create a simple `<div>` that will contain any and all images taken by the webcam, and we're going to write the JavaScript that will listen for the `imageavailable` event on any and all `<component-webcam>` elements in our page. Copy and paste the following code just after ✨`<!--SNIPPET THREE-->`✨

```HTML

<div id="imageHolder"></div>

<script>

    const webcamElements = Array.from( document.body.querySelectorAll('component-webcam') );
    const imageHolder = document.querySelector('#imageHolder');

    webcamElements.forEach(element => {

        element.addEventListener('imageavailable', function(e){

            const img = new Image();

            img.onload = function(){
                imageHolder.appendChild(this);
            }

            img.src = e.detail;

        }, false);

    });

    setTimeout(function(){
        webcamElements[0].dispatchEvent( new CustomEvent('cheese') );
    }, 5000);

</script>
```

9. Save, restart, reload the page. You now have a web component based photo booth at your disposal. Neat, huh?

