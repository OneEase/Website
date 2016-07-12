# EaseLock Software Tutorial

## Note Before Reading The Tutorial

This tutorial shows the softeware part of EaseLock. I assume you have finished seting up the
hardware part of EaseLock during my last tutorial in [Hardware Tutorial](/easelock/hw), as to ensure
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

## Stop Others From Opening My Door

Currently, the system operates in an ideal environment, but the lack of authentication mechanisms
makes it vulnerable to attacks. Unauthorized individuals can easily open the door without
permission, compromising security and privacy.

To prevent unauthorized access to my door, it is essential to implement a robust authentication
process that verifies the legitimacy of the user before allowing interaction with the door. The
proposed solution involves adding a password authentication step before sending and executing door
commands.

### Dynamic Password Generation and Modification

To bring an additional layer of security to the system, the password can be generated dynamically or
modified by legitimate users. This ensures that the password is not static and reduces the risk of
unauthorized access.

### Authentication Procedure

The authentication procedure involves the following steps:

* Android App Connected to ATMega328P via Bluetooth: The Android app establishes a Bluetooth
connection with the ATMega328P microcontroller.
* Android App Sends Authentication Message to Prove Itself: The Android app sends an authentication
message to the ATMega328P to prove its legitimacy.
* ATMega328P Verifies the Authentication Message and Authorizes the Connection: The ATMega328P
verifies the authentication message and authorizes the connection if the message is valid.
* Android App Sends Door Command Message or Requests to Change Password: Once the connection is
authorized, the Android app can send door command messages or request to change the password.
* Connection Closed: After the door command is executed or the password is changed, the connection is
closed.

### Challenges in Implementing the Authentication Procedure

Although the procedure seems straightforward, there are several challenges to overcome:

* Data Transmission Limitations: The size of the data transmission in one round can vary from 1 byte
to multiple bytes (streams), and its size is not fixed.
* Message Type Identification and Categorization: There are multiple message types involved, including
authentication messages, change password messages, and door command messages, which need to be
identified and categorized.

### Communication Convention

To achieve the authentication procedure in code, a communication convention needs to be negotiated
for both sides. The proposed convention involves using:

* 1 byte to identify the message type: The first byte of the message is used to identify the
    message type (e.g., authentication, change password, door command).
* 2 bytes to identify the message size: The next two bytes are used to specify the size of the
    message payload.
* Payload in message size: The remaining bytes are used to carry the payload of the message.
    Code for Communication Convention

 Here is an example code snippet in android app demonstrates the communication convention:

```
public class Message {
    public byte msgType;
    public short msgSize;
    public byte[] payload;

    public Message(byte msgType, short msgSize, byte[] payload) {
        this.msgType = msgType;
        this.msgSize = msgSize;
        this.payload = payload;
    }

    public static final byte MSG_TYPE_AUTH = 0x01;
    public static final byte MSG_TYPE_CHANGE_PASSWORD = 0x02;
    public static final byte MSG_TYPE_DOOR_COMMAND = 0x03;

	public byte[] pack() {
		// Convert the message to a byte array
		byte[] messageBytes = new byte[3 + msgSize];
		messageBytes[0] = msgType;
		messageBytes[1] = (byte) (msgSize >> 8);
		messageBytes[2] = (byte) (msgSize & 0xFF);
		System.arraycopy(payload, 0, messageBytes, 3, payload.length);	
		return messageBytes;
	}

    public static Message createAuthMessage(byte[] payload) {
        return new Message(MSG_TYPE_AUTH, (short) payload.length, payload);
    }

    public static Message createChangePasswordMessage(byte[] payload) {
        return new Message(MSG_TYPE_CHANGE_PASSWORD, (short) payload.length, payload);
    }

    public static Message createDoorCommandMessage(byte[] payload) {
        return new Message(MSG_TYPE_DOOR_COMMAND, (short) payload.length, payload);
    }
}
```

### Find Storage To Store Crendentials

The ATMega328P microcontroller has a limited amount of storage capacity, which includes:

* Flash Memory: 32 KB of flash memory for storing the program code.
* SRAM (Static Random Access Memory): 2 KB of SRAM for storing data temporarily while the program is running.
* EEPROM (Electrically Erasable Programmable Read-Only Memory): 1 KB of EEPROM for storing data that needs to be retained even when the power is turned off.

Considering the storage limitations of the ATMega328P, we can store the passwords in the EEPROM, which provides a non-volatile storage solution. This means that the passwords will be retained even when the power is turned off. However, the EEPROM has a limited number of write cycles (approximately 100,000), which may not be suitable for frequent password changes.

We'll use the EEPROM library, which is part of the Arduino framework, to interact with the EEPROM.

#### Password Storage Structure

To store the password, we'll use a simple structure:

```
typedef struct {
  char password[16]; // 16 characters max
  uint8_t passwordLength; // length of the password
} password_t;
```
#### Storing the Password

To store the password, we'll use the EEPROM.write() function:

```
void storePassword(char* password) {
  password_t pwd;
  strncpy(pwd.password, password, 16);
  pwd.passwordLength = strlen(password);

  EEPROM.write(0, pwd); // store at address 0
}
```
#### Retrieving the Password

To retrieve the password, we'll use the EEPROM.read() function:

```
char* retrievePassword() {
  password_t pwd;
  EEPROM.read(0, pwd); // read from address 0

  char* password = (char*)malloc(pwd.passwordLength + 1);
  strncpy(password, pwd.password, pwd.passwordLength);
  password[pwd.passwordLength] = '\0'; // null-terminate

  return password;
}
```

### Preventing Replay Attack

A replay attack occurs when an attacker intercepts and records a valid authentication message, and then replays it to gain unauthorized access to the system. To prevent replay attacks, we need to ensure that each authentication message is unique and cannot be reused.

Why preventing replay attack is necessary? Replay attacks can be devastating to our door control system. If an attacker can replay a valid authentication message, they can gain unauthorized access to the system and open the door without permission. This can compromise the security and privacy of the system.

#### Challenge-Response Method

One way to prevent replay attacks is to use a challenge-response method. In this method, the ATMega328P microcontroller generates a random challenge and sends it to the Android app. The Android app then responds with a response message that includes the challenge and a hash of the password. The ATMega328P microcontroller verifies the response message by checking the challenge and the hash of the password.

To implement the challenge-response method, we need to add a new message type to the communication convention. Let's call it ```MSG_TYPE_CHALLENGE_RESPONSE```.

Firstly, the Android App sends a ```MSG_TYPE_AUTH``` message to the device and
nn the board side, we can generate a random challenge and send it to the Android app.
Further, the Android App sends a ```MSG_TYPE_CHALLENGE_RESPONSE``` message to the device to verify.

On the board side, we can generate a random challenge using current timestamp and send it to the Android app.

```
byte storedChallenge[4];// 4-byte challenge

void sendChallenge() {
    unsigned long timestamp = millis(); // get the current timestamp
    storedChallenge[0] = (byte) (timestamp >> 24);
    storedChallenge[1] = (byte) (timestamp >> 16);
    storedChallenge[2] = (byte) (timestamp >> 8);
    storedChallenge[3] = (byte) timestamp;

    Message challengeMessage;
    challengeMessage.type = MSG_TYPE_AUTH;
    challengeMessage.length = 4;
    memcpy(challengeMessage.payload, storedChallenge, 4);
    sendMessage(&challengeMessage);
}
```
On the app side, we can receive the challenge information and generate a response message based on the challenge.

```
public static final byte MSG_TYPE_CHALLENGE_RESPONSE = 0x04;
public static Message createChallengeResponseMessage(byte[] challenge, byte[] passwordHash) {
    byte[] payload = new byte[challenge.length + passwordHash.length];
    System.arraycopy(challenge, 0, payload, 0, challenge.length);
    System.arraycopy(passwordHash, 0, payload, challenge.length, passwordHash.length);
    return new Message(MSG_TYPE_CHALLENGE_RESPONSE, (short) payload.length, payload);
}
```
On the device side, the challenge information will be verified as follows:

```
void verifyChallengeResponseMessage(Message message) {
    byte challenge[4]; // 16-byte challenge
    byte passwordHash[16]; // 16-byte password hash
    memcpy(challenge, message.payload, 4);
    memcpy(passwordHash, message.payload + 4, 16);
    // verify the challenge and hash
    if (memcmp(challenge, storedChallenge, 4) == 0 && memcmp(passwordHash, storedPasswordHash, 16) == 0) {
        // authentication successful
    } else {
        // authentication failed
    }
}
```

#### Figuring Out The Timestamp Problem

When developing an Android app to control a door lock via Bluetooth, one crucial aspect to consider is timestamp management. To ensure secure communication, our system requires a reliable timestamp mechanism to prevent replay attacks. Since the ATMega328P microcontroller does not have a built-in Real-Time Clock (RTC) module, we need to implement a software-based solution to generate and manage timestamps.

It's essential to note that we don't require an exact timestamp; instead, we need an incremental timestamp that increments monotonically. This ensures that even if the device reboots, it can synchronize its timestamp before accepting new authentication messages, thereby preventing replay attacks.

#### Synchronize Timestamp From Android App On Request

To maintain timestamp consistency between the Android app and the board, we need to synchronize the timestamp at regular intervals. This can be achieved by sending a timestamp synchronization message from the Android app to the board. We can introduce a new message type, ```MSG_TYPE_SYNC_TIMESTAMP```, to the communication protocol to facilitate this process.


Here's an updated code snippet in the Android app:

```
public static final byte MSG_TYPE_SYNC_TIMESTAMP = 0x05;

public static Message createSyncTimestampMessage(long timestamp) {
    byte[] payload = new byte[4];
    payload[0] = (byte) (timestamp >> 24);
    payload[1] = (byte) (timestamp >> 16);
    payload[2] = (byte) (timestamp >> 8);
    payload[3] = (byte) timestamp;
    return new Message(MSG_TYPE_SYNC_TIMESTAMP, (short) payload.length, payload);
}
```
This code defines a new message type, ```MSG_TYPE_SYNC_TIMESTAMP```, and a method to create a timestamp synchronization message. The method takes a long timestamp value as input, converts it into a byte array, and returns a Message object with the appropriate message type and payload.


## Fixing The Data Trunk Truncated Or Missing Problem

### Problem Statement

During the development I encountered an issue where the authentication process would fail, even when the correct password was entered. Upon further investigation, I discovered that the problem was caused by truncated or missing data in the transmission process.

After conducting a thorough debugging process, I found that the data truncation issue occurred deterministically when the data buffer was large. On the other hand, the data missing problem occurred randomly. I suspected that the Bluetooth transmission mechanism was unreliable, leading to the data missing issue. Furthermore, I discovered that when transmitting a buffer size larger than the Maximum Transmission Unit (MTU) size, the remaining data would be truncated.

#### Understanding the Bluetooth Transmission Mechanism

To better understand the root cause of the problem, I delved into the technical details of Bluetooth transmission. Bluetooth is a wireless personal area network technology that operates on the 2.4 GHz frequency band and it uses a master-slave architecture, where one device acts as the master and the other as the slave. In our case, the Android app acts as the master, and the door control system acts as the slave.

Bluetooth transmission is based on a packet-based protocol, where data is divided into packets and transmitted over the air. Each packet has a maximum size limit, known as the MTU size, which varies depending on the Bluetooth device and the operating system. When transmitting a large amount of data, it is divided into multiple packets, and each packet is transmitted separately.

#### Causes of Data Truncation and Missing

There are two primary causes of data truncation and missing in Bluetooth transmission:

* MTU Size Limitation: When transmitting a buffer size larger than the MTU size, the remaining data is truncated, leading to data loss.
* Unreliable Transmission: Bluetooth transmission is not reliable, and packets can be lost or corrupted during transmission, resulting in missing data.


To overcome the data truncation and missing problem, I implemented the following solutions:

* Packet Fragmentation: I implemented packet fragmentation to divide large data buffers into smaller packets, each within the MTU size limit. This ensures that no data is truncated due to MTU size limitations.
* Error Detection and Correction: I implemented error detection and correction mechanisms to detect and retransmit corrupted or lost packets. This ensures that all data is transmitted reliably and accurately.

### Adjust the MTU size

The MTU size is typically fixed and depends on the Bluetooth device and operating system. To set the MTU size of the HC-05 Bluetooth module using C programming, you just need to send a specific AT command to the module, that is ```AT+MTU=32\r\n```, and remember to modify the Android app code accordingly:

```
public void run() {
        byte[] buffer = new byte[32];  // buffer within MTU size, the size MUST not exceed the MTU size of the HC-05 Bluetooth module
        int bytes; // bytes returned from read()
        logViewAdapter.add("Reading from BT!...");
        logViewAdapter.notifyDataSetChanged();
        // Keep listening to the InputStream until an exception occurs
        while (true) {
            try {
                // Read from the InputStream
                bytes = mmInStream.read(buffer);
                // Send the obtained bytes to the UI activity
                logViewAdapter.add(String.valueOf(bytes));
                logViewAdapter.notifyDataSetChanged();
            } catch (IOException e) {
                logViewAdapter.add("IOException on read: " + e.getMessage());
                logViewAdapter.notifyDataSetChanged();
            }
        }
    }
```

You may wonder why we need to set a small MTU size. Well, it's for compatibility reasons, as a larger MTU size results in less compatibility.

### Extending Packet Header Format

To implement packet fragmentation and reassembly, we need to design a packet header format that includes the necessary fields to identify and reassemble the packets. Here is an extended packet header format:


| Field | Length(bytes) | Description | Is New Extended |
|----------|----------|----------|----------|
| Packet Number (PN)| 2   | Unique packet number for reassembly| new |
| Sequence Number (SN) | 1| Sequence number for packet ordering. The most significant bit(MSB) is set to 1 for first packet,otherwise MSB is 0. The 7 least significant bits (LSB) are used to store the packet sequence number in decreasing order| new |
| Checksum (CK) | 2	| checksum for error detection. CRC-16 is used for checksum and all bytes except CK itself are calculated| new |
| Message Type (MT) | 1| identify the message type| |
| Message Size (MS) | 2| Length of the packet fragment payload (or total length for first packet) | |
| Payload | variable | Packet payload data | |


```
public class Message {
    public byte msgType;
    public short msgSize;
    public byte[] payload;

    public Message(byte msgType, short msgSize, byte[] payload) {
        this.msgType = msgType;
        this.msgSize = msgSize;
        this.payload = payload;
    }

    public static final byte MSG_TYPE_AUTH = 0x01;
    public static final byte MSG_TYPE_CHANGE_PASSWORD = 0x02;
    public static final byte MSG_TYPE_DOOR_COMMAND = 0x03;
    public static final byte MSG_TYPE_CHALLENGE_RESPONSE = 0x04;
    public static final byte MSG_TYPE_SYNC_TIMESTAMP = 0x05;

    public static Message createAuthMessage(byte[] payload) {
        return new Message(MSG_TYPE_AUTH, (short) payload.length, payload);
    }

    public static Message createChangePasswordMessage(byte[] payload) {
        return new Message(MSG_TYPE_CHANGE_PASSWORD, (short) payload.length, payload);
    }

    public static Message createDoorCommandMessage(byte[] payload) {
        return new Message(MSG_TYPE_DOOR_COMMAND, (short) payload.length, payload);
    }

    public static Message createChallengeResponseMessage(byte[] challenge, byte[] passwordHash) {
        byte[] payload = new byte[challenge.length + passwordHash.length];
        System.arraycopy(challenge, 0, payload, 0, challenge.length);
        System.arraycopy(passwordHash, 0, payload, challenge.length, passwordHash.length);
        return new Message(MSG_TYPE_CHALLENGE_RESPONSE, (short) payload.length, payload);
    }

    public static Message createSyncTimestampMessage(long timestamp) {
        byte[] payload = new byte[4];
        payload[0] = (byte) (timestamp >> 24);
        payload[1] = (byte) (timestamp >> 16);
        payload[2] = (byte) (timestamp >> 8);
        payload[3] = (byte) timestamp;
        return new Message(MSG_TYPE_SYNC_TIMESTAMP, (short) payload.length, payload);
    }
}
```

### Packet Fragmentation

When sending a large data buffer, we divide it into smaller packets using the following algorithm:

* Calculate the maximum packet size (MPS) based on the MTU size (32 bytes).
* Divide the data buffer into packets of size MPS.
* Assign a unique Packet Number (PN) to each packet in incremental order.
* For the first packet, set the MSB of Sequence Number (SN) to 1 and the rest of 7 LSB to MPS, set the Packet Length (PL) to the total length of all packets.
* For subsequent packets, set the MSB of Sequence Number (SN) to 0 and the rest of 7 LSB to a decreasing value starting from MPS minus 1 (e.g., 0x1F, 0x1E, ..., 0x1) and the Packet Length (PL) to the length of the packet payload.
* Calculate the Checksum (CK) for each packet using a checksum algorithm CRC-16.
* Construct the packet header using the PN, SN, MT,MS and CK fields.
* Transmit each packet over Bluetooth.


Here's an updated code snippet of ```class Message``` in the Android app to support packet fragmentation:


```
public static short globalPktNO = 0;

public static synchronized short generatePacketNumber() {
    return ++globalPktNO;
}

public static byte[][] fragmentPacket(Message msg, int mtuSize) {
	if (mtuSize <= 8) {
        throw new IllegalArgumentException("MTU size must be greater than 8");
    }

	short pktID = Message.generatePacketNumber();
    int numPackets = (int) Math.ceil((double) msg.payload.length / (mtuSize - 8));
    byte[][] packets = new byte[numPackets][];

    for (int i = 0; i < numPackets; i++) {
        int packetSize = Math.min(mtuSize - 8, msg.payload.length - i * (mtuSize - 8));
        byte[] packet = new byte[mtuSize];

		// set Packet Number
        packet[0] = (byte) (pktID >> 8); // pktNO
        packet[1] = (byte) (pktID & 0xFF); // pktNO

		// set Sequence Number
		packet[2] = (byte) ((i == 0) ? 0x80 : 0x00); // seqNO MSB
		packet[2] = (byte) (packet[2] | (packetSize - i)); // seqNO

        // set Checksum
		short chksum = crc16(msg.payload, packetSize);
        packet[3] = (byte) (chksum >> 8); // checksum high byte
        packet[4] = (byte) (chksum & 0xFF); // checksum low byte

		// set Message Type
        packet[5] = (byte) msg.msgType; // msgType

		// set Message Size
		short msize = (i == 0) ? msg.payload.length : packetSize;
        packet[6] = (byte) ((msize >> 8) & 0xFF); // message size high byte
        packet[7] = (byte) (msize & 0xFF); // message size low byte

        System.arraycopy(msg.payload, i * (mtuSize - 8), packet, 8, packetSize);
        packets[i] = packet;
    }
    return packets;
}



```

### Packet Reassembly

At the receiving end, we reassemble the packets using the following algorithm:

* Receive each packet and extract the PN, SN, MS, MT, and CK fields from the packet header.
* Verify the Checksum (CK) to ensure the packet is not corrupted.
* Use the MSB of Sequence Number (SN) field to determine whether the packet is the first packet or a subsequent packet.
* Use the rest of 7 LSB in Sequence Number (SN) to determine the total number of packets and the correct order of the packets.
* Reassemble the packets in the correct order using the Packet Number (PN) and Sequence Number (SN) fields.
* Verify the reassembled packet using the Checksum (CK) field.


Here's an updated code snippet of ```class Message``` in the Android app to support packet reassembly:

```
public static Message assemblePacket(byte[][] packets, int mtuSize) {
    if (packets == null || packets.length == 0) {
        throw new IllegalArgumentException("No packets to assemble");
    }

    // Extract the packet number, sequence number, total message size and message type from the first packet
    short pktNO = (short) (((packets[0][0] & 0xFF) << 8) | (packets[0][1] & 0xFF));
    byte seqNO = packets[0][2];
    byte msgType = packets[0][5];
	short totalSize = (short) (((packets[0][6] & 0xFF) << 8) | (packets[0][7] & 0xFF));

    // Create a new byte array to store the reassembled payload
    byte[] payload = new byte[totalSize];

    // Reassemble the payload
    int offset = 0;
    for (byte[] packet : packets) {
        int packetSize = packet.length - 8;
		if (offset == 0) {
			packetSize = mtuSize - 8;
		}
        System.arraycopy(packet, 8, payload, offset, packetSize);
        offset += packetSize;
    }

    // Verify the checksum
    short chksum = crc16(payload, totalSize);
    if (chksum!= ((short) (((packets[0][3] & 0xFF) << 8) | (packets[0][4] & 0xFF)))) {
        throw new IOException("Checksum mismatch");
    }

    // Create a new Message object with the reassembled payload
    Message msg = new Message(msgType, (short) totalSize, payload);
    return msg;
}
```


### Packet Transmission Example

Here is an example of packet fragmentation and reassembly for a 120-byte data payload with an MTU size of 32 bytes:

| Packet Number (PN) | Sequence Number (SN)	| Checksum (CK)	| Message Type (MT)	| Message Size (MS)	 | Payload |
|----------|----------|----------|----------|----------|----------|
| 0x0001(Unique packet number)| 0x85(First packet, MSB set to 1, rest bits set to 0x5)	| Calculated CRC-16 checksum | 0x01(Authentication Message) | 0x0078(whole 120 bytes payload)	| First 24 bytes of data |
| 0x0001(Unique packet number)| 0x04(Second packet, MSB set to 0)	| Calculated CRC-16 checksum | 0x01(Authentication Message) | 0x0018(24 bytes payload)	| Next 24 bytes of data |
| 0x0001(Unique packet number)| 0x03(Third packet, MSB set to 0)	| Calculated CRC-16 checksum | 0x01(Authentication Message) | 0x0018(24 bytes payload)	| Next 24 bytes of data |
| 0x0001(Unique packet number)| 0x02(Fourth packet, MSB set to 0)	| Calculated CRC-16 checksum | 0x01(Authentication Message) | 0x0018(24 bytes payload)	| Next 24 bytes of data |
| 0x0001(Unique packet number)| 0x01(Last packet, MSB set to 0)	| Calculated CRC-16 checksum | 0x01(Authentication Message) | 0x0018(24 bytes payload)	| Last 24 bytes of data |

In this example, the 120-byte data payload is divided into 5 packets, each with a maximum size of 32 bytes (MTU size). The Packet Number (PN) field is set to a unique value, and the Sequence Number (SN) field is used to determine the correct order of the packets. The Checksum (CK) field is calculated using a checksum algorithm (e.g., CRC-16) to detect errors during transmission.

At the receiving end, the packets are reassembled using the PN and SN fields, and the CK field is used to verify the integrity of the reassembled packet.

### Another Packet Transmission Example

How about a short packet? Below is an example for sending 24-byte data payload with an MTU size of 32 bytes:

| Packet Number (PN) | Sequence Number (SN)	| Checksum (CK)	| Message Type (MT)	| Message Size (MS)	 | Payload |
|----------|----------|----------|----------|----------|----------|
| 0x0002(Unique packet number)| 0x81(First packet, MSB set to 1, rest bits set to 0x1)	| Calculated CRC-16 checksum | 0x03(Door Command) | 0x0018(whole 24 bytes payload)	| whole 24 bytes of data |



### The Checksum Implementation

Here's the code snippet of ```crc16``` in C to support checksum: 

```
uint16_t crc16(uint8_t *data, uint16_t len) {
    uint16_t crc = 0xFFFF;
    uint8_t i;

    while (len--) {
        crc = (crc >> 8) ^ crc16_table[(crc & 0xFF) ^ *data++];
    }

    return ~crc;
}

const uint16_t crc16_table[256] = {
    0x0000, 0x1021, 0x2042, 0x3063, 0x4084, 0x50A5, 0x60C6, 0x70E7,
    0x8008, 0x9029, 0xA04A, 0xB06B, 0xC08C, 0xD0AD, 0xE0CE, 0xF0EF,
    0x1081, 0x1082, 0x2083, 0x3084, 0x4085, 0x50A6, 0x60C7, 0x70E8,
    0x8009, 0x902A, 0xA04B, 0xB06C, 0xC08D, 0xD0AE, 0xE0CF, 0xF0EF,
    0x2102, 0x3213, 0x4324, 0x5435, 0x6546, 0x7657, 0x8768, 0x9879,
    0xA09A, 0xB0AB, 0xC0AC, 0xD0AD, 0xE0AE, 0xF0AF,
    0x3121, 0x4232, 0x5343, 0x6454, 0x7565, 0x8676, 0x9787, 0xA098,
    0xB0A9, 0xC0AA, 0xD0AB, 0xE0AC, 0xF0AD,
    0x4122, 0x5233, 0x6344, 0x7455, 0x8566, 0x9677, 0xA078, 0xB089,
    0xC0A9, 0xD0AA, 0xE0AB, 0xF0AC,
    0x5123, 0x6234, 0x7345, 0x8456, 0x9567, 0xA078, 0xB089, 0xC0A9,
    0xD0AA, 0xE0AB, 0xF0AC,
    0x6124, 0x7135, 0x8146, 0x9157, 0xA068, 0xB089, 0xC0A9, 0xD0AA,
    0xE0AB, 0xF0AC,
    0x7136, 0x8147, 0x9158, 0xA069, 0xB0A9, 0xC0AA, 0xD0AB,
    0xE0AC, 0xF0AD,
    0x8148, 0x9159, 0xA06A, 0xB0AB, 0xC0AC, 0xD0AD, 0xE0AE,
    0xF0AF,
    0x915A, 0xA06B, 0xB0AC, 0xC0AD, 0xD0AE, 0xE0AF,
    0xF0B0,
    0xA06C, 0xB0AD, 0xC0AE, 0xD0AF, 0xE0B0, 0xF0B1,
    0xB0AE, 0xC0AF, 0xD0B0, 0xE0B1, 0xF0B2,
    0xC0B0, 0xD0B1, 0xE0B2, 0xF0B3,
    0xD0B2, 0xE0B3, 0xF0B4,
    0xE0B4, 0xF0B5,
    0xF0B5
};

```





### Error Detection and Correction

To ensure reliable transmission of packets, we need to detect packet loss and retransmit the packets. We will use a timeout-based and ACK approach to detect packet loss. If the transmitting end does not receive an ACK packet within a certain time period (e.g., 100ms), it assumes the packet is lost and retransmits the packet.

#### Packet Acknowledgment

The receiving end sends an acknowledgment packet (ACK) for each packet received. The ACK packet includes the Packet Number (PN),Sequence Number (SN) and a status field indicating whether the packet was received correctly. The error ACK is sent if receiving end receives incorrect packet (e.g., checksum fails, wrong SN).

Here's an updated code snippet in the Android app to support ACK:

```
public static final byte MSG_TYPE_ACK = 0x06;
public static Message createAckMessage(short packetNumber, byte sequenceNumber, boolean status) {
    byte[] payload = new byte[3];
    payload[0] = (byte) (packetNumber >> 8);
    payload[1] = (byte) (packetNumber & 0xFF);
    payload[2] = sequenceNumber;
    payload[3] = (byte) (status ? 1 : 0);
    return new Message(MSG_TYPE_ACK, (short) payload.length, payload);
}
```


#### Packet Retransmission

If the transmitting end does not receive an ACK packet within a certain time period (e.g., 100ms), it assumes the packet is lost and retransmits the packet using the same Packet Number (PN) and Sequence Number (SN) fields. Specifically, the packet loss detection works in the following way:

* The transmitting end maintains a timer for each packet sent.
* When a packet is sent, the timer is started with a timeout value (e.g., 100ms).
* If the transmitting end receives an ACK packet for the sent packet before the timer expires, the timer is cancelled.
* If an ACK packet of wrong status code is received, the transmitting end will restart the message transmission.
* If the timer expires, the packet is considered lost, and the transmitting end will start retransmitting the packet.
* If the retransmission counter exceeds a maximum value (e.g., 3), the transmitting end gives up retransmitting the message and reports an error.

Here's an updated code snippet in the Android app to support packet retransmission:

```
public class MessageTransmitter {
    // Timeout value for packet retransmission (100ms)
    private static final int TIMEOUT_VALUE = 100; 
    // Maximum number of retransmissions (3)
    private static final int MAX_RETRANSMISSIONS = 3;

    // Map to store packets with their sequence numbers
    private Map<Integer, byte[]> messageMap;
    // Next sequence number to be sent
    private int nextSequenceNumber;
    // Current message being transmitted
    private Message msg;
    // Fragmented packets of the current message
    private byte[][] msgPackets;

    // Input and output streams for Bluetooth communication
    private InputStream mmIn;
    private OutputStream mmOut;

    // Timer for packet retransmission
    private Timer timer;

    public MessageTransmitter(InputStream in, OutputStream out) {
        messageMap = new HashMap<>();
        nextSequenceNumber = 0;
        mmIn = in;
        mmOut = out;
    }

    /**
     * Send a message over Bluetooth
     * @param message the message to be sent
     */
    public void sendMessage(Message message) {
        msg = message;
        // Fragment the message into packets
        msgPackets = Message.fragmentPacket(message, 32); // 32 is the MTU size
        for (byte[] packet : msgPackets) {
            int sequenceNumber = packet[2] & 0x7F;
            messageMap.put(sequenceNumber, packet);
        }
        int seq = msgPackets[0][2] & 0x7F;
        nextSequenceNumber = seq - 1;

        // Send the first packet
        sendPacket(seq, msgPackets[0]);
    }

    /**
     * Send a packet over Bluetooth
     * @param sequenceNumber the sequence number of the packet
     * @param packet the packet to be sent
     */
    private void sendPacket(int sequenceNumber, byte[] packet) {
        // Send the packet over Bluetooth
        mmOut.write(packet);
        // Start a timer for packet retransmission
        startTimer(sequenceNumber, TIMEOUT_VALUE);
    }

    /**
     * Start a timer for packet retransmission
     * @param sequenceNumber the sequence number of the packet
     * @param timeoutValue the timeout value for retransmission
     */
    private void startTimer(int sequenceNumber, int timeoutValue) {
        timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                handlePacketLoss(sequenceNumber);
            }
        }, timeoutValue);
    }

    /**
     * Handle packet loss by retransmitting the packet
     * @param sequenceNumber the sequence number of the packet
     */
    private void handlePacketLoss(int sequenceNumber) {
        byte[] packet = messageMap.get(sequenceNumber);
        if (packet!= null) {
            // Retransmit the packet
            sendPacket(sequenceNumber, packet);
        } else {
            // Unknown error
        }
    }

    /**
     * Resend the entire message
     */
    private void resendMessage() {
        int retransmissionCount = msg.getRetransmissionCount();
        if (retransmissionCount < MAX_RETRANSMISSIONS) {
            msg.setRetransmissionCount(retransmissionCount + 1);
            // Resend all packets
            int seq = msgPackets[0][2] & 0x7F;
            nextSequenceNumber = seq - 1;
            sendPacket(seq, msgPackets[0]);
        } else {
            // Report error: packet lost after max retransmissions
            return;
        }
    }

    /**
     * Receive an ACK packet and handle it accordingly
     * @param sequenceNumber the sequence number of the packet
     * @param status the status of the packet (1 = acknowledged, 0 = error)
     */
    public void receiveAckPacket(int sequenceNumber, int status) {
        byte[] packet = messageMap.get(sequenceNumber);
        if (packet!= null) {
            if (status == 1) {
                // Packet acknowledged, send next packet
                timer.cancel(); // cancel the timer
                int next = nextSequenceNumber;
                if (next == 1) {
                    // All packets are sent
                    return;
                }
                nextSequenceNumber--;
                sendPacket(next, messageMap.get(next));
            } else {
                // Error occurred, resend all packets
                resendMessage();
            }
        }
    }
}

```

By using this design, we can efficiently fragment and reassemble large data buffers over Bluetooth, while ensuring reliable and error-free transmission.


## Get Notified When Door Is Open Or Closed

As we've implemented the door control system, we've noticed that there's a lack of feedback when sending commands to the device. Every time we click the open or close button, we're left wondering whether the command was executed successfully or not. Sometimes, the door opens or closes as expected, but other times, it doesn't respond at all. And occasionally, strange things happen, leaving us with an incorrect door status.

This lack of feedback is not only frustrating but also leads to uncertainty and potential errors. To address this issue, we need to implement a notification mechanism that allows the receiver to notify the sender of the command execution status.

To meet these requirements, we can introduce a new message type, ```MSG_TYPE_NOTIFY```, which can be used for all 
types of notifications between the device and the app. The message payload for ```MSG_TYPE_NOTIFY``` can consist of the following fields:

* Packet Number (PN): a short integer indicating the corresponding packet number of for notification.
* Message Type (MT): a byte indicating the type of command that triggered the notification (e.g., auth, change password, command, challenge response, etc.)
* Result (Ret): a short integer indicating the result of the command (e.g., 0 for success, 1 for failure, etc.)
* Description (Desc): a string describing the result of the command (e.g., "Authentication successful", "Password changed successfully", "Door opened successfully", etc.)

By using MSG_TYPE_NOTIFY as a unified notification message type, we can provide a consistent way of notifying between Android app and the door of message execution status for all types of commands.

Here's an updated code snippet in the Android app to support NOTIFY:

```
public static final byte MSG_TYPE_NOTIFY = 0x07;
public static Message createNotifyMessage(short packetNumber, byte messageType, short result, byte[] desc) {
    byte[] payload = new byte[5 + desc.length];
    payload[0] = (byte) (packetNumber >> 8);
    payload[1] = (byte) (packetNumber & 0xFF);
    payload[2] = messageType;
	payload[3] = (byte) (result >> 8);
    payload[4] = (byte) (result & 0xFF);
	if (desc.length > 0) {
		System.arraycopy(desc, 0, payload, 5, desc.length);
	}
    return new Message(MSG_TYPE_NOTIFY, (short) payload.length, payload);
}
```

## Put It All Together

Wow, we've come a long way since we started this project! From a simple Bluetooth transmission command to a full-fledged door control system with authentication, password change, and status update notification features. It's amazing how much complexity can be added to a system when you try to make it more secure and user-friendly.

As I look back on our journey, I realize that we've been focusing on individual components of the system, like authentication and password change. But now, it's time to put all the pieces together and see how they work in harmony.

### Sequence Diagram

To help us visualize the entire interaction process, let's create a sequence diagram that shows how the Android app and door control system communicate with each other. Here's what it might look like:

```
 Android App                         Door Control System
      |                                      |
      |  Connected via Bluetooth 		  	 |
      |<------------------------------------>|
      |                                      |
      |  Each side set a fix MTU 			 |
      |<------------------------------------>|
      |                                      |
      |  Send auth request                   |
      |------------------------------------->|
      |                                      |
      |  Send timestamp sync request 		 |
      |<-------------------------------------|
      |                                      |
      |  Send timestamp response             |
      |------------------------------------->|
      |                                      |
      |  Receive timestamp sync result       |
      |<-------------------------------------|
      |                                      |
      |  Send auth request                   |
      |------------------------------------->|
      |                                      |
      |  Receive auth challenge              |
      |<-------------------------------------|
      |                                      |
      |  Send auth response                  |
      |------------------------------------->|
      |                                      |
      |  Receive auth result                 |
      |<------------------------------------ |
      |                                      |
      |  Send change password request        |
      |------------------------------------->|
      |                                      |
      |  Receive change password result      |
      |<-------------------------------------|
      |                                      |
      |  Send door control command           |
      |------------------------------------->|
      |                                      |
      |  Receive door control notify result  |
      |<-------------------------------------|
      |                                      |
      |  Receive door status update  		 |
      |<-------------------------------------|
      |                                      |
      |  Display door status update  		 |
      |                                      |

```
As I look at this sequence diagram, I'm struck by how many different interactions are involved in the door control process. From authentication to password change to door control, there are so many different messages being sent back and forth between the Android app and door control system. Note that even a one way interaction in the sequence diagram may contains multiple round-chip data transfer because of the message fragmentation and message notify.

But despite the complexity, I'm proud of what we've accomplished. We've created a system that's not only functional but also secure and user-friendly. And with the sequence diagram, we can see exactly how all the different components fit together to create a seamless user experience.

###  Next Steps

So what's next? Well, now that we have a complete system including hardware and software, it's time to test it out and see how it works in practice. 
You could download the whole project source code and test if yourself. Keep in mind that the UI is urgly as I am not ready to spend time on it.

And then, of course, there's the possibility of adding even more features to the system. Maybe we could add support for multiple doors or integrate with other smart home devices. The possibilities are endless, and I'm excited to see where this project will take us in the future.


