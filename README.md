## lcd

**lcd is io.js and Node.js compatable**

A Node.js Hitachi HD44780 LCD driver for Linux boards such as the BeagleBone or
Raspberry Pi. Heavily inspired by the Arduino
[LiquidCrystal library](http://arduino.cc/en/Tutorial/LiquidCrystal).

Most LCDs compatible with the HD44780 have a sixteen pin interface. This
Node.js module uses six of these interface pins for controlling such displays.
Register select (RS), enable (E), and four data bus pins (D4-D7). The
read/write (R/W) pin is assumed to be tied low to permanently select write
mode.

## Installation

    $ [sudo] npm install lcd

lcd requires Node.js v0.8.0 or higher.

**BeagleBone Prerequisites**

There are no prerequisites for using lcd on the BeagleBone or BeagleBone
Black, when Debian is being used.

Before installing lcd on stock Ångström on the BeagleBone or BeagleBone
Black, three Python modules need to be installed; python-compiler, python-misc,
and python-multiprocessing. They can be installed with the following commands:

```bash
$ opkg update
$ opkg install python-compiler
$ opkg install python-misc
$ opkg install python-multiprocessing
```

## News & Updates

### lcd v0.2.0 breaking asynchronous print change

As of lcd v0.2.0 the print method is asynchronous. In previous versions it was
synchronous.

## Usage

The following nine line program can be used to make a UTC digital clock.

```js
var Lcd = require('../lcd'),
  lcd = new Lcd({rs: 27, e: 65, data: [23, 26, 46, 47], cols: 8, rows: 1});

lcd.on('ready', function () {
  setInterval(function () {
    lcd.setCursor(0, 0);
    lcd.print(new Date().toISOString().substring(11, 19));
  }, 1000);
});
```

Here it is up and running on a BeagleBone Black wired up to an 8x1 display:

<img src="https://github.com/fivdi/lcd/raw/master/example/digital-clock-8x1.jpg">

After requiring the lcd module, the above program creates an Lcd object. The
constructor function is passed all the necessary information.

The six LCD interface pins used to control the display need to be wired up to
six GPIOs on the BeagleBone Black. GPIOs on Linux are identified by unsigned
integers. The relevant information for all six GPIOs used here is shown in the
following table:

BBB Expansion Header | GPIO No. | LCD Function | LCD Pin No.
:---: | :---: | :---: | :---:
P8_13 | 23 | Data Bus Bit 4 | 11
P8_14 | 26 | Data Bus Bit 5 | 12
P8_15 | 47 | Data Bus Bit 7 | 14
P8_16 | 46 | Data Bus Bit 6 | 13
P8_17 | 27 | Register Select |  4
P8_18 | 65 | Enable  |  6

The constructor function is also told how many columns and rows the display
has, eight and one respectively in this case.

It takes several milliseconds to initialize an LCD. The constructor starts the
initialization process, but it doesn't wait for it to complete. Instead,
a 'ready' event is emitted after the LCD has been completely initialized and is
ready for usage.

The 'ready' handler leverages setInterval to execute a function that updates
the time displayed on the LCD once a second.

Adding the following few lines will turn the digital clock into a good citizen
that cleans up after itself.

```js
// If ctrl+c is hit, free resources and exit.
process.on('SIGINT', function () {
  lcd.close();
  process.exit();
});
```

## API

**Lcd(config)**

Returns a new Lcd object which inherits from EventEmitter. A 'ready' event will
be emitted when the display is ready for usage.

The config object has these possibilities:

 * **cols** LCD column count. Defaults to 16.
 * **rows** LCD row count. Defaults to 1.
 * **largeFont** Use 5x10 dot font. Defaults to false for 5x8 dot font.
 * **rs** Register select GPIO number.
 * **e** Enable GPIO number.
 * **data** Array of four GPIO numbers for data bus bits D4 through D7.

**print(val, [cb])**

Converts val to string and writes it to the display asynchronously.

If the optional completion callback is omitted, a 'printed' event is emitted
after the operation has completed. The string representation of val is passed
to the 'printed' event handler as the first argument. If an error occurs, an
'error' event will be emitted and an error object will be passed to the
'error' event handler as the first argument.

If the optional completion callback is specified, it gets two arguments
(err, str), where err is reserved for an error object and str is the string
representation of val. If the optional completion callback is specified, no
'printed' or 'error' event will be emitted.

The example print-twice-20x4.js demonstrates how to print two strings in
succession using events.

**clear([cb])**

Clears display and returns cursor to the home position asynchronously.

If the optional completion callback is omitted, a 'clear' event is emitted
after the operation has completed. If an error occurs, an 'error' event will
be emitted and an error object will be passed to the 'error' event handler
as the first argument.

If the optional completion callback is specified, it gets one argument (err),
where err is reserved for an error object. If the optional completion callback
is specified, no 'clear' or 'error' event will be emitted.

**home([cb])**

Returns cursor to home position asynchronously. Also returns display being
shifted to the original position.

If the optional completion callback is omitted, a 'home' event is emitted
after the operation has completed. If an error occurs, an 'error' event will
be emitted and an error object will be passed to the 'error' event handler
as the first argument.

If the optional completion callback is specified, it gets one argument (err),
where err is reserved for an error object. If the optional completion callback
is specified, no 'home' or 'error' event will be emitted.

**setCursor(col, row)** Moves the cursor to the specified col and row.
Numbering for col and row starts at zero.

**cursor()** Turn cursor on.

**noCursor()** Turn cursor off.

**blink()** Turn cursor blink on.

**noBlink()** Turn cursor blink off.

**scrollDisplayLeft()** Shift display to the left. Cursor follows the display
shift.

**scrollDisplayRight()** Shift display to the right. Cursor follows the display
shift.

**leftToRight()** Sets cursor move direction to left to right.

**rightToLeft()** Sets cursor move direction to right to left.

**autoscroll()** Automatically shift display when data is written to display.

**noAutoscroll()** Turn automatic shifting off.

**close()** Frees (unexports) all GPIOs used by the Lcd.

## Example "Hello, World!" on an 8x1 display

"Hello, World!" is five characters too long for an 8x1 display, but by moving
the cursor to the ninth column, turning autoscroll on, and displaying a new
character every 300 milliseconds the text can be scrolled onto the display
character by character. Note that an 8x1 display actually has eighty columns
but only eight of them are visible.

```js
var Lcd = require('../lcd'),
  lcd = new Lcd({rs: 27, e: 65, data: [23, 26, 46, 47], cols: 8, rows: 1});

function print(str, pos) {
  pos = pos || 0;

  if (pos === str.length) {
    pos = 0;
  }

  lcd.print(str[pos]);

  setTimeout(function () {
    print(str, pos + 1);
  }, 300);
}

lcd.on('ready', function () {
  lcd.setCursor(8, 0);
  lcd.autoscroll();
  print('Hello, World! ** ');
});
```

## Tested with the following displays

[NHD-0108FZ-FL-YBW-33V3](http://www.newhavendisplay.com/nhd0108fzflybw33v3-p-5155.html)

[NHD-0420DZ-FL-YBW-33V3](http://www.newhavendisplay.com/nhd0420dzflybw33v3-p-5168.html)

