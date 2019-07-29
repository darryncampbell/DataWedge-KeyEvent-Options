# DataWedge KeyEvent Options

The recently released [DataWedge 7.3](http://techdocs.zebra.com/datawedge/latest/guide/output/keystroke/) quietly introduced a new feature that customers have been requesting for a long time now, the ability to send data as key events.  What this means in practice is an application can now listen for **keypress events** whereas previously the application could only scan into text fields using the keystroke plugin.

There are a number of advantages to this approach:

- Any web application can now receive barcode scans by **listening for key press events on the DOM** without having to worry about having the cursor in the correct input field or advancing to the next field after the barcode is scanned.
- Applications written for HID based external scanners can be run on Zebra Android devices **without having to modify the application** at all.
- Web applications can be run **entirely within the Browser** without having to worry about external frameworks or native dependencies
- Future compatibility - this is a very simple solution and you are **not tying yourself to the hardware manufacturer**.

Within your DataWedge profile navigate to the Keystroke output section, then Key event options to see the above options under DW 7.3.

![DataWedge options](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/screenshots/datawedge-key-options1.jpg)

## Option: "Send Characters as Events"

**Send Characters as Events** is the critical setting for receiving data as keypress events, set it to true to receive any standard text character as a keypress (there is an ASCII table at the bottom of this post for reference) - everything between space and tilde (~).

When enabled, you can say something like the following in your JavaScript:

## Sample JavaScript
```javascript
document.addEventListener('keypress', keypressHandler);

function keypressHandler(e) 
{
    const keypressoutput = document.getElementById('pressed_keys');
    if (e.keyCode == 13)    //  Enter key from DataWedge
        keypressoutput.innerHTML += "<BR>";
    else
        keypressoutput.innerHTML += e.key;
}
```

For each scanned barcode the text is delivered to the application as key presses and the above example will appended a "<BR>" whenever it sees the 'Enter' character.  The code will appear as follows in Chrome:

![Application output](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/screenshots/app.png)

The sample page is within this project and you will find some example barcodes lower down.  *Yes, I know it does not look very pretty*

### Determining the end of barcodes

Obviously all this page is doing is listening for keystrokes so the application needs to determine where one barcode finishes and the next one begins.  There are a few popular ways to achieve this:

- Where the barcodes are all a fixed length, you can listen for a specific number of characters.
- Where barcodes end in a particular character, you can break on that character (the sample app uses this approach)
- Timing: Humans can only scan so fast so whenever a long enough pause between characters is detected, assume the barcode is complete.

## Options: "Send Enter as String" / "Send Tab as String"

These options are *not needed* by a solution listening for keypress events within the app (or on the DOM for web apps); they still exist for compatibility with applications that scan into text fields, tabbing between those text fields and entering (submitting) the form that contains those text fields.

## Option: Send Control Characters as Events

Do NOT assume (as I did) that e.g. sending ASCII code decimal 7 will result in the web page receiving the BELL character.  The control characters that this option refers to are defined fully in the "**[ASCII Control Character Table](http://techdocs.zebra.com/datawedge/latest/guide/output/keystroke/#asciicontrolcharactertable)**" present in the official documentation.

For example, ASCII code decimal 7 actually refers to the code equivalent of CTRL+G (decimal 71).

By way of a demonstration, **if we enable this option** and send a barcode containing the ASCII code decimal 7 we can look at how this event is received by the application:

![ASCII Decimal 7](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/screenshots/ctrl-plus-g.png)

The barcode used for this is given at the bottom of this document.  Note that the ASCII keycode **71** is received in the application for both KEYUP and KEYDOWN but NOT for KEYPRESS, this is expected behaviour since Android will not generate a KEYPRESS event for ASCII code 71.  You can observe the same behaviour by loading the test page in a desktop browser and pressing CTRL+G, where you will also receive a keycode 71.

## Things to Bear in Mind

- **Android will not generate a keypress event for every character**, as was noted above you will not get keypresses for many of the control keys (which also includes TAB - ASCII decimal 9).
- **Vysor will interfere with the operation of the DataWedge keystroke output plugin** on the device as it installs its own keyboard.  Personally I find it easier to uninstall Vysor when I am not actively using it. 
- **DataWedge seems to return ASCII code 13 for the enter key** regardless of how the barcode was encoded (with \n or \r).  This could be an artifact of how I created the barcodes, I am not sure but I have included both below.

## Barcodes

Use the following barcodes to test the functionality of the test page

![Barcode 1234567890](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/barcodes/1234567890.png)

QR Barcode: 1234567890

The above barcode is a standard barcode without any special characters appended to the end.

![Barcode 1234567890-Enter](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/barcodes/1234567890-Enter.gif)

QR Barcode: 1234567890\n

The above barcode has the ASCII character with decimal 10 (LF) appended to the end of it.

![Barcode 1234567890-r](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/barcodes/1234567890-r.gif)

QR Barcode: 1234567890\r

The above barcode has the ASCII character with decimal 13 (CR) appended to the end of it.

![Barcode 1234567890-7](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/barcodes/1234567890-7.gif)

QR Barcode: 1234567890\7

The above barcode has the ASCII character with decimal 7 (BELL) appended to the end of it.

![ASCII Table](https://raw.githubusercontent.com/darryncampbell/DataWedge-KeyEvent-Options/master/screenshots/ascii-codes-table.png)
