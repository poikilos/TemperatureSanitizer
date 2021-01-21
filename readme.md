# TemperatureSanitizer
TemperatureSanitizer detects when a minimum temperature is sustained for a minimum time. This program uses temperusb (a.k.a. temper) or ccwienk's temper for a wider range of devices, and default settings are for killing bedbugs:
- 120 minutes (though 90 minutes is considered minimum)
- 120 degrees fahrenheit (though 118 is considered minimum).

Bedbug ovens are available from [ZappBug on Amazon](https://www.amazon.com/s?k=ZappBug).

For sustaining a minimum temperature, try to place the temperature sensor in the coolest spot (such as the innermost part of the most insulated part of the load).

If the temperusb script is not compatible with your device, you can use read-loop.py instead if you install temper (not the pypi temper package which is something else--run bad_temper.py to see if that is present!). Alternatively, use get_temp.sh to install dependencies and display the temperature or use read-loop.sh on a GNU+Linux system to install dependencies in a virtual environment and run the loop.


## Reference temperatures
According to [Temperature and Time Requirements for Controlling Bed Bugs (Cimex lectularius) under Commercial Heat Treatment Conditions ](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4553552/) by Stephen A. Kells* and Michael J. Goblirsch, bedbug eggs can survive 71.5 min at 48 °C. Therefore, go longer or higher than that to be sure. Note that your temperature sensor may not be in the coolest spot (attempt to place it in a cool spot). The lethal temperature "for eggs was 54.8 °C" in the study, while the sub-lethal temperatures took longer. Reaching the acute temperature may be impossible in some cases of a large bug oven or house heat treatment, and the acute temperature is more likely to damage your property or bedbug oven. Therefore, the lower temperature and longer times are the defaults for this program, as listed at the beginning of this document.

Conversions table for reference temperatures:
- 113° F = 45° C
- 118.4° F = 48° C
- 120° F = 48.8889° C
- 130.64 F = 54.8 C


## Requirements
The scripts will detect missing dependencies and instruct you how to correct that
- A TEMPer compatible USB thermometer (can be ordered online)
  - TEMPerV1 for temperusb
- [github.com/ccwienk/temper](https://github.com/ccwienk/temper) or another fork of urwen/temper (NOT the pypi package which is not for temperature sensors) or temperusb
- The temperusb library for Python. Installing temperusb requires the temperusb whl file or an internet connection in order to follow those instructions displayed by this program.


## Usage
* Make sure you place the temperature sensor in a "cold spot," such as (if you are using this program to monitor a bedbug oven) inside a book in the middle of your load.
* Change the following settings by editing TemperatureSanitizer or utilizing tempermgr as a library similarly to how TemperatureSanitizer.py does. A settings dictionary passed to the TemperMgr constructor can include the following settings (the defaults are below):
  ```python
settings = {}
settings['target'] = 120
settings['scale'] = "fahrenheit"
settings['interval'] = 60  # the interval in seconds (get one average or other specified stat per interval)
settings['minTime'] = 120 * 60  # the total desired time at the temperature
```
  * and, if you want to chill instead of bake, you can edit the following settings:
      ```python
    settings['compareOp'] = ">="
    settings['useStat'] = "average"
```
  * `settings['compareOp']` can be ">=", ">", "<=", or "<"
  * `settings['useStat']` can be minimum (or min), maximum (or max), or average (or avg) -- for example, for a chill (using < or <=), min or average would be appropriate comparisons to ensure that a temperature reading never was above the desired temperature; but for baking (using >= or >), min or average would be recommended, where min would ensure a temperature reading didn't dip below desired temperature.

* How the settings are used by the program:
  * scale can be "fahrenheit" or "celcius".
  * The interval may need to be at least a few seconds for accuracy, though an interval of 1 can be used if you don't mind having all that data, and your sensor is accurate enough for that type of use (trusting a single reading as part of your raw data).
  * The temperature is always checked every second. This is not configurable, but only the minimum and avarage are displayed from each interval, which is a group of these secondly readings.
  * The seconds will start counting when the minimum temperature of the interval is at least at the settings['target'] temperature. If the total number of seconds of bake time is reached (when there was a consecutive set of spans where the minimum never was below settings['target'] temperature), the program will end and show a summary (including a list of temperatures, each of which is the minimum temperature of each consecutive interval that had the desired temperature as its minimum). Otherwise, the program will continue collecting data indefinitely.
* Run:
```bash
# cd to the directory where you saved the py file, then:
sudo python TemperatureSanitizer
# Or, run without sudo if you have changed the permissions of your USB device.
# NOTE: sometimes that setting is forgotten when the device is unplugged and
# reinserted, even if same port is used.
```


## Known Issues
See [github.com/poikilos/TemperatureSanitizer/issues](https://github.com/poikilos/TemperatureSanitizer/issues).

When reporting issues, provide the USB id of your device, such as via
`lsusb` on Linux (run it before and after inserting the device to see
what appears, as it may not have any name by the id string).

## Developer Notes
* The settings are hard-coded, in the User Settings region of the py file.
* When a span is complete, the span is added to the bake only if the minimum temperature of the span meets the desired temperature.

### Discarded plans
#### TEMPered

#### Include or compile hid-query
Where `/dev/hidraw4` is the correct device (could be any number--usually last one listed via `ls /dev | grep hidraw`),
run `sudo hid-query /dev/hidraw1 0x01 0x80 0x33 0x01 0x00 0x00 0x00 0x00`
the response is 8 bytes such as:
`80 80 0a fc  4e 20 00 00`
where 0a fc is an integer (2825 in this case). Divide that by 100 to get the temperature.

An example of parsing the output is at
<https://github.com/padelt/temper-python/issues/84#issuecomment-393930865>
(possibly auto-install TEMPered such as with script below)
```bash
if [ ! `which hid-query` ]; then
  cd $HOME
  if [ ! -d Downloads ]; then mkdir Downloads; fi
  cd Downloads
  if [ ! -d TEMPered ]; then
    git clone https://github.com/edorfaus/TEMPered.git
    cd TEMPered
  else
    cd TEMPered
    git pull
  fi
  if [ ! -d build ]; then mkdir build; fi
  cd build
  cmake ..
  sudo make install
  if [ ! -d /usr/local/lib ]; then mkdir -p /usr/local/lib; fi
  sudo cp libtempered/libtempered.so.0 /usr/local/lib/
  sudo cp libtempered-util/libtempered-util.so.0 /usr/local/lib/
  sudo ln -s /usr/local/lib/libtempered.so.0 /usr/local/lib/libtempered.so
  sudo ln -s /usr/local/lib/libtempered-util.so.0 /usr/local/lib/libtempered-util.so
fi
```

Find hid like:
```bash
  #LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib tempered
  # previous line doesn't work for some reason so:
  hid-query --enum
  # lists 2 for some reason:
  # /dev/hidraw0 : 413d:2107 interface 0 : (null) (null)
  # /dev/hidraw1 : 413d:2107 interface 1 : (null) (null)
```

A script such as readTEMPer-driverless-withdate.sh (by jbeale1 from link above) could be made like:
```bash
#!/bin/bash
OUTLINE=`sudo ./hid-query /dev/hidraw3 0x01 0x80 0x33 0x01 0x00 0x00 0x00 0x00|grep -A1 ^Response|tail -1`
OUTNUM=`echo $OUTLINE|sed -e 's/^[^0-9a-f]*[0-9a-f][0-9a-f] [0-9a-f][0-9a-f] \([0-9a-f][0-9a-f]\) \([0-9a-f][0-9a-f]\) .*$/0x\1\2/'`
HEX4=${OUTNUM:2:4}
DVAL=$(( 16#$HEX4 ))
bc <<< "scale=2; $DVAL/100"
```

The script above would be used like:
```bash
while [ true ]; do
  temp=`./readTEMPer-driverless-withdate.sh`
  echo $(date +"%F %T") " , " $temp
  sleep 14
done
```
