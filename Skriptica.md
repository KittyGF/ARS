
# Core ideas

Some must-knows for this kolokvij


<br>

## Code structure
```cpp
void setup() {
}

void loop() {
}
```

The code in `setup()` runs only once, while `loop()` runs over and over again.

---

## Serial and digital pins
<img width="741" height="381" alt="image" src="https://github.com/user-attachments/assets/b26a59d7-943c-4b77-a61c-81fb67a93876" />

### Example of a blinking LED
```cpp
void setup() {
  pinMode(12, OUTPUT);
}

void loop() {
  digitalWrite(12, HIGH);
  delay(1000);
  digitalWrite(12, LOW);
  delay(1000);
}
```

Something I like to do is define the pin number for better organization (useful when there are many pins).  
However, professor does not do this in his codes and we've been told he likes it his way. Here is an example anyway.

```cpp
#define LED 12

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  digitalWrite(LED, HIGH);
  delay(1000);
  digitalWrite(LED, LOW);
  delay(1000);
}
```

---

## delay();
It pauses the execution of the code for however many milliseconds you write.

```cpp
delay(500);
```

<br>
<br>
<br>

# Repeating ideas in his codes

These are some repeating things in his codes. Instead of trying to memorize each code, learning these patterns may be more beneficial.


<br>


## TimerOne.h

We use Timer1 whenever the task tells us that something needs to happen every specific amount of time.  
For example, if we want the LED to become brighter every 2 seconds.

Include the library:
```cpp
#include <TimerOne.h>
```

Set timer period in microseconds:
```cpp
Timer1.initialize(6000000); // 6 seconds
```

Attach interrupt function:
```cpp
Timer1.attachInterrupt(yourFunctionHere);
```

Example:
```cpp
#include <TimerOne.h>

void setup() {
  Timer1.initialize(1000000); // 1 second
  Timer1.attachInterrupt(function);
}

void loop() {
}

void function() {
  // This function runs every second
}
```

---

## External Interrupt

Used when something happens on a pin (like pressing a button).  
So whenever a task says that something happens when a button is pressed - this is what we use.

```cpp
attachInterrupt(digitalPinToInterrupt(numberOfThePin), theFunctionWeWantCalled, mode);
```

Example:
```cpp
void setup() {
  pinMode(2, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2), tipka, RISING);
}
```


There are 4 modes:
- RISING
- FALLING
- CHANGE
- LOW

Professor always uses `RISING`.  
Variables used in interrupt functions should be declared `volatile`.  

---

## Speeding up and slowing down logic
```cpp
void tipka2() {
  brzina = (brzina*1.1 > 255) ? 255 : brzina*1.1;
}
void tipka3() {
  brzina = (brzina*0.9 < 1) ? 1 : brzina*0.9;
}
```
same logic is used for brightness.

---

## Setting up and writing to multiple pins with a for loop
Instead of writing
```cpp
pinMode(2, OUTPUT);
pinMode(3, OUTPUT);
pinMode(4, OUTPUT);
pinMode(5, OUTPUT);
digitalWrite(2, HIGH);
digitalWrite(3, HIGH);
digitalWrite(4, HIGH);
digitalWrite(5, HIGH);

```
we can do
```cpp
bool state[4];
for(int i=0; i<4; i++) { 
  pinMode(i+2, OUTPUT);  
  state[i] = HIGH;       
  digitalWrite(i+2, state[i]); 
}
```
Professor does it this way in his codes!

---

## Setting analogRead() values to keys
First we read from analog keypad.
```cpp
int x = analogRead(A0);
```
Then we map the values to each key.
```cpp
key = x>900 ? 1 : (x>700 ? 2 : (x>500 ? 3 : (x>300 ? 4 : key)));
```
That's how it's written in examples but it's the same as doing this. I find this way more readable.
```cpp
if (x > 900) key = 1;
else if (x > 700) key = 2;
else if (x > 500) key = 3;
else if (x > 300) key = 4;
```
This is useful when we want something to happen as we press different keys. We can use switch case for this.  
Example:

```cpp
// 2. Analogna tipkala s vrijednostima {0, 400, 600, 800, 1000} ± 10
// postavljaju stanja 4 LED diode sa zajedničkom anodom
// + Ana   + anoda

bool state[4]; 
void setup() {
  Serial.begin(9600); 
  for(int i=0; i<4; i++) { 
    pinMode(i+2, OUTPUT);  
    state[i] = HIGH;
    digitalWrite(i+2, state[i]);
  }
}
void loop() {
  int key = 0;
  int x = analogRead(A0); 
  key = x>900 ? 1 : (x>700 ? 2 : (x>500 ? 3 : (x>300 ? 4 : key))); 
  switch(key) {
    case 1: digitalWrite(2, state[0] = !state[0]); break;
    case 2: digitalWrite(3, state[1] = !state[1]); break;
    case 3: digitalWrite(4, state[2] = !state[2]); break;
    case 4: digitalWrite(5, state[3] = !state[3]); break;
  }
  Serial.println(key);
  delay(100);
}
```
---

## map
Map is used to rescale from one range to another.  
It is used to map analog input ranges to PWM output ranges.
```cpp
map(value, oldLow, oldHigh, newLow, newHigh);
```
| Analog values | PWM values |
| -------------  | ------------- | 
| 0 - 1023 | 0 - 255 |


Example (task 3):
```cpp

/* Analogna tipkala s vrijednostima   {0, 400, 600,  800, 1000} ± 10
postavlja 5 razina svjetla na jednoj LED (0, 25,   50,   75,  100) %
                                     pwm   0  64   127   190   255*/

void setup() {
  Serial.begin(9600);
}
void loop() {
  int key = 0;
  int x = analogRead(A0);
  key = x>900 ? 1 : (x>700 ? 2 : (x>500 ? 3 : (x>300 ? 4 : key))); // 0,1,2,3,4
  
  int pwm_value = map(key,0,4,0,255); // 0..4 --> 0..255
  analogWrite( 9, pwm_value);
  
  Serial.println(key);
  delay(100);
}
```
---

## Changing colors on an RGB LED
```cpp
void postaviBoju(){
  analogWrite( 9, state ? brightness : 0); // r
  analogWrite(10, state ? brightness : 0); // g
  analogWrite(11, state ? brightness : 0); // b
}
```

It sets the color of an RGB LED using PWM, depending on the variable `state` and the variable `brightness`.  
`state` and `brightness` are volatile variables we set in the rest of the code.  
The example underneath this one shows a similar way of doing this.

---


## Joystick 
Mapping joystick values and setting LED color (task 7).

```cpp
// Jedna os joysticka PWM upravlja s RGB LED. U lijevo svijetli plavo, u desno bijelo proporcionalno poziciji joysticka

void loop() {
  int x = analogRead(A0); // joystick: 0=lijevo...512=sredina...1023=desno
  int b = map(x,0,512,    255,0); // smanjuje se u lijevo 512...0    => 0...255
  int w = map(x,512,1023, 0,255); // povećava se u desno  512...1023 => 0...255
  if(x<512) {
    analogWrite( 9, 0); // r
    analogWrite(10, 0); // g
    analogWrite(11, b); // b
  } else {
    analogWrite( 9, w); // r
    analogWrite(10, w); // g
    analogWrite(11, w); // b
  }
}
```
---

## Stop watch

```cpp
// 8. Izmjerite proteklo vrijeme između dva pritiska jedne tipke. Prikaz vremena treba biti u sekundama na ekranu računala

#include "TimerOne.h"

volatile int vrijeme, s;

void setup() {
  Serial.begin(9600);
  Timer1.initialize(1e6); // 1000000 μs = 1 s
  Timer1.attachInterrupt(sekunda);
  pinMode(2, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2),tipka,RISING);
  vrijeme = s = 0;
}
void sekunda() {
  s++; // brojač sekundi
}
void tipka() {
  vrijeme = s; // stisnuta tipka -> dohvati vrijeme
  s = 0;       // reset brojača
}
void loop() {
  if(vrijeme) { // ako vrijeme nije 0
    Serial.println(vrijeme);
    vrijeme = 0;
  }
}

```

---

## Getting RPM of a motor

```cpp
// 9. Koristeći prekid, treba izmjeriti brzinu istosmjernog motora RPM (okretaja u minuti)
//    i ispisati je na ekranu računala
#include "TimerOne.h"
volatile int cnt, rpm;
volatile bool bIspis;
void setup() {
  Serial.begin(9600);
  Timer1.initialize(1e6); // 1000000 μs = 1s
  Timer1.attachInterrupt(sekunda);
  pinMode(2, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2),senzor,RISING);
}
void senzor() {
  cnt++;
}
void sekunda() {
  rpm = cnt/60;
  cnt = 0;
  bIspis = true;
}
void loop() {
  if(bIspis) {
    Serial.println("RPM = " + String(rpm)+" o/min");
    bIspis = false;
  }
}

```
---

## Dice logic
Without using rand() we can make a 6D die this way.
```cpp
  k = k < 6 ? k + 1 : 1;
```
We would put this inside of a loop so the number keeps on changing.

Example:
```cpp
// 10. Realizirajte jednu igraću kocku pomoću tipke za prekid ispisivanjem broja od 1 do 6 na ekranu računala, bez rand()

int k; // globalna varijabla kocka
volatile bool bIspis;

void setup() {
  Serial.begin(9600);
  pinMode(2, INPUT_PULLUP); // tipka
  attachInterrupt(digitalPinToInterrupt(2), tipka, RISING);
  k = 1;
}

void tipka() {
  bIspis = bIspis ? false : true; // postavi na true ako nije postavljen
}

void loop() { // 16 MHz ~ μs
  k = k < 6 ? k + 1 : 1;
  if(bIspis) {
    Serial.println(k);
    delay(100); // debounce tipke
    bIspis = false;
  }
}

```
This is the logic for two dice.
```cpp
void loop() {  // 16 MHz ~ μs
  for (int k1 = 1; k1 <= 6; k1++)
    for (int k2 = 1; k2 <= 6; k2++)
      if (digitalRead(2) == LOW) {
        Serial.println(String(k1) + ", " + String(k2));
        delay(100); // debounce
      }
}

```
---

## UART input

```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  if (Serial.available()) {
    char c = Serial.read();
  }
}
```
Primjeri zadataka u kojima se koristi:
<br>
<br>
20. Znakovi sa UART-a se trebaju brojati pomoću tri brojača znakova, ovisno o vrsti znaka: broj, malo slovo, veliko slovo
```cpp

void setup() {
  Serial.begin(9600);
}
void loop () {
  while(Serial.available()) {
    char c = Serial.read();
    if(c == '\n') {
      int x, y, z; // tri brojača
      if(isDigit(c) || (c>='0' && c<= '9') || (c>=48 && c<=57))
        x++;
      else if(isUpperCase(c) || (c>='A' && c<= 'Z') || (c>=65 && c<=90))
        y++;
      else if(isLowerCase(c) || (c>='a' && c<= 'z') || (c>=97 && c<=122))
        z++;
      Serial.println(String(x)+", "+String(y)+", "+String(z));
    }
  }
}

```
21. UART postavlja razinu svjetla RGB LED. Prima se redni broj LED od 1 do 3, te razina svjetline od 0 do 100 %
```cpp


String str;
void setup() {
  Serial.begin(9600);
}
void loop () {
  while(Serial.available()) {
    char c = Serial.read();
    if(c == '\n') {
      int iLed, duty;
      sscanf(str.c_str(), "%d %d", &iLed, &duty);
      iLed = constrain(iLed, 1, 3);
      duty = constrain(duty, 0, 100);
      analogWrite(9+(iLed-1), map(duty,0,100,0,255)); // pwm_pin=9,10,11  pwm_value=0..255
      str = "";
    } else {
      str += c;
    }
  }
}


```
22. Kada se sa UART-a primi niz znakova FERIT i znak za kraj niza, na zaslonu računala treba ispisati tekst OK

```cpp

String str;
void setup() {
  Serial.begin(9600);
}
void loop() {
  while (Serial.available()) {
    char c = Serial.read();
    if (c == '\n') {
      if (str == "FERIT" || str.equals("FERIT")) {
        Serial.println("OK");
      }
      str = "";
    } else {
      str += c;
    }
  }
}

```
---
<br>
<br>

# Tasks that repeat with slight differences 

## Mjerenje vremena
12. Vrijeme treba mjeriti pomoću biblioteke `TimerOne.h`. Ispišite vrijeme na zaslonu računala u formatu hh-mm-ss
```cpp
#include "TimerOne.h"
volatile int h, m, s;
volatile bool bIspis;
void setup() {
  Serial.begin(9600);
  Timer1.initialize(1e6);  // sekunde u μs
  Timer1.attachInterrupt(sekunda);
}
void sekunda() {
  if (++s > 59) {
    s = 0;
    bIspis = true; // Serial ne ide u interrupt
    if (++m > 59) {
      m = 0;
      if (++h > 23) {
        h = 0;
}}}}
void loop() {
  if(bIspis) {
    Serial.println(String(h)+":"+String(m)+":"+String(s));
    bIspis = false;
  }
}
```
13. Vrijeme treba mjeriti pomoću funkcije `millis()`. Ispišite vrijeme na zaslonu računala u formatu hh-mm-ss
```cpp
int h, m, s;
long ms;
void setup() {
  Serial.begin(9600);
  ms = 0;
}
void sekunda() {
  if (++s > 59) {
    s = 0;
    if (++m > 59) {
      m = 0;
      if (++h > 23) {
        h = 0;
}}}}
void loop() {
  if(millis() - ms > 1000) {
    ms = millis();
    sekunda();
    Serial.println(String(h)+":"+String(m)+":"+String(s));
  }
}

```
14. Vrijeme treba mjeriti pomoću funkcije `delay()`. Ispišite vrijeme na zaslonu računala u formatu hh-mm-ss
```cpp
int h, m, s;
void setup() {
  Serial.begin(9600);
}
void sekunda() {
  if (++s > 59) {
    s = 0;
    if (++m > 59) {
      m = 0;
      if (++h > 23) {
        h = 0;
}}}}
void loop() {
  delay(1000); // blokirajuća funkcija
  sekunda();
  Serial.println(String(h)+":"+String(m)+":"+String(s));
}

```
---

## Running light 
15. Koristite `delay()` za trćeće svjetlo s 8 LED. Period kretanja od 5 s i promjena smjera svakih 5 minuta
```cpp
int index;   // 2B
bool smjer;  // 1B
int m, s;
void setup() {
  for (int i = 0; i < 8; i++) {  // index i=0..7
    pinMode(i + 2, OUTPUT);      // pin i+2=2..9
    digitalWrite(i + 2, LOW);    // LEDs OFF
  }
}
void loop() {
  delay(1000);  // 1 sekunda
  if (++s >= 59) {
    s = 0;
    if (++m >= 59) {
      m = 0;
    }
  }
  if (s % 5 == 0) {
    digitalWrite(index + 2, LOW); // LED OFF // ugasi koja je svjetlila do sada
    index = smjer ? (index < 7 ? index + 1 : 0) : // 0..7
                    (index > 0 ? index - 1 : 0) ; // 7..0
    digitalWrite(index + 2, HIGH);// LED ON // upali slijedeću koja svijetli
  }
  if (m % 5 == 0) {
    smjer = !smjer;
  }
}

```
16. Koristite `millis()` za trćeće svjetlo s 8 LED. Period kretanja od 5 s i promjena smjera svakih 5 minuta
```cpp
int index;   // 2B
bool smjer;  // 1B
long ms;
int m, s;
void setup() {
  for (int i = 0; i < 8; i++) {  // index i=0..7
    pinMode(i + 2, OUTPUT);      // pin i+2=2..9
    digitalWrite(i + 2, LOW);    // LEDs OFF
  }
  ms = 0;
}
void sekunda() {
  if(millis() - ms > 1000) {
    ms = millis();
    if (++s >= 59) {
      s = 0;
      if (++m >= 59) {
        m = 0;
      }
    }
  }
  if (s % 5 == 0) {
    digitalWrite(index + 2, LOW); // LED OFF // ugasi koja je svjetlila do sada
    index = smjer ? (index < 7 ? index + 1 : 0) : // 0..7
                    (index > 0 ? index - 1 : 0) ; // 7..0
    digitalWrite(index + 2, HIGH);// LED ON // upali slijedeću koja svijetli
  }
  if (m % 5 == 0) {
    smjer = !smjer;
  }
}

```


# Turning the boiler on 

17. Koristite `delay()` za uključivanje grijača vode od 21:00-7:00,
pretpostavite da je mikroupravljač uključen u 21:00 sat
```cpp
#define HOUR_10 10 * 60 * 60 * 1000  // 10 sati u ms
#define HOUR_14 14 * 60 * 60 * 1000  // 14 sati u ms
void setup() {
  pinMode(13, OUTPUT);
}
void loop() {
  digitalWrite(13, HIGH);
  delay(HOUR_10); // ms
  digitalWrite(13, LOW);
  delay(HOUR_14); // ms
}

```
18. Koristite `millis()` za uključivanje grijača vode od 21:00-7:00,  
pretpostavite da je mikroupravljač uključen u 21:00 sat

```cpp
#define HOUR 60 * 60 * 1000  // sati u ms
int h;
long ms;
void setup() {
  pinMode(13, OUTPUT);
  digitalWrite(13, HIGH);
  h = 21;
  ms = 0;
}
void loop() {
  if (millis() - ms >= HOUR) {
    ms = millis();
    h = h < 23 ? h + 1 : 0;  // h = h % 23;
    digitalWrite((h >= 21 || h < 7) ? HIGH : LOW);
  }
}

```
19. Koristite `Timer1` za uključivanje grijača vode od 21:00-7:00,
pretpostavite da je mikroupravljač uključen u 21:00 sat

```cpp
#include "TimerOne.h"
#define HOUR 60*60*1000*1000 // sati u μs
volatile int h;
void setup() {
  pinMode(13, OUTPUT);
  digitalWrite(13, HIGH);
  Timer1.initialize(HOUR);
  Timer1.attachInterrupt(sat);
  h = 21;
}
void sat() {
  h = h < 23 ? h+1 : 0; // h = h % 23;
  digitalWrite(13,(h>=21 || h<7) ? HIGH : LOW);
}
void loop() { }

```
---

Das it.
