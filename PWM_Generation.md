## This should explain how we genenerate the PWM signals to Control ESC's and Servos ##

### What is a PWM Signal ###
PWM [(Pulse Width Modulation)](http://en.wikipedia.org/wiki/Pulse-width_modulation) is a square wave signal with a variable duty time (High, on time)
we use it to simulate a PPM [(Pulse-position modulation)](http://en.wikipedia.org/wiki/Pulse-position_modulation) signal that is used to tell the ESC's their trottle state or servos their position.
to have a valid PPM signal the PWM signals duty time needs to variate between 1 and 2 millisecons (1000us - 2000us).



### Arduino PWM Pin and Software PWM Basics ###

There are two possibilities to generate PWM signals with an [Arduino](http://www.arduino.cc/) [MCU](http://en.wikipedia.org/wiki/Microcontroller)

  * **Hardware PWM (HW PWM)**
> This is the best way to generate PWM signals because its done by a
> macro of the MCU. In this case the MCU does it automatically so the
> main software is not affected. it is also unaffected by software
> interrupts that may cause [jitter](http://en.wikipedia.org/wiki/Jitter).

> There are two ways to activate and controll the HW PWM.
> the easyest is the simplified Arduino function [analogWrite(pin, duty time)](http://arduino.cc/en/Reference/analogWrite). its disadvantage is that it use a large arduino library code and it works only with 8-bit(0-256) resolution.


> The second one (that we use in MWC 2.0) is to access the MCU's PWM register in a direct way. this gives much more possibilities. We can change the resolution and the type of PWM (fast PWM Mode, Phase correct PWM Mode or Phase and frequency correct PWM Mode).
![http://multiwii.googlecode.com/svn/wiki/Images/pwm_mode.jpg](http://multiwii.googlecode.com/svn/wiki/Images/pwm_mode.jpg)

> Hardware PWM works only on some specified pins. that is also the reason why we cant run 8 HW PWMs on the promini.




  * **Software PWM (SW PWM)**
> We use it in MWC for servos and ESC's but only if there are not enough HW PWM pins available. there are many ways to do it. for example a small sketch that does it with the main loop:
```
void setup(){
  pinMode(13,OUTPUT); // set pin 13 witch is no HW PWM pin as output
}

void loop(){

  digitalWrite(13,HIGH); // set it to High

  delayMicroseconds(1500); //wait for 1500us. this gives a PPM signal of 1500us (50% throttle or servo's middle position)

  digitalWrite(13,LOW); // set it to LOW

  delayMicroseconds(500); // let it stay 500us at low state to have a overall loop time of 2000us (500Hz)

}
```
> this works well if the arduino has nothing else to do then generating a PWM signal. but we need the main loop to read the RX, calculate the movent and many other things. so we cant use it like this.
> To be more independent from the main loops code we use [interrupts](http://en.wikipedia.org/wiki/Interrupts) thay do (as the name says) an interrupt at the moment thay need to be processed. so thay stop the main loops code as long as it takes to finish the interrupts code.

> One problem if the interrupt use is if two or more interrupts want to be processed at the same time. than some of them must wait for the others. that causes [jitter](http://en.wikipedia.org/wiki/Jitter) to thair time calculations.

> a small example for a interrupt driven soft PWM code
```
// i use timer 0 for example it runs with default settings at 976Hz
// and it resolution is 8 bit (0-255)

int duty_time = 200; // 200 will give at 488hz a PPM value of 1600us

void setup(){
  pinMode(13,OUTPUT); // set pin 13 to output

  TCCR0A = 0; // clear timer 0's TCCRnA register so it works in normal counting mode

  TIMSK0 |= (1<<OCIE0A); // set timer interrupt mask register to Enable CTC interrupt with timer 0's comperator A
}

// this is the interrupts "loop" for the comperator A of timer 0
// it is called everytime the comperator A reaches its loaded value (it counts from 0 to 255 with 488Hz)

// the register for timer 0's comperator A is OCR0A
ISR(TIMER0_COMPA_vect){
  static uint8_t state = 0; // we need 4 states .. two high and two low to have a 488Hz Frequency

  if (state == 0){

    PORTB |= (1<<5); // set pin 13 high 
     
    OCR0A += duty_time;// load the rigister with the duty time .. the interrupt loop will be called again if this time is over

    state = 1; 
  
  }else if(state == 1){
    OCR0A += duty_time;// load it again because we simulate a fast PWM mode
    state = 2;// set the state to 2 because in the next call we need to set the low time
  }else if(state == 2){

    PORTB &= ~(1<<5);// set the pins state to low

    OCR0A += 255-duty_time;// now we neet to wait for the reminding time to dont have just a high signal

    state = 3;
  }else if(state == 3){
    OCR0A += 255-duty_time;// load it again because we simulate a fast PWM mode
    state = 0; // set the state to 0 so it starts again
  }
}

void loop(){
  // the loop isnt needed for this method
}
```


### Use of this PWM methods in Multiwii ###

> In MultiWii we need to do it for 3 different MCU types (ATmega328P ([Arduino ProMini](http://arduino.cc/it/Main/ArduinoBoardProMini)), Atmega32u4 ([arduino ProMicro](http://www.sparkfun.com/products/10998)) and the [Arduino Mega](http://arduino.cc/en/Main/ArduinoBoardMega) MCU's)

  * **the HW PWM capabilities of the MCU's we use**

> - The Promini has two 8-bit timers  (0 and 2)  each with two compare registers (A & B) and one 16-bit timer (1) with two compare registers (A & B)

> - The Promicro has one 8-bit timer (0) with two compare registers (A & B), two 16-bit timer(1 and 3) with three compare registers (A, B & C) and one 11-bit timer (4) with thee compare registers (A, C & D)

> - The Mega  has two 8-bit timers  (0 and 2)  each with two compare registers (A & B), three 16-bit timers (3, 4 & 5) with three compare registers (A, B & C) and one 16-bit timer (1) with two compare registers (A & B)

> the moust compare registers have a fixed pin that can be used with it.

> this also explains why we can use a 11-bit HW PWM only on the Poromicro and the Mega. (two motors with 11-bit on the Promini would be possible).

  * **Why we use which timer for what**

> Timer 0 can only be used for software PWM because its frequency is too high (976Hz) and we cant change it because Arduino uses Timer0 for its time calculation (Delay, Micros ...)
> Timer 3 has on the Promicro just one pin 5 (timer 3 A) that it also shares with timer 4 A. so we use it for 11-Bit Software PWM

> in the normal case we use timer 0 with SW PMW for the servos and all others for motors.
> in some special cases (promini in a hexa with servos or octo setup) we use one or both timer 0 comperators for doing SW PWM for motors.


  * **Code examples of the initialisation and use in MultiWii**

> use of a 8 bit Timer.  ...timer 2 comperator A for example  (wont work on the promicro because there is no timer 2)
```
  
// MultiWii uses values from 1000 to 2000 for the motor and servo calculation (because of the PPM singnal range of 1000-2000us)
// this means that we would need 11-bit to keep this resolution. some timers have just 8-bit (0-255) so wee need to divide the value by 8 
// 1000/8 = 125,  2000/8 = 250 this matches also the dutytimes to have 1000-2000us
// because our MCU's dont likes to divide (it takes very long to be processed) we do it by a bit shift
  
int dutytime = (1500>>3); // this wloud give us a half throttle signal
// X>>1 = X/2, X>>2 = X/4 and X>>3 = X/8...
  
void setup(){
   pinMode(11, OUTPUT); // set pin 11 to output
     
   TCCR2A |= _BV(COM2A1); // this connects Comperator 2A to its pin
    // note the _BV(REGISTER) is the same as (1<<REGISTER) its just a Arduino wise
    
   OCR2A = dutytime;
}

void loop(){
    // to change the dutytime we just need to write a value to the OCR2A Register
}
```


> use of a 16-bit timer  ...timer 1 comperator A for example
```
// using a 16-bit timer in arduino is a bit more complex because arduinos standad setup stets them to 8 bit
// the higher the resolution is the slower is the timer .. (it counts always in the same speed .. so it takes 3* longer to count till 30 then it takes to count to10)
// because of this and the fact that MultiiWii dont uses 16-bit for the motor calculations we dont need to set them to thair full resolution
// we set it to 0 - 16383 with that value and the  phase and frequency correct mode we dont need a prescaler.
// to use the MWC's values of 1000-2000 we need to multiply the value by 8 .. (1000*8 = 8000, 2000*8 = 16000)  
// we do it also with bit shifting .. but in the other direction
  
int dutytime = (1500<<3); // this wloud give us a half throttle signal
// X<<1 = X*2, X<<2 = X*4 and X<<3 = X*8.... 
  
void setup(){
   pinMode(9, OUTPUT); // set pin 9 to output
     
   TCCR1A |= (1<<WGM11); TCCR1A &= ~(1<<WGM10); TCCR1B |= (1<<WGM13);  // phase correct mode.. to know how to set which register please see the datasheets of the MCU's
   
   TCCR1B &= ~(1<<CS11); // no prescaler
   ICR1 |= 0x3FFF; // set timers to count to 16383 (hex 3FFF)
   TCCR1A |= _BV(COM1A1); // connect pin 9 to timer 1 channel A
    
   OCR1A = dutytime;
}

void loop(){
    // to change the dutytime we just need to write a value to the OCR1A Register
}
```

> use of a 11-bit timer  ...timer 4 comperator A for example (for now just the ATmega32u4 -> promicro has a 11-bit timer)
```
// this 11-bit timer is some kind of different to the others because it is made to work with very high frequencys 
// but we use it with 488Hz just as the other ones.
// because it is 11-bit we dont need to divide or multiply .. we take the MWC's values just as thay are
  
int dutytime = 1500; // this wloud give us a half throttle signal

  
void setup(){
   pinMode(5, OUTPUT); // set pin 5 to output
     
   TCCR4E |= (1<<ENHC4); // this enables the enhanced mode of the atmega32u4's timer 4. it gives one more bit of resolution (without is has just 10-bit)
   TCCR4B &= ~(1<<CS41); TCCR4B |= (1<<CS42)|(1<<CS40); // sets the prescaler to 16 .. this is needed because it it a high frequency timer
   TCCR4D |= (1<<WGM40); //set it to phase and frequency correct mode
   
   // now there is one special thing with timer 4... it counts with 11-bit resolution but the compare registers are just 8-bit
   // so we cant set a 11 bit value to them with just register = 2000..
   // it has a special 3 bit register for the high byte. means we need to split the 11-bit value to one 3-bit and one 8-bit
   // to set its top value to 2048 (hex 3FF) we set its high byte bit to hex 3
   TC4H = 0x3;
   // and the the 8 bit register to hex FF
   OCR4C = 0xFF;
   // the Timer combines this values to hex 3FF (2048)
   
   // we need to do the same thing to write 11-bit values to the comperators
   // but we can do the split automaticly ;)
   
   static uint8_t pwm4_HBA;
   static uint16_t pwm4_LBA; // high and low byte for timer 4 A
   
   pwm4_LBA = 2047-dutytime; // timer 4 A is inverted
   pwm4_HBA = 0; 
   
   while(pwm4_LBA > 255){ // splits the MWC value of 1000-2000 
     pwm4_HBA++;
     pwm4_LBA-=256;
   }
   
   TC4H = pwm4_HBA; 
   OCR4A = pwm4_LBA; //  pin 5
   
   
}

void loop(){
  // to change the dutytime we just need to write the high and the low byte to TC4H(high byte) and OCR4A (low byte)
}
```


  * **Servo software PWM in MultiWii**
> ... to be continued







# Details #

Add your content here.  Format your content with:
  * Text in **bold** or _italic_
  * Headings, paragraphs, and lists
  * Automatic links to other wiki pages