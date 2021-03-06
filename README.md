zabbix-weather
==============

A script and zabbix template to track weather data pulled from weather underground


Prerequisites
-------------

* An account for API access on weatherunderground.com
* Admin access for at least one host on a Zabbix server (works with Zabbix versions 2.2 and 2.4)
* shell access to the host
* python 2 installed and working on the host machine (tested with Python 2.7)

This has been tested with linux hosts only. The python should be portable enough to run in windows, but the file     paths have to be changed.  The file used is only a temporary file, so it should not matter where it is saved as long  as the user running the script has write access to the directory in which the file resides.


First Time Setup
----------------

* Sign up for API access at weather underground (500 API calls per day are free) at http://www.wunderground.com/weather/api
* Take note of your API key.


Installing and Updating
-----------------------

Installation and Updating must be done manually for now.

0. Place the weather.py on a host of your choice where the zabbix agent can execute it
The default location is `/etc/zabbix/scripts`

0. Make it executable `chmod +x /etc/zabbix/scripts/weather.py`

0. Place the default config file `weather.cfg` in the same directory as the script `/etc/zabbix/scripts`

0. Open the config file and replace the default options with your own values. The following values must be changed:

```
api_key = <API_KEY>
zabbix_server = <ZABBIX_SERVER>
zabbix_hostname = <ZABBIX_HOSTNAME>
```
For example, replace `<API_KEY>` with your actual API key, such as `cd164d6297962648`.

Config file description:
```
api_key = weather underground API key
location = The location to collect weather data for.
zabbix_server =  Hostname or IP of your zabbix server
zabbix_hostname = Zabbix Host name of the host with the weather script installed
zabbix_sender = Complete filename and path of the zabbix_sender program
```

0. The location must be something that weather underground supports.  Examples include cities in State/City form such as `AK/Talkeetna`, or actual weather station IDs such as `zmw:00000.1.71123`.  Some foreign cities are supported as well, such as `Canada/Ontario/Atikokan`. For larger cities with many stations, it is best to use the station ID closest to your location.

0. Change the `zabbix_sender` option if the zabbix_sender program is not in your path.  For example, change it to `/opt/zabbix/bin/zabbix_sender` if the zabbix_sender binary is installed at `/opt/zabbix/bin`

0. Python2 is required and is expected to be installed at `/usr/bin/python2`. Modify the first line of the script if your python2 is named just python or resides in a different path.

0. Run the script from the command line to ensure that it works properly.  An example of a successful run is below
```
Using config file: /etc/zabbix/scripts/weather.cfg
Read these settings from config file:
api_key: cd47c13676331651
location: CO/Fort_Collins
zabbix_server: 127.0.0.1
zabbix_hostname: thebean-media
zabbix_sender: zabbix_sender

Current Conditions:
Observed at: Wed, 25 Nov 2015 18:16:54 -0700
Location: Hanna Farm, Fort Collins
Weather: Overcast
Temp: 27.9°F (-2.3°C)
Humidity: 90%
Pressure: 30.02in +
Visibility: 10.0mi
Precipitation: 0.00in last hour, 0.00in today
Wind: From the WNW (292°) at 2.7mph, gusting to 5.8mph
Wind Chill: 28°F (-2°C)

Sending values to zabbix...
info from server: "processed: 17; failed: 0; total: 17; seconds spent: 0.000180"
sent: 17; skipped: 0; total: 17
```

0. Import the weather template into Zabbix `Configuration -> Templates -> Import`. This template only includes Applications, Items, Triggers and Graphs.  If you are using Zabbix 2.2, use template `zabbix_weather_template_zbx2.2.xml`, if you are using Zabbix 2.4, use template `zabbix_weather_template.xml`.  The templates have not been tested with other Zabbix versions.

0. Add a host to be in the template.  This should be the same host as where the script is installed.

0. Modify the Item `Collect weather data` with the full path of the script if it is installed to a different directory.  In the Zabbix web front end, go to `Configuration -> Templates -> Weather -> Items`. Modify the `Key` field to match your path.  Defult is: `system.run[/etc/zabbix/scripts/weather.py,wait]`

0. Look at the host's `latest data` in the Zabbix front end.  Go to `Monitoring -> Latest Data`.  It should be populated with weather data in a few minutes.  The values will appear under a heading called `Weather`.

0. Any errors from the script will be in the latest data of the `Collect weather data` item.

