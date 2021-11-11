# Code
code for bridge 
/*
 */
#include <avr/io.h>
#include <avr/interrupt.h>
#include <util/delay.h>
#define F_CPU 16000000ul

// LED Defines
#define PORT_LED    PORTC
#define PIN_LED     PINC
#define DDR_LED     DDRC

#define LED_BV_rood         (1 << PC5)  // LED rood boven
#define LED_geel_ODROOD     (1 << PC6)  // LED geel & rood
#define LED_groen           (1 << PC4)  // LED groen

// Button defines
#define PORT_BT     PORTA
#define PIN_BT      PINA
#define DDR_BT      DDRA

#define BUTTON_GREEN (1 << PA2)   // KNOP VOOR OPEN PIN 23
#define BUTTON_RED  (1 << PA0)    // KNOP VOOR DICHT PIN 24
#define DRUK_OPEN   (1 << PA3)    // DRUK sensor open
#define DRUK_DICHT  (1 << PA4)    // DRUK sensor dicht
#define S_BEHIND    (1 << PA5)    // Sensor behind the bridge
#define S_INFRONT   (1 << PA6)    // Sensor in front of the bridge
#define S_WIND      (1 << PA7)    // Flowmeter

int main(void)
{
    init_servo();
    DDR_LED |= LED_BV_rood;
    DDR_LED |= LED_geel_ODROOD;
    DDR_LED |= LED_groen;

    DDR_BT &= ~DRUK_DICHT;
    DDR_BT &= ~DRUK_OPEN;
    DDR_BT &= ~BUTTON_GREEN;
    DDR_BT &= ~BUTTON_RED;

    DDR_BT &= ~S_BEHIND;
    DDR_BT &= ~S_INFRONT;
    DDR_BT &= ~S_WIND;

    // Insert code

int G = 0;
    // resets servo once at the start of the program
    servo1_set_percentage(-100);
    servo2_set_percentage(100);

    while(1)
    {
      /* only for TEST!!!!
        if (!(PINA & (S_BEHIND)))
        {
          PORT_LED &= ~LED_groen;
        }

        else if (PINA & (S_BEHIND))
        {
            PORT_LED |= LED_groen;
        }
*/
        if ((PINA & (S_INFRONT) && (PINA & (DRUK_DICHT))))  // LED_BV_ROOD / LED_GEEL_ODROOD on
        {
            PORT_LED |= LED_BV_rood;
            PORT_LED |= LED_geel_ODROOD;
        }

        if ((!(PINA & (S_INFRONT))) || (PINA & (BUTTON_GREEN))) // adds 1 to G for the servo's
        {
            G++;
            _delay_ms(1000);
            if ((!(PINA & (S_INFRONT))) || (PINA & (BUTTON_GREEN)))// turns led_groen on, led_geel_odrood off
            {
               // flowmeter check if in if
            if(PINA & (S_WIND))   //aanpassing
            {
                _delay_ms(10000);//aanpassing
                G = 0;               //aanpassing
            }
            else if(!(PINA & (S_WIND)))  //aanpassing
            {
                 PORT_LED |= LED_groen;
            PORT_LED &= ~LED_geel_ODROOD;
            _delay_ms(1000);
            }

            if (G == 1)  // if statment is correct servo's go brrrrr
            {
            // servo omlaag
            servo1_set_percentage(100);
            servo2_set_percentage(-100);
            }
            }

            // brug gaat open
        }
        if (PINA & (DRUK_OPEN))  // if the limit switch is pressed turn off LED_BV_rood & after 5 secs LED_groen + reset G to 0
        {
           PORT_LED &= ~LED_BV_rood;
           _delay_ms(5000);
           PORT_LED |= LED_BV_rood;
           _delay_ms(5000);
           PORT_LED &= ~LED_groen;
           G = 0;
        }
            if ((!(PINA & (S_BEHIND))) || (PINA & (BUTTON_RED)))  // checks if the aquatic vehicle has passed the bridge
            {

                if (G == 0)  // checks if G has been reseted if this is true opens servo's
                {
            // brug omlaag
            _delay_ms(2000);
            servo1_set_percentage(-100);
            servo2_set_percentage(100);
            _delay_ms(1000);
                }
            }
    }

    return 0;
}
