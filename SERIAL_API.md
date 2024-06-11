
# 🎛️ Nano_D++ API

## 👋 Overview

This page describes the configuration and control protocol for the Binaris [Nano_D++](https://store.binaris.io/products/nano_d-sensory-hid).

The current implementation uses the device's serial port, via USB. The messages are JSON encoded, and are sent in both directions.

This is the protocol used to configure and customize the device via [ZERO/ONE](https://github.com/katbinaris/zeroone), or it could be used to implement your own projects involving Nano_D++ communications.

It is not recommended to use this protocol for regular use, for that you should use either MIDI or HID.

### 🔛 Serial Transport

The Nano_D++ has a USB 2.0 HS (10MBit) USB port. 

In addition to presenting the Nano_D++ as a HID (human input device - like a kind of keyboard) to a laptop or computer, the USB port allows you to open a Serial connection to the device as well.

The Serial speed is automatically negotiated when connecting via USB. The connection settings should be 8 bits, no parity and 1 stop bit (8N1).

You can connect with a serial terminal program like CoolTerm, TeraTerm or the unix "screen" command, or fire up [ZERO/ONE](https://github.com/katbinaris/zeroone), which will automatically detect your devices.

## 📑 Serial protocol

The Serial protocol is simple: small JSON messages are sent in both directions, which encode commands or data to be exchanged with the device. The message formats are described below.

Each JSON message is separated from the next one by a newline character. Newline characters within the JSON structure are not permitted. Where newline characters are used within the transported field values of the JSON message, they should be text escaped as '\n'.

### ➡️ Outgoing messages

**Error messages** are sent when errors occur in the system, or as response to erroneous command messages. **Debug messages** should be output to the console to aid in debugging.

```json
{ "error": "An error occurred." }
{ "error": "Another kind of error.", "msg": "This one comes with a detail message." }

{ "debug": "A message for the console." }
```

**Idle messages** are sent once per second by the device if no event messages have been sent recently. They include the idle time, the duration (ms) since the last event message (e.g. the last user interaction):

```json
{ "idle": 16233 }
```

**Saved messages** are sent after every save of the device settings and/or profiles. Note: if errors occur during save, details of these are sent as error messages.

```json
{ "saved": true }
```

**Event messages** are sent by the system on user interaction with the device. There are a few types depending on the event. Multiple events could arrive in the same JSON message.

```json
{ "kd": "A", "ks": "AbCd" }              // kd = key-down, ks = keys-state
{ "ku": "AC", "kd": "D", "ks": "abcD" }  // ku = key-up
{ "p": 42 }                              // p = current position
```

Other outgoing message are sent in response to commands, and are described below.

### ⬅️ Commands

#### Profile commands

**Get a list of all profile names:**
```json
{ "profiles": "#all" }
```

Response:
```json
{
  "profiles": [
    "DEFAULT PROFILE",
    "EXAMPLE PROFILE",
    "BINARIS",
    "DEMO",
    "DEMO COPY"
  ],
  "current": "EXAMPLE PROFILE"
}
```

<hr>

**Get a single profile's details:**

```json
{ "profile": "EXAMPLE PROFILE" }
```

The response includes all profile fields, and would arrive in one line, but is shown here formatted for legibility:
```json
{
  "profile": {
    "version": 2,
    "name": "EXAMPLE PROFILE",
    "desc": "HELLO WORLD!",
    "profileTag": "DEMO",
    "ledEnable": true,
    "ledBrightness": 100,
    "ledMode": 0,
    "pointer": 16777215,
    "primary": 547180,
    "secondary": 4654093,
    "buttonAIdle": 547180,
    "buttonBIdle": 1715794,
    "buttonCIdle": 2098468,
    "buttonDIdle": 4654093,
    "buttonAPress": 16777215,
    "buttonBPress": 16777215,
    "buttonCPress": 16777215,
    "buttonDPress": 16777215,
    "keys": [
        {
            "pressed": [
                {
                    "type": "key",
                    "keyCodes": [
                        17
                    ]
                },
                {
                    "type": "midi",
                    "channel": 5,
                    "cc": 5,
                    "val": 5
                }
            ]
        },
        {
            "pressed": [
                {
                    "type": "mouse",
                    "buttons": 1
                }
            ]
        },
        {
            "pressed": [
                {
                    "type": "gamepad",
                    "buttons": 1
                }
            ]
        },
        {
            "pressed": [
                {
                    "type": "next_profile"
                }
            ]
        }
    ],
    "knob": [
      {
        "valueMin": 0,
        "valueMax": 127,
        "angleMin": 0,
        "angleMax": 0,
        "wrap": false,
        "step": 0,
        "keyState": 0,
        "haptic": {
          "mode": 0,
          "startPos": 0,
          "endPos": 127,
          "detentCount": 127,
          "vernier": 0,
          "kxForce": false,
          "outputRamp": 0,
          "detentStrength": 0
        },
        "type": "midi",
        "channel": 1,
        "cc": 28
      }
    ],
    "guiEnable": false,
    "audio": {
      "clickType": "hard",
      "keyClickType": "clack",
      "clickLevel": 100
    }
  }
}
```

Values for haptic mode:

 - 0: REGULAR    // Only coarse detents used
 - 1: VERNIER    // Coarse with fine between
 - 2: VISCOSE    // Resistance while turning
 - 3: SPRING     // Snap back to center point

<hr>

**Changing profiles from keys:**

```json
{
    "profile": {
        "keys": [
            {
                "pressed": [
                    {
                        "type": "profile",
                        "name": "SPECIFIC PROFILE"
                    },
                    {
                        "type": "profile",
                        "name": "ANOTHER PROFILE"
                    },
                    {
                        "type": "next_profile"
                    },
                    {
                        "type": "prev_profile"
                    },
                ]
            },
        ]
    }
}
```


<hr>

**Update one or more profile fields:**

```json
{
  "profile": "EXAMPLE PROFILE",
  "updates": {
    "desc": "LOOK AT ME!",
    "ledBrightness": 50,
    "secondary": 3654093,
  }
}
```

<hr>

**Rename a profile:**

```json
{
  "profile": "EXAMPLE PROFILE",
  "updates": {
    "name": "NEW NAME"
  }
}
```

<hr>

**Create a new profile, just send an update with a non-existing profile name:**

```json
{
    "profile": "NEW PROFILE",
    "updates": {
        "desc": "A NEW PROFILE",
    }
}
```

All required fields will be set to default values if not provided.

<hr>

**Set the current profile:**

```json
{ "current": "DEFAULT PROFILE" }
```

TODO what would be the best response?

<hr>

**Delete or reorder the profiles**

e.g. put "EXAMPLE PROFILE" first, "DEFAULT PROFILE" last, and remove the "DEMO COPY" profile:

```json
{
  "profiles": [
    "EXAMPLE PROFILE",
    "BINARIS",
    "DEMO",
    "DEFAULT PROFILE",
  ]
}
```

#### Motor commands

**Set SimpleFOC registers:**

```json
{ "R": "17=2.0 19=7.7" }
```

**Reset motor calibration:**

```json
{ "recalibrate": true }
```

#### System commands

**Get device settings:**

```json
{ "settings": "?" }
```

Response (formatted on multiple lines for clarity):

```json
{
  "settings": {
    "debug": false,
    "ledMaxBrightness": 150,
    "maxVelocity": 10,
    "maxVoltage": 5,
    "deviceOrientation": 1,
    "deviceName": "Nano_3053f07554dc",
    "wifiEnabled": false,
    "serialNumber": "9052f07554dc",
    "firmwareVersion": "1.0.0",
    "midiUsb": {
      "in": true,
      "out": true,
      "thru": false,
      "route": false,
      "nano": true
    },
    "midi2": {
      "in": true,
      "out": true,
      "thru": false,
      "route": false,
      "nano": true
    },
    "sysexId": 0,
    "idleTimeout": 60000
  }
}
```

<hr>

**Enable or disable debug messages or set device options:**

One or more settings values can be set in the same message.

```json
{
  "settings": {
    "debug": true,
    "ledMaxBrightness": 170,
    "maxVelocity": 45,
    "maxVoltage": 4.4
  }
}
```

<hr>

**Save the settings and profiles to SPIFFs:**


```json
{ "save": true }
```

<hr>

**Reload the settings and profiles from SPIFFs:**


```json
{ "load": true }
```

Note: the save command can be included with other commands, e.g. updating settings and saving at the same time...

<hr>

**Show a message on the device screen:**

```json
{
  "message": {
    "title": "Hey!",
    "text": "Get some work done.",
    "duration": 5
  }
}
```

<hr>

**Control the device screen:**

```json
{
  "screen": {
    "title": "The Title",
    "data1": "Subtitle",
    "data2": "Text Content",
    "data3": "Text Content",
    "data4": "Text Content"
  }
}
```

All fields are optional, if omitted the layout will blank them.

Note: the meaning of the data1 - data4 fields depends on the layout of the screen.
