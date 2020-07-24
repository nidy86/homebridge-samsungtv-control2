# Homebridge-SamsungTV-Control

This plugin adds SamsungTVs available in your local network to homebridge. Just install the plugin in the Homebridge UI in the Plugin tab, turn on all your Samsung TV's so they can be discovered and restart homebridge.

# Customize devices

After you started homebridge you should see the device names with their usn in the log. To customize a device, add an object with the usn (including `uuid:`) and any option you want to modify to the devices list under the `SamsungTVControl` platform. You could e.g. change the initial name like so:

```json
{
  "platform": "SamsungTVControl",
  "devices": [
    {
      "usn": "uuid:a7001fbe-c776-11ea-87d0-0242ac130003",
      "name": "Bedroom TV"
    }
  ]
}
```

# Pairing

Younger TV's (2014+) might require being paired before the plugin is able to remote control them. The plugin comes with a command line utility that among other things can run the pairing procecure and give you a token you then add to the configuration. If pairing is required and you didn't add the token to your configuration yet, the plugin should log the two methods you can try to pair your TV. When `samsung-ctrl` finished pairing it tries to send the mute key to your tv for you to be able to check if it worked. If you didn't observe mute being toggled on your TV you might try the second method.

```json
{
  "platform": "SamsungTVControl",
  "devices": [
    {
      "usn": "uuid:a7001fbe-c776-11ea-87d0-0242ac130003",
      "token": "eyJzZXNzaW9uSWQiOjQyLCJhZXNLZXkiOiJZb3UgcmVhbGx5IGRlY29kZWQgdGhpcz8g8J+YiSJ9",
      "name": "Bedroom TV"
    }
  ]
}
```

# Add TV's to home app

There is a homekit limitation that allows only one TV per bridge. Therefore each TV will be exposed as external accessory and will not show up when only the homebridge-bridge was added. To add each of your Samsung TV's

1. Open the Home App
2. Type `+` in the top right corner to add a device
3. Then click on **Don't Have a Code or Can't scan?**
4. The found TV should appear under **Nearby Accessories** ... click on it
5. Use the pin that you configured under `config > bridge > pin`

# Turning on a TV

Some Samsung TV's (actually all I was able to test) turn off their network card when being turned off completely. Therefore these models cannot be turned on again by this plugin since they are just not reachable over the network. However some newer models might support Wake-on-LAN or WoWLAN which this plugin tries to use to turn your TV back on.

# Custom inputs

By default only TV will be added as input source which triggers sending the TV key to e.g. get back to the tuner when hdmi was previously selected. The `inputs` property allows you to add extra inputs to e.g. switch channels, start sleep mode or open an app (if supported). Each input requires a `name` which represents the input name and a `keys` property which is either a string containing numbers only, a comma seperated list of keys or a single app name from the list below. The keys will then be send to the tv with a delay of `500ms` (or what you configured) in between.

Here is an example:

```json
{
  "platform": "SamsungTVControl",
  "devices": [
    {
      "usn": "uuid:a7001fbe-c776-11ea-87d0-0242ac130003",
      "name": "Bedroom TV",
      "inputs": [
        {
          "name": "HDMI",
          "keys": "hdmi"
        },
        { "name": "ZDF HD", "keys": "102" },
        {
          "name": "Sleep 30m",
          "keys": "tools,down*3,enter,down,enter,return"
        },
        {
          "name": "Open YouTube",
          "keys": "YouTube"
        }
      ]
    }
  ]
}
```

* [List of apps](https://github.com/Toxblh/samsung-tv-control/blob/HEAD/src/apps.ts) - The support for opening apps is unclear. These however won't work when paired with `pair1` definitely.
* [List of keys](https://github.com/Toxblh/samsung-tv-control/blob/master/src/keys.ts) - You just have to test which keys work on your tv and which don't since this differs stronlgy between all the models. Casing doesn't matter and you can also leave away `KEY_` for convenience as seen in the example. If you need to send a key multiple times in row you can add e.g. `*3` to send it three times.

**Note: Unfortunatelly when editing inputs it might be required to remove the tv from homekit and add it again after homebridge restarted for the home app to see the changes.**
