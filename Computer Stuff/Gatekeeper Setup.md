x# Gatekeeper Setup
## libnfc
`curl -L https://github.com/nfc-tools/libnfc/releases/download/libnfc-1.8.0/libnfc-1.8.0.tar.bz2 | tar xvj`
Arch requires both libusb and libusb-compat
```bash
./configure --prefix=/usr
make
sudo make install
```

## libfreefare
`curl -L https://github.com/nfc-tools/libfreefare/releases/download/libfreefare-0.4.0/libfreefare-0.4.0.tar.bz2 | tar xvj`
```bash
./configure --prefix=/usr
make
sudo make install
```


## configuration
Add `/usr/local/lib` to `/etc/ld.so.conf` so you can find the newly installed libraries

Place the following in `/etc/nfc/libnfc.conf`:
```ini
# Allow device auto-detection (default: true)
# Note: if this auto-detection is disabled, user has to set manually a device
# configuration using file or environment variable
#allow_autoscan = true

# Allow intrusive auto-detection (default: false)
# Warning: intrusive auto-detection can seriously disturb other devices
# This option is not recommended, user should prefer to add manually his device.
#allow_intrusive_scan = false

# Set log level (default: error)
# Valid log levels are (in order of verbosity): 0 (none), 1 (error), 2 (info), 3 (debug)
# Note: if you compiled with --enable-debug option, the default log level is "debug"
#log_level = 1

# Manually set default device (no default)
# To set a default device, you must set both name and connstring for your device
# Note: if autoscan is enabled, default device will be the first device available in device list.
device.name = "PN532 UART"
device.connstring = "pn532_uart:/dev/ttyUSB0"
```

Add yourself to the `uucp` group so you can read/write `/dev/ttyUSB0`.

`nfc-list` should now show your nfc reader once you plug it in.




## NOTES
Real tag UID: 04581b92d65c80
