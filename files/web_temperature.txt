#!/usr/bin/python

import RPi.GPIO as GPIO
import sys
import temperature

#
# Replace the blank below with your device file
#
DEVICE_FILE = "/sys/devices/w1_bus_master1/10-0008032719e2/w1_slave"
MY_DEVICE_FILE = "/home/chrism/Desktop/Backup/temp_log"

HTML_START = """
<html>
<head>
	<meta http-equiv="refresh" content="5">
	<title>Current Temperature</title>
</head>
<body>
"""

HTML_END = """
</body>
</html>
"""



#Write 'temp' to the file called "temp_log"
def write_temp(temp, temp_log):
    f = open(temp_log, "w")
    f.write(str(temp))
    f.close()
                                                                                                                
#return the value contained in the file called "temp_log"
def read_temp(temp_log):
    f = open(temp_log, "r")
    temp = f.read()
    f.close()
    return float(temp)

def application(env, start_response):   
    YELLOW = 27
    GREEN = 22
    RED = 18
    
    GPIO.setmode(GPIO.BCM)

    GPIO.setup(YELLOW, GPIO.OUT)
    GPIO.setup(GREEN, GPIO.OUT)
    GPIO.setup(RED, GPIO.OUT)

    status = "200 OK"

    headers = [("Content-type", "text/html")] # no UTF-8 encoding (yet)
    start_response(status, headers)

    body = HTML_START # no square brackets
                                                                                                                                                
    try:
        temp = temperature.read_temp(DEVICE_FILE)
        body += "It is currently " + str(temp) + "&degC\n"
        oldTemp = read_temp(MY_DEVICE_FILE)
        if (oldTemp == temp):
            GPIO.output(YELLOW, 1)
        else:
            GPIO.output(YELLOW, 0)

        if (oldTemp < temp):
            GPIO.output(GREEN, 1)
        else:
            GPIO.output(GREEN, 0)
            
        if (oldTemp > temp):
            GPIO.output(RED, 1)
        else:
            GPIO.output(RED, 0)
            
        write_temp(temp, MY_DEVICE_FILE)

    except Exception as details:
        sys.stdout.write(str(details))
        body += "Error\n" + str(details) # no append

    finally:
        body += HTML_END
        GPIO.output(YELLOW, 0)
        GPIO.output(GREEN, 0)
        GPIO.output(RED, 0)
        GPIO.cleanup()
        
    yield bytes(body, encoding='utf-8')
    
    

    
