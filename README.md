# Kaleidoscope-EEPROM-Settings

[![Build Status][travis:image]][travis:status]

 [travis:image]: https://travis-ci.org/keyboardio/Kaleidoscope-EEPROM-Settings.svg?branch=master
 [travis:status]: https://travis-ci.org/keyboardio/Kaleidoscope-EEPROM-Settings

To be able to reliably store persistent configuration in `EEPROM`, we need to be
able to split up the available space for plugins to use. We also want to make
sure that we notice when the `EEPROM` contents and the firmware are out of sync.
This plugin provides the tools to do that.

It does not guard against errors, it merely provides the means to discover them,
and let the firmware Sketch handle the case in whatever way it finds reasonable.
It's a building block, and not much else. All Kaleidoscope plugins that need to
store data in `EEPROM` are encouraged to make use of this library.

## Using the plugin

There are a few steps one needs to take to use the plugin: we must first
register it, then either let other plugins request slices of `EEPROM`, or do so
ourselves. And finally, seal it, to signal that we are done setting up. At that
point, we can verify whether the contents of the `EEPROM` agree with our
firmware.

```c++
#include <Kaleidoscope.h>
#include <Kaleidoscope-EEPROM-Settings.h>

static uint16_t settingsBase;
static struct {
  bool someSettingFlag;
} testSettings;

KALEIDOSCOPE_INIT_PLUGINS(EEPROMSettings, /* Other plugins that use EEPROM... */);

void setup () {
  Kaleidoscope.setup();

  settingsBase = EEPROMSettings.requestSlice(sizeof(testSettings));

  EEPROMSettings.seal();

  if (!EEPROMSettings.isValid()) {
    // Handle the case where the settings are out of sync...
    // Flash LEDs, for example.

    return;
  }

  EEPROM.get(settingsBase, testSettings);
}
```

## Plugin methods

The plugin provides the `EEPROMSettings` object, which has the following methods:

### `requestSlice(size)`

> Requests a slice of the `EEPROM`, and returns the starting address (or 0 on
> error, including when the request arrived after sealing the layout).
>
> Should only be called **before** calling `seal()`.

### `default_layer([id])`

> Sets (or returns, if called without an ID) the default layer. When the
> keyboard boots up, it will automatically switch to the configured layer - if
> any.
>
> This is the Focus counterpart of the `default_layer()` method documented
> above.

### `seal()`

> Seal the `EEPROM` layout, so no new slices can be requested. The CRC checksum
> is considered final at this time, and the `isValid()`, `crc()`, `used()` and
> `version()` methods can be used from this point onwards.
>
> If not called explicitly, the layout will be sealed automatically after
> `setup()` in the sketch finished.

### `update()`

> Updates the `EEPROM` header with the current status quo, including the version
> and the CRC checksum.
>
> This should be called when upgrading from one version to another, or when
> fixing up an out-of-sync case.

### `isValid()`

> Returns whether the `EEPROM` header is valid, that is, if it has the expected
> CRC checksum.
>
> Should only be called after calling `seal()`.

### `invalidate()`

> Invalidates the `EEPROM` header. Use when the version does not match what the
> firmware would expect. This signals to other plugins that the contents of
> `EEPROM` should not be trusted.

### `version([newVersion])`

> Sets or returns the version of the `EEPROM` layout. This is purely for use by
> the firmware, so it can attempt to upgrade the contents, if need be, or alert
> the user in there's a mismatch. Plugins do not use this property.
>
> Should only be called after calling `seal()`.

### `crc()`

> Returns the CRC checksum of the layout. Should only be used after calling
> `seal()`.

### `used()`

> Returns the amount of space requested so far.
>
> Should only be used after calling `seal()`.

## Focus commands

The plugin provides two - optional - [Focus][FocusSerial] command plugins:
`FocusSettingsCommand` and `FocusEEPROMCommand`. These must be explicitly added
to `KALEIDOSCOPE_INIT_PLUGINS` if one wishes to use them. They provide the
following commands:

 [FocusSerial]: https://github.com/keyboardio/Kaleidoscope-FocusSerial

### `settings.defaultLayer`

> Sets or returns (if called without arguments) the ID of the default layer. If
> set, the keyboard will automatically switch to the given layer when connected.
> Setting it to `255` disables the automatic switching.

### `settings.crc`

> Returns the actual, and the expected checksum of the settings.

### `settings.valid?`

> Returns either `true` or `false`, depending on whether the sealed settings are
> to be considered valid or not.

### `settings.version`

> Returns the (user-set) version of the settings.

### `eeprom.contents`

> Without argument, displays the full contents of the `EEPROM`, including the
> settings header.
>
> With arguments, the command updates as much of the `EEPROM` as arguments are
> provided. It will discard any unnecessary arguments.

### `eeprom.free`

> Returns the amount of free bytes in `EEPROM`.

## Dependencies

* [Kaleidoscope-FocusSerial][FocusSerial]

## Further reading

Starting from the [example][plugin:example] is the recommended way of getting
started with the plugin.

  [plugin:example]: https://github.com/keyboardio/Kaleidoscope-EEPROM-Settings/blob/master/examples/EEPROM-Settings/EEPROM-Settings.ino
