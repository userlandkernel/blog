# Hacking a toy lasersword
What do you do when someone breaks her toy (my friend did), you save the hardware from the bin and hack it for a new begin.  

## Introduction
The toy lasersword seemed to consist of blue and red led lights that were wired to eachother in parallel.  
Following these leds we end up at a small circuit board that has some resistors on it, which are needed for the leds to work and not break.  
On the circuit we also see three small push buttons, these are simply switches in the logic board.  
In physics there's a simple law of electricity that a circuit needs to be closed in order for it to transport power.  
So one of the buttons must be for turning on and off the leds.  
We also see that there is sound chip on the logic board, but after searching the web for the serial number it is a non-reflashable chip.  
The speaker that is connected can ofcourse still be hacked :)  
When powering the circuit it became clear the other two buttons are for the audio and for switching between blue and red lights.  
The circuit is mostlikely designed so that another resistor can prevent the red leds from being powered if a specific voltage is going through the circuit.  
Which is what makes switching between the leds possible.  
So what can we do with this when we have little hardware?  
We at least need an Arduino.  
I used an Arduino Uno (32kb bootrom + flash).  

## The revival of Alan Turing
Well not exactly yet as Turing cryptographically broke the enigma code and we are going to make a morse machine.  
For the morse machine to work we need some basic knowledge about leds.  
Leds are diodes, they need a minimum voltage to transport power and light up.  
The buttons are no obstacle, we can just leave them as is.  
We need to seperate the wires of the speaker from the logic board since we will not need them and don't want the default sounds anyway.  
Since Arduino has digital output, it is easy to control the led pins with a low an high voltage digital write.  
For controlling the leds we can just take the power wires that go into the circuit.  
These wires go into pin 10 and ground so that we can control the leds.  
The code for controlling leds is very simple.  
We only need to define pin 10 as digital output.  
We can then use digitalWrite(10, LOW); and digitalWrite(10, HIGH); for turning the led on and off.  
So for the morse alphabet we need to do some research, a dit is one time unit and a dah is three time units.  
Because we also want to see when a character is finished, we need to have some other interval according to the standards which is one dit.
We also want to have some time between words, whitespaces. These are defined as seven dits according to the morse standards.  
So how will we do this? We use the delay() function and use 100 miliseconds as one time unit.
First we will define the dits and dahs in functions as following:

```c
morse_dit(){ // .
  digitalWrite(10, HIGH); // LED ON
  delay(100);
  digitalWrite(10, LOW); // LED OFF
}

morse_dah(){ // -
   digitalWrite(10, HIGH); // LED ON
   delay(300);
   digitalWrite(10, LOW); // LED OFF
} 
```

We also need an interval function for characters, which is simple:
```c
  morse_interval_letter(){
    delay(100); // One time unit of pure nothingnesss
  }
```

Now we can simply write functions for each letter morse_A(), morse_B() and so on. 
All we do in these functions is accordingly turn on and off the leds with the correct dits and dahs.  

```c
  morse_S(){
    morse_dit(); // .
    morse_dit(); // .
    morse_dit(); // .
    morse_letter_interval(); // Each letter should call the interval.
 }
```

Since we are lazy, it is fun to write a morse decode function too so that we can write messages.  
All we need for that is a for loop that loops through each character in a string, then in a switch case plays the correct morse for it.  
Piece of cake right?  

```c
    morse_decode(char *str, int strLen){
      for(int i = 0; i < strLen; i++){
        switch((int)str[i]){  // Characters are just integers
          case 'A':
              morse_A();
              break;
          ....
```

(More coming soon).
