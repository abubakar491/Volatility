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
#### Screenshots
## Known Problems
-This Chrome extension works best with headphones.  Because it uses the microphone to analyze ambient noise, music or other noise coming from your computer's speakers will artificially inflate the output levels.
