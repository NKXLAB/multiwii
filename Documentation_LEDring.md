
---

Full Instructions for setting up the LEDring are here:
http://code.google.com/p/ledring/w/list

---



# Configuring MultiWii for LEDring - Supported releases #


---

## MultiWii 2.1 ##

### option 1 - standard MultiWii ledring functionality ###
Find and enable //#define LED\_RING in config.h
Upload the new sketch to your MultiWii controller

Ensure the following is set in the sketch uploaded to the LEDring board:
```
//#define MultiWii_I2C_v2
#define MultiWii_I2C_v1
```


### option 2 - enhanced MultiWii ledring functionality ###
Replace MultiWii led.ino with alternative led.ino from:  http://code.google.com/p/ledring/downloads/list

Find and enable //#define LED\_RING in config.h
Upload the new sketch to your MultiWii controller

Ensure the following is set in the sketch uploaded to the LEDring board:
```
#define MultiWii_I2C_v2
// #define MultiWii_I2C_v1 
```


---

## MultiWii 2.0 ##

### standard MultiWii ledring functionality ###
Find and enable //#define LED\_RING in config.h
Upload the new sketch to your MultiWii controller

Ensure the following is set in the sketch uploaded to the LEDring board:
```
//#define MultiWii_I2C_v2
#define MultiWii_I2C_v1
```


---

## Pre MultiWii 2.0 ##
Not supported

---
