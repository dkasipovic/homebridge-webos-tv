# homebridge-webos-tv

`homebridge-webos-tv` is a plugin for Homebridge which allows you to control your webOS TV! It should work with all TVs that support webOS2 and newer.
The idea is that the TV should be controlled completely from the native HomeKit iOS app and Siri, that is why volume appears as a light bulb or external input as a switch.

### Features
* HomeKit TV integration
* HomeKit automation
* Turn TV on / off
* Mute / Unmute
* Volume control (as light bulb, buttons or through iOS remote app)
* Change sound output
* Switch inputs
* Open apps
* Channel control
* Media control
* Show notifications
* Emulate remote control
* Run sequences of remote control button presses

To use the plugin iOS 12.2 or higher is recommended. If your iOS device is not running at least on iOS12.2 then you should use the legacy tv service.

## Installation

If you are new to Homebridge, please first read the Homebridge [documentation](https://www.npmjs.com/package/homebridge).
If you are running on a Raspberry, you will find a tutorial in the [homebridge-punt Wiki](https://github.com/cflurin/homebridge-punt/wiki/Running-Homebridge-on-a-Raspberry-Pi).

Install homebridge:
```sh
sudo npm install -g homebridge
```

Install homebridge-webos-tv:
```sh
sudo npm install -g homebridge-webos-tv
```

## Configuration

Add the accessory in `config.json` in your home directory inside `.homebridge`.

Example configuration:

```js
{
  "accessories": [
    {
      "accessory": "webostv",
      "name": "My webOS tv",
      "ip": "192.168.0.40",
      "mac": "ab:cd:ef:fe:dc:ba",
      "keyFile": "/home/pi/.homebridge/lgtvKeyFile",
      "pollingInterval": 10,
      "inputs": [
          {
            "appId": "com.webos.app.livetv",
            "name": "Live TV"
          },
          {
            "appId": "com.webos.app.hdmi1",
            "name": "PS4"
          },
          {
            "appId": "youtube.leanback.v4",
            "name": "YouTube",
            "params":{
		"contentTarget": "https://www.youtube.com/tv?v=Bey4XXJAqS8"
            }
          },
          {
            "appId": "com.webos.app.photovideo",
            "name": "Photo Video"
          }
      ],
      "showInputButtons": true,
      "volumeControl": "buttons",
      "channelControl": false,
      "mediaControl": false,
      "channelButtons": [3,5,7,8],
      "notificationButtons": [
         "Motion detected - living room",
         "Motion detected - kitchen"
      ],
      "remoteControlButtons": [
         "HOME",
         "LIST",
         "EXIT"
      ],
      "soundOutputButtons": [
         "tv_speaker",
         "external_optical",
         "headphone"
      ],
       "remoteSequenceButtons": [
          {
              "sequence": ["HOME","RIGHT","RIGHT","RIGHT","ENTER"],
              "name": "screen_share_seq"
          },
          {
              "sequence": ["VOLUMEUP","VOLUMEDOWN","MUTE","MUTE"],
              "name": "volume_seq",
              "interval": 1000
          }
      ]
    }
  ]  
}
```

You also need to enable **mobile TV on** on your TV for the turn on feature to work correctly.

This is located on your TV under `Settings > General > Mobile TV On`

On newer TVs **LG Connect Apps** under the network settings needs to be enabled.

### Configuration fields
- `accessory` [required]
Should always be "webostv".
- `name` [required]
Name of your accessory.
- `ip` [required]
ip address of your TV.
- `mac` [required]
Mac address of your TV.
- `broadcastAdr` [optional]
If homebridge runs on a host with more than one network interface use this to specify the broadcast address.
- `keyFile` [optional]
To prevent the TV from asking for permission when you reboot homebridge, specify a file path to store the permission token. If the file doesn't exist it'll be created. Don't specify a directory or you'll get an `EISDIR` error.
- `prefsDir` [optional]
The directory where input names and TV model info should be saved. **Default: "~/.webosTv"**
- `pollingInterval` [optional]
The TV state background polling interval in seconds. **Default: 5**
- `legacyTvService` [optional]
Wheter to use the legacy TV service. Should be used if your iOS device is not running on iOS12.2 or newer. TV will be emulated as a switch.  **Default: false**  
- `inputs` [optional] 
Inputs which should appear under the *Inputs* list on yor TV accessory. When you use the legacy tv service, buttons for the inputs will be created. **Default: "" (disabled)**
  - Set an array of app IDs or objects as the value. An object needs to have the *appId* and *name* property
  - You can optionally specifiy the *params* property as key/value object to launch the applciation with the specified parameters
  - To get the app ID simply open an app on your TV and check the homebridge console. The app ID of the opened app will be printed.
  - Some of the default TV inputs which can be used:
    - *com.webos.app.livetv*
    - *com.webos.app.hdmi1*
    - *com.webos.app.hdmi2*
    - *com.webos.app.hdmi3*
    - *com.webos.app.externalinput.component*
    - *com.webos.app.externalinput.av1*
  - Inputs and apps can also be switched when the TV is off, in that case an attempt to power on the TV and switch to the chosen input will be made
- `showInputButtons` [optional]
Wheter to additionally show inputs as buttons. Useful for automation. **Default: false**
- `volumeControl` [optional]
Wheter the volume control service is enabled. **Default: true**  
Available values:
  - *true* - slider and buttons
  - *false* - no volume control
  - *"slider"* - just slider
  - *"buttons"* - just buttons
- `volumeLimit` [optional]
The max allowed volume which can be set using the volume service. Range 1-100. **Default: 100**
- `channelControl` [optional]
Wheter the channel control service is enabled. **Default: true**
- `mediaControl` [optional]
Wheter the media control service is enabled. Buttons: play, pause, stop, rewind, fast forward. **Default: false**
- `channelButtons` [optional] 
Wheter the channel buttons service is enabled. This allows to create buttons for the channels of your choice. This way you can quickly switch between favorite channels. **Default: "" (disabled)**
  - Set an array of channel numbers as the value
  - Channels can also be opened when the TV is off, in that case an attempt to power on the TV and afterwards open the chosen channel will be made.
- `notificationButtons` [optional] 
Wheter the notification buttons service is enabled. This allows to create buttons which when pressed display the specified text on the TV screen. Useful for HomeKit automation or to display text on TV for viewers. **Default: "" (disabled)**
  - Set an array of notification texts as the value
- `remoteControlButtons` [optional] 
Wheter the remote control buttons service is enabled. This allows to emulate remote control buttons. **Default: "" (disabled)**
  - Set an array of commands as the value. Possible values are:
    - *1*, *2*, *3*, *4*, *5*, *6*, *7*, *8*, *9*, *0*, *LIST*, *AD*, *DASH*
    - *MUTE*, *VOLUMEUP*, *VOLUMEDOWN*, *CHANNELUP*, *CHANNELDOWN*, *HOME*, *MENU*
    - *UP*, *DOWN*, *LEFT*, *RIGHT*, *CLICK*, *BACK*, *EXIT*, *PROGRAM*, *ENTER*, *INFO*
    - *RED*, *GREEN*, *YELLOW*, *BLUE*, *LIVE_ZOOM*, *CC*, *PLAY*, *PAUSE*, *REWIND*, *FASTFORWARD*
  - Most probably there are also other values possible which i didn't find yet (like settings or voice command), you can try typing some other values and if you find some that work then please let me know
- `remoteSequenceButtons` [optional] 
Wheter the remote sequence buttons service is enabled. This allows to run a sequence of remote control button presses. **Default: "" (disabled)**
  - Set an array of objects as the value. An object needs to have the following properties:
    - *sequence* - an array of remote control keys. For possible values see `remoteControlButtons` above,
    - *name* - an optional name, 
    - *interval* - an optional interval beetwen sequence actions. Default is 500ms
- `soundOutputButtons` [optional] 
Wheter the sound output buttons service is enabled. This allows to switch between sound outputs on the TV. **Default: "" (disabled)**
  - Set an array of sound outputs as the value. Example values are:
    - *tv_speaker* - internal tv speaker,
    - *external_optical* - optical audio, 
    - *external_arc* - hdmi arc, 
    - *lineout* - line out, 
    - *headphone* - headphones, 
    - *tv_external_speaker* - tv speaker and optical, 
    - *tv_speaker_headphone* - tv speaker and headphones
    - *bt_soundbar* - bluetooth soundbar and bluetooth devices
   - Depending on the TV and connected devices to the TV there might also be other values. In that case just switch sound outputs on the TV and check the homebridge log.
- `infoButtonAction` [optional]
The action (button press emulation) for the info button press on the remote control in iOS control center. For possible values see the `remoteControlButtons` property above. **Default: "INFO"**
  
## Troubleshooting
If you have any issues with the plugin or TV services then you can run homebridge in debug mode, which will provide some additional information. This might be useful for debugging issues. 

Homebridge debug mode:
```sh
homebridge -D
```

## Special thanks
[lgtv2](https://github.com/hobbyquaker/lgtv2) - the Node.js remote control module for LG WebOS smart TVs.

[homebridge-webos2](https://github.com/zwerch/homebridge-webos2) - the basic idea for the plugin.

[HAP-NodeJS](https://github.com/KhaosT/HAP-NodeJS) & [homebridge](https://github.com/nfarina/homebridge) - for making this possible.
