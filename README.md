# M5Stack_Micropython_Fireware_Factory_Program_cheatsheet
There are many M5Stack devices. The M5Dial is taken as an example for analysis

## Precondition

1. [uiflow-micropython](https://github.com/m5stack/uiflow-micropython)
2. Device, for example dial, core, core2 and so on.
3. VScode
4. Git 

## Program loading flow 

```
    boot.py --> class Startup (startup function) --> class Dial_Startup (Startup function) --> class Framework (install and start function)
                                                            |                        |
                                                            |                        |
                                                            |                        |   
                                                            |-----------------> class apps <-- class app ( base is derived)
```

## Flow description

### boot.py

```python
from startup import startup 
startup(boot_option, NETWORK_TIMEOUT)
```

### class startup

```python

#Load the Startup class of each device based on the board.id. for M5Dial this class is Dial_Startup.

elif board_id == M5.BOARD.M5Dial:
    from .dial import Dial_Startup    
    dial = Dial_Startup()

    # call dial class startup function

    dial.startup(ssid, pswd, timeout)

```

### class Dial_Startup

```python
# 1. Define the framework

from .framework import Framework
fw = Framework()

#2. Initialize the app (DevApp, RunApp, ListApp, SettingsApp are specific app definitions and exist in the apps directory in the diagram).

from .apps.settings import SettingsApp, WiFiSetting
from .apps.dev import DevApp
from .apps.app_run import RunApp
from .apps.app_list import ListApp
wifi_app = WiFiSetting(None, data=self._wlan)
setting_app = SettingsApp(None, data=self._wlan)
dev_app = DevApp(None, data=self._wlan)
run_app = RunApp(None, data=self._wlan)
list_app = ListApp(None, data=self._wlan)

#3. Install apps (one of which is the launcher)

fw.install_launcher(dev_app)
fw.install(wifi_app) fw.install(setting_app)
fw.install(dev_app)
fw.install(run_app)
fw.install(list_app)

#4. Launch framework and run all apps

fw.start()

```

### class Framework
### class apps
```
The following is the official implementation of the M5 sample app:
DevApp --dev.py,
RunApp --app_run.py,
ListApp --app_list.py,
SettingsApp --settings.py,
EzDataApp --ezdata.py,
StatusBarApp --status_bar.py,
WiFiSetting --settings.py
```
### class App
 

