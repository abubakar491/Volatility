# Volatility - CS50 Final Project
## Introduction
Computers dyanamically adjust the brightness of their screens, adapting to the ambient light of their surroundings.  For instance, when in a bright room or in direct sunlight, the screen brightens, and when in a dim room, the screen dims.  In essence, computers are capable of adapting to their surroundings, causing their users to rarely have to adjust settings.  However, this is not the case with volume, even though almost every computer has a microphone that could analyze ambient background noise.  Instead, users need to constantly adjust their volume settings to fit the current environment they are in. 

Votalitity is a Chrome extension that solves this problem by dynamically adjusting the volume of YouTube videos and other audio playing in Chrome by analyzing a user's ambient surroundings.  When a user is in a noisy environment, such as a bustling Starbucks, Volatility automatically increases the playback volume of audio in Chrome.  When a user returns to a quieter environment, Volatility then reduces the playback volume to the user's desired level.  The algorithm takes into account several factors, including sensitivity and minimum, maximum, and desired volume levels, to best predict desired output levels in the current environment.  
## How To Use
Once the Chrome extension is installed, you need to open the accompanying web page for Volatility to work.  Click on the extension icon in the extension toolbar and then select open.  This opens the control page, which allows you to adjust preferences and see a visual of the volume level.  Due to audio permission complications, this page must be open whenever you want Volatility to dynamically adjust the volume. 
## Features
### Focus Mode
Focus Mode is the main mode of this extension.  In this mode, Volatility dynamically adjusts the volume based upon ambient background noise, meaning that your volume will be higher in noisy environments and lower in quieter environments.
### Talking Mode
In talking mode, Volatility will decrease your volume whenever it detects talking.  This is great for when you are in a room listening to music with a few friends and want the music to turn down when you are trying to talk to each other.  When no speech is detected, the volume is set to the desired volume setting.  When speech is detected, the volume level is lowered to the minimum volume level setting.
### Sensitivity
The sensitivity preference determines how drastically the volume will change based upon ambient noise.
### Minimum, Maximum, Desired Volume
These preferences are used by the algorithm along with microphone input to determine the desired volume output level.  The output level will never go below the minimum level, never go above the maximum level, and will be centered at the desired level.
## Technical Description
There are two main components of Volatility: the web page and the Chrome
extension. There were many reasons we chose to split this project into these two
components. First, we were unable to access the computer microphone via just an
extension (Chrome does not allow for extensions to access their Chrome.audio
developer API). We considered making a Chrome application (different from an
extension) that could access user audio, but we decided against it, largely
because Chrome is discontinuing their web app store in 2017. Moreover, we
couldn’t use only a web page, because a single website could not access all the
tabs in a user’s browser window. Thus, the solution was to use a web page to get
microphone input and a chrome extension to access all the tabs. The application
works by getting microphone input via webpage, calculating new output volume
according to the input volume and the settings selected, and sending the new
volume to the chrome extension’s background script, which then sends a message
to all audible tabs to adjust the sounds of the Youtube videos in open tabs.

### Pop-up

We included a pop-up for our chrome extension icon, as seen in popup.html. Using
jQuery and javascript in extension.js, we added event listeners to buttons on
the pop-up so that they open the Volatility page and turn the extension on/off
when they are clicked.  In order to turn the extension on/off, the popup must
send a chrome message to the Volatility tab.

### Getting microphone input via webpage

Microphone input was received in main.js. Users can choose to be in focus mode
or talking mode. In focus mode, when there’s more ambient noise, the output
volume gets louder. In talking mode, the output volume is at a desired volume
when it does not detect speech and goes down to the minimum volume when it
detects speech. We used different methods of getting microphone input for the
two modes. This is because we needed to use different API’s for the modes’
separate purposes. For focus mode, we needed to analyze amplitude. For talking
mode, we needed to recognize speech.

#### Focus Mode
To get user input in focus mode, we used the GetUserMedia function, a part of
the WebRTC API
(link: https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API).
Specifically, we began by looking for examples of GetUserMedia that took in
microphone input on a webpage, and found this repository:
https://github.com/webrtc/samples/tree/gh-pages/src/content/getusermedia/volume,
a simple program that displayed the volume of microphone input detected on a
webpage. This is what we based our application off of. Most critical to our app
was the soundmeter.js class declared in this repo. Built off of GetUserMedia,
the Javascript file defines a “SoundMeter” object. When a SoundMeter is created,
it requests microphone access on the webpage and calculates the amplitude of
audio input every 50 milliseconds. Every SoundMeter has two attributes.
“Instant”, which is the audio amplitude at any given instant, and “slow,” which
calculates the average amplitude over a given, specified, number of seconds.

We calculate the “slow” average volume by creating a queue. Every time a new 50
millisecond interval passes, we add the new amplitude into the queue, and remove
the oldest value. We then calculate and return the average of each element.

#### Talking Mode
To get user input in talking mode, we used the Web Speech API (link:
https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API). We used a
“webkitSpeechRecognition” object. This object takes in microphone input and
performs speech recognition on the audio. We get the audio continuously, despite
the 60 second time limit specified by the API, by continuously stopping and
starting the speech recognition.

### Calculating and updating the new volume

Most of the application settings were adjusted in main.js. In order to access
the values that users input on the Volatility page, we downloaded and
implemented JQuery as taught in class. Inside of main.js, we implemented the
following: max/min volume settings, focus/talking mode, calibration, and
sensitivity.

#### Focus Mode
The new output volume is determined based on the user’s calibration values. At
calibration, we associate their specified desired volume and the average input
noise at the time. This average input volume is calculated in SoundMeter, which
is given to main as value from 0.0 - 1.0 (as described above). That is, we
increment the desired volume by how much the average input noise
increases/decreases relative to the average input noise at calibration. This
increment is scaled by the sensitivity the user specifies. The larger the
sensitivity, the more the volume changes when the input noise changes. The newly
calculated output volume is then sent as a chrome message to the Volatility
page, and the Volatility page (index.html) is updated to display this volume.

In implementing the minimum and maximum volumes, there is an if statement that
runs every 50 milliseconds when output volume is adjusted. It checks if the
output volume is less than the minimum volume variable, or if it is greater than
the maximum volume variable. If the output volume is greater than max, it
changes the volume to be max. Likewise, if it is less than the minimum variable,
the output volume is reset to the min variable value.

#### Talking Mode
In talking mode, only the minimum volume and desired volume during calibration
matters. We created a “webkitSpeechRecognition” object and added event listeners
to it. The first event listener is for when a word is recognized in speech. When
more than five words are recognized, we send the minimum volume to the chrome
extension and update the new volume on the webpage (index.html). We do this by
creating a count and only updating the new volume when this count reaches the
desired word minimum, which we default to 5. The second event listener is for
when speech has ended. In this case, it updates the volume to the user’s
specified desired volume on the web page and sends this value to the chrome
extension. Then, we make sure the speech recognition is reset, so it doesn’t end
after speech has ended.

When the user switches from focus to talking mode, the interval for the focus
mode’s microphone input is cleared, and speech recognition begins. When the user
switches from talking to focus mode, speech recognition is stopped and the
stream goes back to the focus mode’s microphone input.

### Adjusting video volumes of all tabs

In background.js, we receive messages from the Volatility page. Then, we compose
new messages with the received volume as an attribute, query all audible tags in
chrome, and send the new message to them. In content.js, these messages are
received and adjust the volume of all the tabs by adjusting all audio tags.

Manifest.json is a file that any chrome extension requires, which details its
title, description, background scripts, content scripts, and browser actions.

### Design

To stylize our website and popup, we used bootstrap and modified other css
files. These files interacted with the html to make the website user-friendly
and nicely designed. We got the icon from freepik and the website background
from Tech&All (link:
http://techandall.com/5-white-studio-backgrounds-for-your-product-display/). We
made the website and popup minimalistic and easy to use so that users can focus
on enjoying the product’s functionality, which is an automatic volume adjuster
for their audio.
## Screenshots
Extension Popup:

![extension popup](https://github.com/dsmith3197/Volatility/blob/master/screenshots/Screen%20Shot%202017-07-24%20at%206.49.46%20PM.png?raw=true "Extension Popup")


Accompanying Webpage:

![accompanying webpage](https://github.com/dsmith3197/Volatility/blob/master/screenshots/Screen%20Shot%202017-07-24%20at%206.53.12%20PM.png?raw=true "Accompanying Webpage")

## Known Problems
-This Chrome extension works best with headphones.  Because it uses the microphone to analyze ambient noise, music or other noise coming from your computer's speakers will artificially inflate the output levels.

