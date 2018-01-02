---

comments: true
date: 2013-08-28T00:00:00Z
title: Pure C In Arduino
url: /2013/08/28/pure-c-in-arduino/
---

1\. 输入.c文件，用于点亮arduino板上的LED，默认为pin 5口
```
#include <avr/io.h>
#include <util/delay.h>
 
enum {
 BLINK_DELAY_MS = 1000,
};
 
int main (void)
{
 /* set pin 5 of PORTB for output*/
 DDRB |= _BV(DDB5);
 
 while(1) {
  /* set pin 5 high to turn led on */
  PORTB |= _BV(PORTB5);
  _delay_ms(BLINK_DELAY_MS);
 
  /* set pin 5 low to turn led off */
  PORTB &= ~_BV(PORTB5);
  _delay_ms(BLINK_DELAY_MS);
 }
 
 return 0;
}
```

2\. 编译并生成映像文件：
```
$ avr-gcc -Os -DF_CPU=16000000UL -mmcu=atmega328p -c -o led.o led.c
$ avr-gcc -mmcu=atmega328p led.o -o led
$ avr-objcopy -O ihex -R .eeprom led led.hex
```

3\. 使用avrdude上传之
```
/media/y/arduino/1.0.5/hardware/tools/avrdude -C/media/y/arduino/1.0.5/hardware/tools/avrdude.conf -v -v -v -v -patmega328p -carduino -P/dev/ttyUSB0 -b57600 -D -Uflash:w:led.hex:i
```

上传完毕后板子会自动运行新上传的文件。
感觉用purec来开发的灵活性会比较强。

