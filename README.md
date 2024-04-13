# M5Stack_Micropython_Fireware_Factory_Program_cheatsheet
There are many M5Stack devices. The M5Dial is taken as an example for analysis

## Precondition

1. [uiflow-micropython](https://github.com/m5stack/uiflow-micropython)
2. Device, for example dial, core, core2 and so on.
3. VScode
4. Git 

## Program loading flow 

```
    boot.py --> class Startup (startup function) --> class Framework (install and start function)
                        |                                     |
                        |                                     |
                        |                                     |   
                        |--------------------------> class apps <-- class app ( base is derived)
```
