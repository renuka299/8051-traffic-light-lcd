# 8051-traffic-light-lcd
Built a traffic light controller using the 8051 microcontroller with a 16x2 LCD countdown, programmed in Embedded C.
#include <reg51.h>

// ---------- LED pins ----------
sbit red    = P1^1;
sbit green  = P1^2;
sbit yellow = P1^3;

// ---------- LCD pins ----------
#define data_line P2
sbit RS = P3^1;
sbit RW = P3^2;
sbit EN = P3^3;

// ---------- Function prototypes ----------
void lcd_init(void);
void lcd_cmd(unsigned char);
void lcd_data(unsigned char);
void lcd_string(unsigned char *);
void delay(int);

void red_light(void);
void green_light(void);
void yellow_light(void);

// ---------- Red light ON ----------
void red_light(void)
{
    int i;
    red    = 1;
    green  = 0;
    yellow = 0;

    lcd_cmd(0x01);              // clear display
    lcd_cmd(0x80);              // line 1, position 0
    lcd_string("Red");

    for (i = 9; i >= 0; i--)    // countdown 9 to 0
    {
        lcd_cmd(0xC0);          // line 2, position 0
        lcd_data(i + '0');      // convert number to ASCII character
        delay(1000);            // 1 second
    }
}

// ---------- Green light ON ----------
void green_light(void)
{
    int i;
    red    = 0;
    green  = 1;
    yellow = 0;

    lcd_cmd(0x01);              // clear display
    lcd_cmd(0x80);              // line 1, position 0
    lcd_string("Green");

    for (i = 9; i >= 0; i--)    // countdown 9 to 0
    {
        lcd_cmd(0xC0);          // line 2, position 0
        lcd_data(i + '0');
        delay(1000);            // 1 second
    }
}

// ---------- Yellow light ON ----------
void yellow_light(void)
{
    int i;
    red    = 0;
    green  = 0;
    yellow = 1;

    lcd_cmd(0x01);              // clear display
    lcd_cmd(0x80);              // line 1, position 0
    lcd_string("Yellow");

    for (i = 3; i >= 0; i--)    // shorter countdown for yellow
    {
        lcd_cmd(0xC0);          // line 2, position 0
        lcd_data(i + '0');
        delay(1000);            // 1 second
    }
}

// ---------- Main ----------
void main(void)
{
    lcd_init();                         // initialise LCD
    lcd_string("Traffic light");        // show title
    delay(2000);                        // wait 2 seconds

    red    = 0;                         // all LEDs off
    green  = 0;
    yellow = 0;
    lcd_cmd(0x01);                      // clear display

    while (1)                           // repeat forever
    {
        red_light();
        green_light();
        yellow_light();
    }
}

// ---------- LCD functions ----------
void lcd_init(void)
{
    lcd_cmd(0x38);   // 2 lines, 5x7 matrix, 8-bit mode
    lcd_cmd(0x0C);   // display ON, cursor OFF (use 0x0F for blinking cursor)
    lcd_cmd(0x06);   // auto-increment cursor
    lcd_cmd(0x01);   // clear display
    lcd_cmd(0x80);   // cursor to line 1
}

void lcd_cmd(unsigned char c)
{
    data_line = c;   // put command on data pins
    RS = 0;          // command register
    RW = 0;          // write mode
    EN = 1;          // enable pulse high
    delay(5);
    EN = 0;          // enable pulse low (latches command)
}

void lcd_data(unsigned char c)
{
    data_line = c;   // put data on data pins
    RS = 1;          // data register
    RW = 0;          // write mode
    EN = 1;          // enable pulse high
    delay(5);
    EN = 0;          // enable pulse low (latches data)
}

void lcd_string(unsigned char *p)
{
    while (*p != '\0')   // loop until end of string
    {
        lcd_data(*p);    // send one character
        p++;             // move to next character
    }
}

// ---------- Delay using Timer 0 (about 1 ms per count, 11.0592 MHz) ----------
void delay(int t)
{
    int i;
    for (i = 0; i < t; i++)
    {
        TMOD = 0x01;     // Timer 0, mode 1 (16-bit)
        TH0  = 0xFC;     // load high byte
        TL0  = 0x66;     // load low byte
        TR0  = 1;        // start timer
        while (TF0 == 0);// wait for overflow flag
        TR0 = 0;         // stop timer
        TF0 = 0;         // clear overflow flag
    }
}
