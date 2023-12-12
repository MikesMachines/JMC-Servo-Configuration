
# JMC-Servo-Configuration
Information on JMC integrated servos via PC Serial RS232 and ESP32 Modbus for tuning and controls
 
Info here has not been proofed, I just want to get as much down quickly while it's somewhat fresh. Please contact me with any corrections. My response times are slower than average, I definitely appreciate patience. 

Background
This is an aggregation of all the relative information I have found regarding the JMC ISHV integrated servos. My main goal is to be able to use digital values or a PWM signal for position control rather than steps and pulses. As the speeds increase, more pulses need to be calculated and sent. Live changes to acceleration are especially resource intensive. The servo adds possible data sources from the driver for feedback and alarms. 

There is a lot of excellent information out there I never would have been able to figure out. But it has been a lot more work than expected finding all the pieces to put together. It’s been useful to try and sort it all out to make it easier in the future. Hopefully someone else will as well. 

Links

Nearly all of the info here is from one of the links below. There's more to add.

The best source for manufacturer manuals and software also supplies their own custom guides that get into changing the control modes. They sell the entire line of servos and also sell cables to connect the motor with the PC software.
https://www.rocketronics.de

CNC User Blog with a full tear down
https://drmrehorst.blogspot.com/2020/04/ihsv-servomotor-information.html

A video on connecting to the service with a PC. Part 2 gets into specific tuning parameters.
https://youtu.be/_9Q-VFesnA0

Fixes for driver issues with the PL2303 (mostly due to older cables)
https://www.ifamilysoftware.com/Prolific_PL-2303_Code_10_Fix.html

I purchased my motor from the G-Penny store on AliExpress. My motor came with 6.07 firmware. The recommendation is to go with v6 firmware as it has additional parameters available. The motor was on sale and they have answered my questions amazingly fast so.
https://www.aliexpress.com/store/1100702143


Modbus software

List of simulators to validate connections
https://www.dalescott.net/modbus-development/

Python application which connects and configures JMC servos. Of note is the use of a Modbus library for communication. Lot of positive reviews, I haven't tried it yet.
https://github.com/robert-budde/iHSV-Servo-Tool


Tips
Use COM3 - COM8 only. I came across an article for a different manufacturer's servo software that said to not use any COM port number higher than 8. The software simply stops scanning. An unrelated article mentioned possible conflicts with COM1 and COM2.  So be safe and just use COM3 through COM8. Numbering is easy to change in the driver parameters,



Step 1: To connect the PC to the servo, get a USB to RS232 cable with a Prolific PL2303 chipset

PC serial connections can operate at low / CMOS voltage levels. RS232 serial can be as high as 15v, but will work with at least 5v. The cables are doing more than just wiring. I tried two FTDI USB serial programming boards with no success. I have to imagine that some other brands would work, but several sources were very direct about the need to use a cable with the PL2303 chip. I ended up buying two, LED indicators are very useful.  

The pl2303 chip is inside the DB9 end of the USB cables I found, so I couldn’t just cut the end off to wire to the servo. Getting a female DB9 made testing possible by pushing wires in and connecting to the servo. That worked for a couple days until I accidently shorted it out. And now I have a cable with indicator lights 😊


Step 2: Install drivers before connecting cable to PC.

The catch with the PL2303 chip is that it only recently has been included with Windows and there can be problems with various combinations of chip and drivers. Installing the desired driver before connecting the cable will stop Windows from auto installing the latest driver. The 4100 version did not work with my first cable 

Don’t buy older or sketchy PL2303 cables. To reduce the amount of fake chips being sold, Prolific added a check to verify authenticity. If the test fails, the driver ends up not starting. A pirated cable might pass today but fail with a driver update. 


Step 3: Install and run the JMC PC Software

Once the cable and driver are set, the PC Control Software connects reliably and instantly. I used versions 1.7.6, 2.4.4, and 2.4.6.  The 2.4 versions will connect with just the name of the com port and scan the other settings. The motor shipped configured for 57600 / 8 / Even parity / 1 stop bit.

Some parameters are only available by selecting the general AC servo over the integrated line. I cannot say what the risk is by selecting a different line of servos, but all the features seem to work fine in my very limited testing.

Step 4: Validate parameter changes

With the PC software running and servo ready to be run,  send one update at a time and follow the steps to apply. Some changes are made live, some require the servo be offline, some won't take effect until the next power on. The software makes it obvious.

I've done everything up to here several times, but no success with modbus yet. 

Step 5: Use an ESP32 via Modbus or other method to control the motor by serial positioning or other non-pulse mode.
In Progress, 
tried using a UART serial out with a pl2303 chip 
Trying a MAX3232 chip for TTL to RS282. 
Overwhelmed by ModBus overhead. It seems to be very device specific. Each parameter has an address and ID associated with it. Once some connectivity is verified, a header file or key/value set will be made for the parameters of use. 


Step 6: Read the Alarm and configure other outputs as needed. 
What will be the burden on the ESP?


My observation of parameters so far

Rigidity - P01-02 / P01-03
The rigidity settings has a huge impact on the feel the machine. It creates a springy feel on lower settings that has slip and give to it. Too low and the stroke doesn't have enough force to move. High settings stiffen it up until it is so stiff it starts vibrating. This can be changed while running. I can see potential with changing it with the direction of the stroke. Going in loose might be safe or more natural feeling. But the power and force of high settings have their own potential

Location complete P03-05 P03-06
You can actually change when a move is considered complete. The P03-06 setting allows you to provide a range for position deviation. The P03-05 setting determines how that's used - by itself, or only for validation once the move is completed. A third setting allows it to be further refined with filtering. 

Pulses per revolution P03-09
If you want to change the number of pulses that need to be sent to make a complete revolution here, set the DIP switches to 'default' and it will read the value here. Set this to be 0 to use electronic gearing.
