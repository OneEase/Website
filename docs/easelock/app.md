# EaseLock Software Tutorial

## Note Before Reading The Tutorial

This tutorial shows the softeware part of EaseLock. I assume you have finished seting up the
hardware part of EaseLock during my last tutorial in [Hardware Tutorial](/easelock/), as to ensure
we are in the same context even though the design article has been devided.

The tutorial has been revised several times to improve the quality of EaseLock design.

## Conduct A Meticulous Analysis of The Code

Let's conduct a meticulous analysis of the code, breaking it down into individual steps to
elucidate the functionality of this straightforward program. The code in question is a modified
version of Elliot Williams' (author of the aforementioned AVR Programming book) work, which can be
accessed on his GitHub repository:
[hexagon5un/AVR-Programming](https://github.com/hexagon5un/AVR-Programming/blob/master/Chapter05_Serial-IO/serialLoopback/serialLoopback.c), specifically in the serialLoopback.c
file.

### Obtain the Complete Project in a Single Zip File

You can download the entire sample project, neatly packaged in a single zip file (ActivateDoor.zip) located
at the download page. This zip file also includes a makefile for the project, which allows you
to build the project by simply typing "c:>make" in the command line, provided your GNU C toolchain
is set up correctly (which is the case if you've installed WinAVR). This will generate the
activateDoor.hex file, which is the file that gets uploaded to the ATMega328P chip.

### Header Files

The first four lines of code reference the libraries used by this program. The ones enclosed in
brackets are part of the AVR-C standard library or the C standard libraries, and are included in
your WinAVR installation and the GNU C toolchain.

The other two files, pinDefines.h and USART.h, are additional libraries for this program, obtained
from Elliot Williams' GitHub repository. These libraries provide functionality for reading and
writing to the serial pins (Tx and Rx) on the ATMega328P.

Using the makefile (also provided by Elliot Williams) simplifies the process, as typing
"```c:\ActivateDoor>make flash```" will instruct the GCC compiler to follow the make script, build the
executable (.hex file), and flash it into the chip's memory in one step.

The first task the code performs inside the ```main()``` function is to set up a variable to hold a
character that will be read from the RXD pin, which is the value sent to us via Bluetooth.

### Configure Output Pin

The line "```DDRB |= 0b0000001;```" prepares the PORT B pins for output by setting the first bit of PORT B
to a voltage, indicating to the chip that the pin is available for writing to. This line uses a bit
operator to set the first bit of PORT B, which corresponds to PB0 (pin number 14 in the diagram).

### Initialize Serial Pins

We also need to prepare the serial pins for sending and receiving, which is conveniently handled by
the code in the usart.h library and the implementation of ```initUSART()``` in the usart.c file, included
in the project zip. We don't need to delve into the details, as this work is already done for us.

Before we dive into the infinite while loop, let's take a moment to appreciate the fact that the
chip we're using is a fully functional computer.

## AVR Chip: A Self-Contained Computer

When you connect the ATMega328P chip's VCC pin (pin 7 in the diagram) to 5V and the ground pin (pin
8) to ground, the chip will automatically start executing the program stored in its memory. This is
true even if no other components are connected to the chip. However, if there is no program stored
in the chip's memory, or if there are no output devices connected to the chip, you won't be able to
observe its operation.

### As Long as Power is Available, the Program Runs

The key point here is that as long as the chip has power, it will continue to execute the program.
This is why we use an infinite loop construct, such as the following:

``` while (1) { // perform some action... } ```

This creates a program loop that will continue to run as long as the chip has power.

The first line within our loop is:

```serialCharacter = receiveByte(); ```

This line calls a pre-written USART function to retrieve any incoming byte (character) and store it
in our char variable.

The next line is somewhat unusual:

```transmitByte(serialCharacter); ```

We then transmit the received character back out using another library function. I did this to
verify that I could see the characters I typed echoed back to me on the terminal screen during
program testing.

### The Core Functionality

Finally, we arrive at the interesting code that performs the actual functionality. This is the core
of our entire system and enables communication with the Android program. (Later, we'll explore how
the Android program sends character values to our Bluetooth module.)

When a character is received, we check its value. If it's the letter Y, we apply voltage to PB0 (Pin
14), and if it's an N, we remove voltage from PB0.

```
if (serialCharacter == 'y')
 {
     PORTB = 0b00000001;  // set voltage high (5v) on PB0
 }
 if (serialCharacter == 'n')
 {
     PORTB = 0b00000000;  // set voltage low (less than 5v (0v)) on PB0
 }

```
## PB0 (Pin 14) Directly Controls the Relay Input Port

Now that we've examined the code, the entire system's operation becomes clear. Since PB0 is directly
connected by wire to the input port on our relay, it controls when the relay switches on and off.

As a result, when the relay switches, the output circuit is activated, and the switch on the ODM
(overhead door motor) is closed, ultimately activating the motor.


## The Missing Piece: The Android App

Now, the only remaining component is the Android app, which enables us to pair with our Bluetooth
module and send values to set PB0 high and low, thereby activating and deactivating the switch.

To develop the Bluetooth Android app, I relied heavily on the official Google documentation at:
[Bluetooth | Android
Developers](https://developer.android.com/guide/topics/connectivity/bluetooth.html).

You can download the Android app at the download page and build it using Android Studio.

Note that the app only requests permissions to control
and set up Bluetooth, so it's not overly intrusive.

## Android App: The Initial Unrefined Version

Instead of polishing the app's UI for this tutorial, I've decided to share the rough version with
you, providing a closer look at the development process.

The app's interface appears as follows:

The app loads a list of paired Bluetooth devices at the top.

To connect to your Bluetooth module after powering it up, simply tap the device in the top list.

When you select the device, the app sends test data to the Bluetooth device and displays a message
to indicate whether the connection was successful.

The app prints a status message in the bottom scrollable list, regardless of whether the connection
succeeds or fails.

### Semi-Manual Operation

If the initial connection is successful, click the [Yes] button to send the "y" character, which
activates the relay. However, you'll need to manually deactivate the relay, so after clicking the
[Yes] button, click the [No] button.

Each time you click a button, a new status message is written to the bottom scrollable list,
confirming that the app has taken the desired action.

## Android Development Challenge: No Bluetooth Emulation

One of the significant challenges of developing the Bluetooth app is the lack of Bluetooth emulation
on Android emulated devices. This means that the only way to test the app is to deploy it to a
physical device, run the app, and observe the results. This is why I included numerous status
messages written to the scrollable list view.

When you launch the app, it will appear as follows: (Note that this screenshot was taken on an
emulator, but imagine each test item representing a Bluetooth device)

![btfinder1](/static/015.png)

One of those items will likely be named HC-05 (our Bluetooth module).

Next, you would simply tap the item in the top list, and the app will attempt to connect to the
device and send test data.

On the emulator, it will fail, but at least it provides some output, giving you an idea of what's
happening.

![btfinder2](/static/016.png)

If you run the app on a real device and it connects successfully, it will appear as follows:

![btfinder3](/static/017.png)

Finally, after establishing a successful connection, clicking the [Yes] button will set the PB0 pin
high (to 5V) and activate the relay.

To keep things concise, here is the code in the ConnectThread.java class that writes the "y" and "n"
characters:


```
public void writeYes() {
    try {
        byte [] outByte = new byte[]{121};

        mmOutStream.write(outByte);
        logViewAdapter.add("Success; Wrote YES!");
        logViewAdapter.notifyDataSetChanged();

    } catch (IOException e) { }
}

public void writeNo() {
    try {
        byte [] outByte = new byte[]{110};
        mmOutStream.write(outByte);
        logViewAdapter.add("Success; Wrote NO");
        logViewAdapter.notifyDataSetChanged();

        mmOutStream.write(outByte);
    } catch (IOException e) { }
}
```
As you can see, in the ```writeYes()``` method, we set outByte to the ASCII value 121, which represents
the lowercase letter "y". Similarly, in the ```writeNo()``` method, we set outByte to 110, which
corresponds to the lowercase letter "n". It's a straightforward process.

Additionally, the app writes a brief message to inform you that it has written bytes to the stream,
which is then transmitted over the Bluetooth signal.
