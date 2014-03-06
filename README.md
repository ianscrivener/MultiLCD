MultiLCD Library for Arduino
============================

Written by Stanley Huang, distributed under GPL.
MultiLCD is a highly optimized library designed for displaying characters and bitmaps on multiple models of Arduino display shields/modules with easy-to-use and unified API. It is developed with performance in first priority. Though supported models are not as many as some widely used Arduino display libraries (e.g. UTFT), it renders faster with simpler API.

For more information, please visit http://arduinodev.com

This library supports following Arduino shields/modules:

* Freematics 2.2" TFT LCD shield (16-bit color)
* IteadStudio 2.8" TFT LCD shield (16-bit color)
* ILI9341 based TFT LCD shields (SPI)
* ILI9325D based TFT LCD shields
* SSD1306 based OLED module (monochrome)
* SH1106 based OLED module (monochrome)
* Nokia 3310/5100 module (monochrome)
* DFRobot LCD4884 shield (monochrome)

To obtain any of above hardware, please visit: http://arduinodev.com/store/?route=product/category&path=17

![Arduino LCD shields](http://arduinodev.com/wp-content/uploads/2013/03/arduino_lcd_shields-300x195.jpg)

The library embeds font data for ASCII characters (5x7 and 8x16) and digits (8x8, 16x16, 16x24). It is extremely simple for display texts and numbers on desired position on a LCD screen with the library, while very little change in code is needed to switch from one LCD module to another.

To use a specific shield or module as the display for Arduino, you need to include library header at the beginning of the sketch.

    #include <MultiLCD.h>

And use one of following declarations before your code.

For SSD1306 OLED module:

    LCD_SSD1306 lcd;

For LCD4884 shield or Nokia 5100 module:

    LCD_PCD8544 lcd;

For ILI9325D shield:

    LCD_ILI9325D lcd;

For ILI9341 based TFT LCD module:

    LCD_ILI9341 lcd;

The library class inherits the Print class of Arduino, so that you can display texts on LCD with standard Arduino functions like this:

    lcd.print("Hello, World!";)
    lcd.print(foo, DEC);
    lcd.print(bar, HEX);
    lcd.print(1.23)         // gives "1.23" 
    lcd.print(1.23456, 2);  // gives "1.23" 

Besides, it provides unified APIs for initializing and controlling the LCD, as well as some convenient operations.

    void begin(); /* initializing */
    void clear(); /* clear screen */
    void setCursor(uint16_t column, uint8_t line); /* set current cursor, column is in pixel */
    void setFont(FONT_SIZE size); /* set font size */
    void printInt(unsigned int n); /* display a integer number */
    void printLong(unsigned long n); /* display a long number */
    void draw(const PROGMEM uint8_t* buffer, uint16_t x, uint16_t y, uint16_t width, uint16_t height); /* draw monochrome bitmap */

The code using the library can be extremely simple.

    #include <Wire.h>
    #include <MultiLCD.h>

    LCD_SSD1306 lcd; /* for SSD1306 OLED module */

    static const PROGMEM uint8_t smile[48 * 48 / 8] = {
    0x00,0x00,0x00,0x00,0x00,0x00,0x80,0xC0,0xE0,0xF0,0xF8,0xF8,0xFC,0xFC,0xFE,0xFE,0x7E,0x7F,0x7F,0x3F,0x3F,0x3F,0x3F,0x3F,0x3F,0x3F,0x3F,0x3F,0x3F,0x7F,0x7F,0x7E,0xFE,0xFE,0xFC,0xFC,0xF8,0xF8,0xF0,0xE0,0xC0,0x80,0x00,0x00,0x00,0x00,0x00,0x00,
    0x00,0xC0,0xF0,0xFC,0xFE,0xFF,0xFF,0xFF,0x3F,0x1F,0x0F,0x07,0x03,0x01,0x00,0x80,0x80,0x80,0x80,0x80,0x80,0x00,0x00,0x00,0x00,0x00,0x00,0x80,0x80,0x80,0x80,0x80,0x80,0x00,0x01,0x03,0x07,0x0F,0x1F,0x3F,0xFF,0xFF,0xFF,0xFE,0xFC,0xF0,0xC0,0x00,
    0xFE,0xFF,0xFF,0xFF,0xFF,0xFF,0x07,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x06,0x1F,0x1F,0x1F,0x3F,0x1F,0x1F,0x02,0x00,0x00,0x00,0x00,0x06,0x1F,0x1F,0x1F,0x3F,0x1F,0x1F,0x02,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x07,0xFF,0xFF,0xFF,0xFF,0xFF,0xFE,
    0x7F,0xFF,0xFF,0xFF,0xFF,0xFF,0xE0,0x00,0x00,0x30,0xF8,0xF8,0xF8,0xF8,0xE0,0xC0,0x80,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x80,0xC0,0xE0,0xF8,0xF8,0xFC,0xF8,0x30,0x00,0x00,0xE0,0xFF,0xFF,0xFF,0xFF,0xFF,0x7F,
    0x00,0x03,0x0F,0x3F,0x7F,0xFF,0xFF,0xFF,0xFC,0xF8,0xF0,0xE1,0xC7,0x87,0x0F,0x1F,0x3F,0x3F,0x3E,0x7E,0x7C,0x7C,0x7C,0x78,0x78,0x7C,0x7C,0x7C,0x7E,0x3E,0x3F,0x3F,0x1F,0x0F,0x87,0xC7,0xE1,0xF0,0xF8,0xFC,0xFF,0xFF,0xFF,0x7F,0x3F,0x0F,0x03,0x00,
    0x00,0x00,0x00,0x00,0x00,0x00,0x01,0x03,0x07,0x0F,0x1F,0x1F,0x3F,0x3F,0x7F,0x7F,0x7E,0xFE,0xFE,0xFC,0xFC,0xFC,0xFC,0xFC,0xFC,0xFC,0xFC,0xFC,0xFC,0xFE,0xFE,0x7E,0x7F,0x7F,0x3F,0x3F,0x1F,0x1F,0x0F,0x07,0x03,0x01,0x00,0x00,0x00,0x00,0x00,0x00,
    };

    void setup()
    {
        lcd.begin();
    }

    void loop()
    {
        lcd.clear();
        lcd.draw(smile, 40, 8, 48, 48);
        delay(2000);

        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.setFont(FONT_SIZE_SMALL);
        lcd.print("Hello, world!");

        lcd.setCursor(0, 1);
        lcd.setFont(FONT_SIZE_MEDIUM);
        lcd.print("Hello, world!");

        lcd.setCursor(0, 3);
        lcd.setFont(FONT_SIZE_SMALL);
        lcd.printLong(12345678);

        lcd.setCursor(64, 3);
        lcd.setFont(FONT_SIZE_MEDIUM);
        lcd.printLong(12345678);

        lcd.setCursor(0, 4);
        lcd.setFont(FONT_SIZE_LARGE);
        lcd.printLong(12345678);

        lcd.setCursor(0, 6);
        lcd.setFont(FONT_SIZE_XLARGE);
        lcd.printLong(12345678);

        delay(3000);
    }


Drawing bitmap on SSD1306:

![Drawing bitmap on SSD1306 OLED](http://www.arduinodev.com/wp-content/uploads/2013/05/oled_smile-300x247.jpg)

