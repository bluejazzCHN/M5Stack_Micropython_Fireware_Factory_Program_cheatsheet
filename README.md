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

*** The file location is in the root directory of the device. ***

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
```
Three important methods of APP Class:
install
start
run
```

```python

    #run method analysis

    async def run(self):
        self.i2c0 = I2C(0, scl=Pin(15), sda=Pin(13), freq=100000)
        self._kb_status = False
        if 0x5F in self.i2c0.scan():
            self._kb = CardKBUnit(self.i2c0)
            self._event = KeyEvent()
            self._kb_status = True

        #the most important input device for Dial is rotary.
        # rotary init

        rotary = Rotary()
        self._bar and self._bar.start()

        if self._launcher:
            self._app_selector.select(self._launcher)
            self._launcher.start()

        last_touch_time = time.ticks_ms()
        while True:
            M5.update()

            #Dial touch screen process 
            if M5.Touch.getCount() > 0:
                detail = M5.Touch.getDetail(0)
                cur_time = time.ticks_ms()
                if cur_time - last_touch_time > 150:
                    if detail[9]:  # isHolding
                        pass
                    else:
                        M5.Speaker.tone(3500, 50)
                        x = M5.Touch.getX()
                        y = M5.Touch.getY()
                        app = self._app_selector.current()
                        if hasattr(app, "_click_event_handler"): 
                            await app._click_event_handler(x, y, self)
                    last_touch_time = time.ticks_ms()

            #Switch between apps installed in the Framework based on the rotary status
            if rotary.get_rotary_status():
                app = self._app_selector.current()
                app.stop()
                if rotary.get_rotary_increments() > 0:
                    M5.Speaker.tone(7000, 20)
                    app = self._app_selector.next()
                else:
                    M5.Speaker.tone(6000, 20)
                    app = self._app_selector.prev()
                app.start()

            #if you connect KB_Card unit , default support it
            if self._kb_status:
                if self._kb.is_pressed():
                    M5.Speaker.tone(3500, 50)
                    self._event.key = self._kb.get_key()
                    self._event.status = False
                    await self.handle_input(self._event)

            await asyncio.sleep_ms(10)
```

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

```
App.py contains the base class of the app, class AppBase, and a function class of the app, class AppSelector

Three important methods of APP Class:
on_ready
on_run
on_view
```

