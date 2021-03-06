# CamBot!
Arduino & C# Automated desktop webcam robot.

This is a project to make a (somewhat) expressive, desk toy webcam that looks for, follows, and recognizes faces. It's based on [Hackerbox kit #24](https://hackerboxes.com/collections/frontpage/products/hackerbox-0024-vision-quest) with a few added parts where needed for additional functionality.

## Video walkthrough of the hardware build
[![Cambot Video](https://img.youtube.com/vi/nbJNLVBo_g4/0.jpg)](https://www.youtube.com/watch?v=nbJNLVBo_g4)

### Current developement progress:

| Progress        | Description           
| ------------- |:-------------:
| Done! | Initial build. |
| Done! | Write Initial Arduino Serial interface API |
| Done! | Write Initial C# desktop app to prove out tracking and Serial interfacing. |
| Done! | Put together github for everything. |
| Done! | Put together tutorial video on initial build, initial code, key learnings and demonstration. |
| 50% | 2nd pass at Arduino code, clean things up, break them into seperate files, etc. |
| 40% | 2nd pass at C# desktop app, clean things up, break them into seperate classes, etc. |
| 0% | 3d print nicer cover for electronics and paint / finish stand. |

There are 2 sides to the software on this project.
- The Arduino code that creates a Serial Interface to control the robot and other hardware.
- The C# code to create a desktop app to read the webcam, look for interesting activity and control the robot ( via Arduino Serial Interface mentioned above. )

The hardware consists of:
1. An Arduino Nano V3 to control everything
2. A single Neopixel RGB LED
3. 2 MG996R Servos
4. A Pan / Tilt Assembly
5. A laptop webcam assembly + USB adapter
6. A repurposed power adapter from an old cellphone
7. 2 usb cables
8. Scrap floor samples as a weighted base + cardboard for 'feet'
9. A tiny breadboard + prototyping connectors to keep things open to experimentation.

The arduino code is pretty straightforward. There is a set of commands to control every piece of hardware. I generally write the Serial interpretter to take commands in the form of <<COMMAND:PARAMETER>>, that way if we end up with multiple commands in the buffer at a time they can easily be broken apart and dealt with individually. I also like to write this code so that you can define where you want the state of things to be, and how fast you want to move towards those states. That way you can just send one command that says "Hey, I want the light to fade to green slowly" with a few commands and not have to actually send all of the rgb values to animate that yourself.

The C# code is currently using a wrapper around the amazing [Open CV library](https://opencv.org/) for movement detection from the webcam feed. You can find this library [Here](http://www.emgu.com/wiki/index.php/Main_Page). At the moment I am just hacking up the movement detection example to suite my needs, but eventually I will start my own, more organized, app.

I would love for the CamBot to have some personality, it would look for faces and follow them. When it see's people it 'likes' it would light up with a happy color and beep in delight. Something that just sits on the desk and looks adorable. Maybe it would unlock your computer for you, or do other more useful things. Some kind of google home type integration?!

## Before you try to run the C# code!
You will have to be able to run .net 4 code, and you will also have to install the EMGU libs ( linked above ) or even better, use Nuget to pull down the latest EMGU.CV libs. **Warning. This is hacked together code at the moment, jammed sideways into an example from EMGU.**

## Some things to consider ##

Movement detection is VERY sensitive to alot of things. Take care to minimize the amount of noise in your video feed. For example; dark rooms will be a problem if your camera doesn't use infrared. Also cheap cameras have alot of noise in general. Your robot will need to be seated on a totally still surface. ANY unplanned movement will totally 'unsettle' your motion detection ( it will think EVERYTHING is moving until it has time to settle again. ) 

If you go into the Desktop app C# code ( VideoSurveilance.cs ) you will see some settings towards the top of the file that you can play with to fine tune things alittle. I will add more comments for this, but in the meantime I'll describe a few that might be usefull to combat false positives and bad motion detection in general.

MS_PAUSE_DETECT_AFTER_MOVE is the number of milliseconds the desktop software will wait before it moves towards ( what it thinks to be ) movement. The significance of this is the more frames of video the library ( Open CV ) 'stacks' for detecting movement the more the stationary objects will 'settle' out of the image. ( This is why the image will slowly go mostly black over time. Open CV is trying to remove things that are the same between frames leaving only the moving stuff. ) So if you turn this up, it will help remove stationary objects, but it will make your camera less responsive ( as it has to wait before it can move again. )

SUBSTRACTION_HISTORY works in tandem with the above setting. If you give the library more time to settle, you have to increase the number of frames it has in it's buffer to 'stack'. So if you increase MS_PAUSE_DETECT_AFTER_MOVE you should turn this up as well. 

SUBTRACTION_THRESHOLD is something you will just have to play with, but the idea is the higher this number the less the library will assume a difference between frames is actually movement. Turning this up would probably help remove some webcam noise but if it's too high it will make the motion detection alot less usefull.

FRAME_BLUR_STRENGTH would help with webcam noise alot like SUBTRACTION_THRESHOLD. ( the higher the number, the more it smooths or blurs frames. ) Just be careful, this MUST be an odd number and the higher it gets, the less accurate the motion detection will be.
