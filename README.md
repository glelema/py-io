
'''ESP8266'''

'''Each part runs independently. Code needs to be stopped by ctrl+c, because
a lot of answers create an infinite loop.'''

'''1. Make​ ​a​ ​red​ ​led​ ​blink​ ​with​ ​a​ ​frequency​ ​of​ ​1Hz​ ​(500ms​ ​on/500ms​ ​off)​ ​as​ ​long​ ​as
you​ ​press​ ​a​ ​button.'''

import machine; import time;

led = machine.Pin(0, machine.Pin.OUT);
button = machine.Pin(12, machine.Pin.IN, machine.Pin.PULL_UP)

def btnpres1():
    while button.value() == 0:
        led.on(); time.sleep(0.5); led.off(); time.sleep(0.5);
    return

time.sleep(0.2); btnpres1();

'''2.​ ​Connect​ ​a​ ​button​ ​and​ ​all​ ​the​ ​three​ ​5mm​ ​colored​ ​LEDs​ ​(in​ ​this​ ​order:​ ​green,
yellow​ ​and​ ​red)​ ​to​ ​the​ ​Feather​ ​Huzzah.​ ​​Remember​ ​that​ ​for​ ​each​ ​led​ ​you​ ​should
use​ ​a​ ​resistor​ ​as​ ​well.​​ ​Write​ ​a​ ​micropython​ ​program​ ​that​ ​each​ ​time​ ​you​ ​press​ ​the
button​ ​will​ ​light​ ​up​ ​another​ ​LED.​ ​When​ ​the​ ​program​ ​first​ ​starts​ ​only​ ​the​ ​green
LED​ ​is​ ​lit.​ ​After​ ​the​ ​first​ ​button​ ​press​ ​only​ ​the​ ​yellow​ ​LED​ ​is​ ​lit.​ ​After​ ​second
button​ ​press​ ​only​ ​the​ ​RED​ ​led​ ​is​ ​lit.​ ​After​ ​the​ ​third​ ​button​ ​press​ ​the​ ​green​ ​LED​ ​is
lit.​ ​So​ ​on,​ ​after​ ​each​ ​button​ ​press,​ ​the​ ​next​ ​LED​ ​will​ ​light​ ​up​ ​(in​ ​a​ ​cyclical​ ​fashion).'''

import machine; import time;
# assign 3 variables to 3 pins controlling 3 LEDs
ledg = machine.Pin(0, machine.Pin.OUT); ledy = machine.Pin(16, machine.Pin.OUT);
ledr = machine.Pin(2, machine.Pin.OUT); i=1;

def btnpres2():
    while button.value() == 0:
        if i%2 == 0: ledy.on(); i+=1;
        elif i%3 == 0: ledr.on(); i+=1;
        else: ledg.on(); i+=1;
    return

time.sleep(0.2); btnpres2();

'''3. Temperature and 3 LEDs'''

import machine; import time;

ledg = machine.Pin(0, machine.Pin.OUT); ledy = machine.Pin(16, machine.Pin.OUT);
ledr = machine.Pin(2, machine.Pin.OUT);

i2c = machine.I2C(scl=machine.Pin(5), sda=machine.Pin(4));
data = bytearray(2); i2c.readfrom_mem_into(address, temp_reg, data);

def temp_c(data): # temp reading function
    value = data[0] << 8 | data[1]
    temp = (value & 0xFFF) / 16.0
    if value & 0x1000:
        temp -= 256.0
    return temp

while True: # relevant LED on, rest off
    if temp_c(data) < 22: ledg.on(); ledy.off(); ledr.off(); time.sleep(0.2);
    elif 22 <= temp_c(data) < 24: ledy.on(); ledg.off(); ledr.off(); time.sleep(0.2);
    else: ledr.on(); ledg.off(); ledy.off(); time.sleep(0.2);

'''4. Neopixel temperature. Way neopixel LED are designed is serial connection.
So they can form strips, screens and other shapes. Then they are controlled
by 1 pin in and 1 pin out.'''

import machine, esp, time; # neopixel module won't import on this board

pixel_pin   = machine.Pin(15, machine.Pin.OUT);  # Pin connected to the NeoPixels.
pixel_data = bytearray(3*3);

# thermometer variable
i2c = machine.I2C(scl=machine.Pin(5), sda=machine.Pin(4));
data = bytearray(2); i2c.readfrom_mem_into(address, temp_reg, data);

def temp_c(data): # temp reading function
    value = data[0] << 8 | data[1]
    temp = (value & 0xFFF) / 16.0
    if value & 0x1000:
        temp -= 256.0
    return temp

while True: # neopixel relevant LED on, rest off
    if temp_c(data) < 22:
        pixel_data = bytearray ([128,0,0,0,0,0,0,0,0]);
        esp_neopixel.write(pixel_pin, pixel_data, True);
        time.sleep(0.2);
    elif 22 <= temp_c(data) < 24:
        pixel_data = bytearray([0, 0, 0, 64, 64, 0, 0, 0, 0]);
        esp_neopixel.write(pixel_pin, pixel_data, True);
        time.sleep(0.2);
    else:
        pixel_data = bytearray([0, 0, 0, 0, 0, 0, 0, 128, 0]);
        esp_neopixel.write(pixel_pin, pixel_data, True);
        time.sleep(0.2);

'''5. Potentiometer / resistor, 3008 board'''

'''​In​ ​this​ ​assignment​ ​we’ll​ ​connect​ ​an​ ​analog​ ​input​ ​to​ ​the​ ​Feather​ ​Huzzah​ ​with
the​ ​help​ ​of​ ​the​ ​​MCP3008​ ​ADC​ ​conversion​ ​chip​.​ ​​ ​The​ ​analog​ ​input​ ​is​ ​provided​ ​by
the​ ​potentiometer​ ​available​ ​in​ ​your​ ​kit.​ ​Write​ ​a​ ​micropython​ ​program​ ​that​ ​can
control​ ​the​ ​intensity​ ​of​ ​light​ ​from​ ​a​ ​neopixel​ ​LED​ ​with​ ​the​ ​help​ ​of​ ​the
potentiometer​ ​connected​ ​to​ ​channel​ ​3​ ​of​ ​the​ ​MCP3008​ ​chip.​ ​By​ ​turning​ ​the
potentiometer​ ​you​ ​should​ ​be​ ​able​ ​to​ ​control​ ​the​ ​brightness​ ​of​ ​the​ ​LED​ ​from​ ​off​ ​to
full​ ​brightness.​ ​For​ ​solving​ ​this​ ​assignment​ ​you’ll​ ​find​ ​these​ ​references​ ​to​ ​be
useful:'''

'''Dierct ADC is not here. ADC is done by 3008. 3008 outputs digital
data to a pin on ESP8266.'''

import machine; import time; import ubinascii; import esp;

# assign 1 neopixel
pixel_pin   = machine.Pin(15, machine.Pin.OUT);  # Pin connected to the NeoPixels.
pixel_data = bytearray(3); # array of 3 = 1 neopixel led

spi = machine.SPI(0, baudrate=1000000, polarity=0, phase=0);
cs = machine.Pin(15, machine.Pin.OUT); cs.high();

ubinascii.hexlify(data);

cs.low(); data = spi.read(4); cs.high(); # channel 3 CH3 = spi.read(4)?
data = bytearray(4); cs.low(); spi.readinto(data); cs.high();

def temp_c(data):
    temp = data[0] << 8 | data[1]
    if temp & 0x0001:
        return float('NaN')  # Fault reading data.
    temp >>= 2
    if temp & 0x2000:
        temp -= 16384  # Sign bit set, take 2's compliment.
    return temp * 0.25




