# Frequently Asked Questions and Answers

## Hardware and Prototype

### Relay Connection Conundrum: NO or NC?

Initially, I connected the output wires to the common and NO (Normally Open) side of the relay,
assuming it was the correct configuration. My reasoning was that the switch would be voltage-free
initially, and then we would apply voltage to it.

### App Success, But Wall Switch Failure!

The Android app worked flawlessly, allowing me to open and close the door remotely. However, the
wall switch ceased to function, and its LED indicator remained dark. I should have investigated
further, but my excitement about the app's success overshadowed my curiosity.

### Other Remotes Also Malfunction

Later, my wife attempted to use the wall switch to open the garage door, but it didn't work. She
then tried using her radio frequency (R/F) remote, but that too failed. It dawned on me that the
circuit required a steady current, provided by the ODM, to power the wall switch's LED and enable
the R/F remotes.

### The App's Unexpected Behavior

Interestingly, the app now briefly disconnects the switch and then reconnects it to the NC side.
This is the opposite of what I initially expected, but fortunately, the app and all switches now
work seamlessly together.
