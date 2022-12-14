Wood Pellet Stove with Tuya TYWE1S chip flashed with ESPHome gives you COMPLETE local control over ALL stove functions (and adds some).

<img width="200" alt="image" src="https://user-images.githubusercontent.com/52110065/206867278-c315a1e4-ac1a-437a-a0ab-9b436e2e7e1c.png"><img width="200" alt="image" src="https://user-images.githubusercontent.com/52110065/206867295-c3bf5f91-b33f-4f54-bfee-15c9a0ac25a4.png"><img width="190" alt="image" src="https://user-images.githubusercontent.com/52110065/206867320-d2f332c3-bdea-4863-acfd-e0982a6786ba.png"><img width="200" alt="image" src="https://user-images.githubusercontent.com/52110065/206867354-89dded84-38fb-4934-8f31-ad49d3f32fe0.png">




This project dives deep into the TuyaMCU parts of ESPHome that can be used to control various Tuya cloud based pellet stoves.  While the ESPHome TuyaMCU implementation is mostly robust, getting information on what it does and how it does it and why has been challenging to say the least.  I hope this helps someone else. I now have two of these stoves running ESPHome and FINALLY they function the way I want them to, not how some idiotic low-budget coder tried to program them.  I used to wake up cursing my pellet stoves.  Now I just wake up all warm and fuzzy.

The Discussions section also has LOTs of info: https://github.com/jazzmonger/wood-stove-with-TYWE1S-Tuya-chip/discussions

TuyaMCU Docs: https://esphome.io/components/tuya.html

The Tuya TYWE1S chips and those like it are basically just repackaged ESP8266 modules.  All the functions of the 8266 are all there.  The TuyaMCU libray in ESPHome provides an abstraction layer between the display/control module and Home Assistant. It allows control messages to be both Sent and Received from the device the module is controlling. In this case, our stove's main brain/control unit, or MCU as its called.

I'm an electronic engineer/designer by education and training, not a programmer.  When chips are soldered in place on a commercial circuit board and each pin has a live trace going to and from it, then we, as trained engineers, MUST assume that those pins are being used for some purpose, hence, likely not available for use.  but...

That said, I LOVE hacking this stuff and just because a trace is there doesnt mean the pin cant be used.  Best practice is to ground unused inputs and outputs on all integrated circuit chips, with or without using current limiting resistors.  However, in the case of multilayer PCBs, it is next to impossible for the average person to know what or how to trace those pins to see if they are being used.  But, often they can be repurposed because in applications such as this, the Tuya chip is just there to provide a WiFi function, and the GPIOs are just sitting there idle.  More on this later.

We start with replacing the buggy, error prone Tuya control software by flashing ESPHome onto the display's TYE1S Tuya communications & controller chip.

**Product Models:** 
Cleveland Ironworks Wood Pellet Stove (there are 4 models, all use the same controller).  
https://www.cleveland-ironworks.com/products/pellet-stoves.html
I have a test bench set up with a spare control board. i have 2 models of this stove which both fail, but for different reasons.

It also appears from the Wifi module (N12210) that this exact same controller is used in Nemaxx Pellet Stove Pellet Heater P6 P9 P12.  https://www.ebay.com/itm/193755684864

Masterforge has the exact same models of this stove, all will use the same controller.

Ther are several others sold by Lowes, and other retailers that use the same display control board and TYWE1S chip.

There are a few displays that look slightly different but I believe all have the same basic internals. The displays have exactly the same function, just dufferent color faceplates. if they look like any of these then youre in luck!
All will have a TYWE1S chip that provide wifi functions that can be reprogrammed with ESPHome.

![591DD0B1-FA41-4E61-9443-4FD6ECA933A9](https://user-images.githubusercontent.com/52110065/211733976-057da24a-9941-4c60-8027-599b9a545b8c.jpeg)

<img width="738" alt="image" src="https://user-images.githubusercontent.com/52110065/201829503-b268795e-132c-4b8f-9ec9-96a7dc361b03.png">

They work ok most of the time, but the stove malfunctions fairly frequently.  
Common mafunctions include:
- randomly turning off w/ the msg "Goodbye!" on the display.
- pellets failing to light in the hopper
- not coming back on after power failure
- various cryptic E messages from time to time. E1,E2,ESC1, ESO1,ESC2,ESO2,etc  
- the stove will overheat when set to ECO2 and P1, throwing ESC1 errors. mitigate this with a fan behind the stove, triggered by exhaust temp.  set combustion fan to -10 for P1 in stove settings. 
The fan also fixes E6 error code indicating the temp sensor below the pellet hopper is reading too high

**Problem Definition**
Pellet stoves made by Cleveland Ironworks all use the TuyaMCU chip TYWE1S to communicate with the stove's MCU inside the display unit.  Their shoddy programmer defined multiple datapoints for the same attribute in at least 2 cases, the most important being temperature.  This results in the Tuya integration reporting a random (usually negative) false value for current temp. The other problem is it sends its temp values as degrees F and ALL esphome climate entities originate in C.  If you set your HA instance to imperial, it will automagically convert C to F.  In this case it convering what it thinks is C ( but is really F) into F again.  temp readings are whacked and there was nothing you can do until now.

link to product manual https://www.cleveland-ironworks.com/mwdownloads/download/link/id/2763

Their app is a vendor written Tuya based app that looks like this:

![image](https://user-images.githubusercontent.com/52110065/201826389-4d6983e2-d4e9-4c8c-8e9a-fdad121f7824.png)

simply flashing esphome to the chip gives incorrect temp readings due to the stove using F instead of C for values. a fix for this is coming.  The stove below might work great in purgatory (which, Im sure, is where these stoves were built), but not in the real world...

<img src="https://user-images.githubusercontent.com/52110065/201826977-369853d6-650e-4048-9e9a-6701e3d1621c.png" width="200">

also, the main problem with trying to control the stove by brute force manually is setting your target temp doesn't work bc it fights with the stove's internal "set temp" that is set thru the front panel. when the room temp sensor reaches the user set value it shuts off to stove - a self contained system. there is no manual way to get around it and convert it to a manual operation. too many sensors, blowers, igniters, relays and safety issues to etc to deal with.

**Solution**
- Flash ESPHome to the TWYE1S chip using TuyaMCU. This is the module that relays info to and from the stoves control board and also provides a WiFi interface to the Tuya iOT cloud platform  
- use the new mod I got one of the ESPhome devs to make for me and you're golden.  its defined in the esphome config file.
- if your stove is turning off for no reason w/ "Goodbye" displayed on the screen, then the automations I've provided should help mitigate this

See the Duscussions for specific solutions to some of the errors and problems I've solved over the past year.

Post a message in the discussions if you have comments/questions.  I've tried to be as complete as possible and probably seen every problem these stoves have.
Have fun!

**Data Points:**
```
1 - Power on (Heat)
4 - Mode P1/P2/P3P4
101 - ECO1/ECO2
104 - Error Code
106 - Set desired room Temp
107 - Current Temp
108 - Pipe Temp
109 - Protect Temp
```
Error code - 104 (these come back from the server as numbers and are then mapped to error codes in the vendor app)

Flashing the chip
Remove it from the display module.  It unplugs easily.
![image](https://user-images.githubusercontent.com/52110065/201829197-9dbe42c8-2a3b-4ed6-bd42-652bc7af61cb.png)
![image](https://user-images.githubusercontent.com/52110065/201829293-01f14b40-6578-4f13-be3f-9fe1ad663b63.png)

this link shows you how to do it: https://tasmota.github.io/docs/TuyaMCU-Devices/#costco-charging-essentials

EDIT: for future reference, I used 3.3v, U0TX and U0RX to my flashing interface and only grounded GPIO0 to flash the chip. Also, connect 3.3V (not 5v) to the 3V3 pin on the TYWE1S chip, pin 4.
WARNING - be careful when you solder wires to the board/chip to flash the chip.  I used breadbord wires w/pins and my VCC conneection broke and it took most of the circuit board trace off both the board AND the chip.  I salvaged it with a Frankenstein jumper, but you've been warned!



![image](https://user-images.githubusercontent.com/52110065/201826705-5dca2316-681d-4a11-b2fe-dcdd6da0d05e.png)

![2214C401-0F23-4C0E-A11A-6B35EF960212](https://user-images.githubusercontent.com/52110065/206728504-6cf685a2-e121-4c3a-acbe-583272950f5c.jpeg)

The spiffy new entity with options to change Power and ECO mode. See the file uploads for all the cide to generate these.

special cards used frim HACS
type: custom:slider-entity-row
type: custom:plotly-graph

![FD170D9C-2115-40DC-B2CB-CFCBC79D0ED4](https://user-images.githubusercontent.com/52110065/205919641-3d850cbc-5b52-4826-8e8e-4072a784e0c3.jpeg)

My dev stove Command Module after modifying the MCU and display board for new power level P5
<img width="1365" alt="image" src="https://user-images.githubusercontent.com/52110065/209392856-158ed57f-7128-4268-abff-75737c9c6241.png">


The last piece of the puzzle was to snoop on the uart line going from the display to the MCU control board.  this lets you echo messages from the display to home assistant as there is no other way to do it. See the discussions section on how i achieved this magic!

<img src="https://user-images.githubusercontent.com/52110065/211300934-491579a7-015f-4c4d-9932-c056d6074059.jpeg" width=30% height=30%>   <img src="https://user-images.githubusercontent.com/52110065/211300939-f88f6053-2541-433f-be82-1dfb1940b813.jpeg" width=30% height=30%> 

![20230107_171807](https://user-images.githubusercontent.com/52110065/211175176-b1841d1c-f93c-4ed8-881f-b44f5f96258c.GIF)
