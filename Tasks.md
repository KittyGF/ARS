## Zadatak 1
### LED na pinu 13 treba blinkati, a Timer1 ubrzava blinkanje za 20 % svakih 6 sekundi, interrupt tipka usporava za 30 %
```cpp
#include "TimerOne.h"
volatile int ms;   // volatile --> 2 stvari: 1. varijabla u RAM, ne u registar (nije privremena varijabla)
//                                           2. compiler neće ignorirati funkc. koja se ne poziva
void setup() {
  pinMode(13, OUTPUT); // LED
  Timer1.initialize(6e6); // [μs] 6 milijuna mikrosekundi. ~ takt 16 MHz
  Timer1.attachInterrupt(speedUpBlinking);
  pinMode(2, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2),tipka,RISING); // hardware int
  ms = 1000;
}
void speedUpBlinking() {
  //ms *= 0.8; // 20% kraći delay ili 80% brži blink
  ms = ms>100 ? ms*0.8 : 100;
}
void tipka() {
  ms *= 1.3; // 30% veći delay ili 70% sporiji blink
  ms = ms<15e3 ? ms*1.3 : 15e3;
}
void loop() {
  digitalWrite(13, HIGH);
  delay(ms);
  digitalWrite(13, LOW);
  delay(ms); // ovo zaborave napisati
}
```
---
<br>
<br>

## Zadatak 2
### Analogna tipkala s vrijednostima {0, 400, 600, 800, 1000} ± 10 postavljaju stanja 4 LED diode sa zajedničkom anodom
```cpp
bool state; // 1B
bool state[4]; // 1B x 4
 int state[4]; // 2B x 4
void setup() {
  Serial.begin(9600); // 115200 baudrate
  for(int i=0; i<4; i++) { // index i=0..3
    pinMode(i+2, OUTPUT);  // pin i+2=2..5
    state[i] = HIGH; // ugašena
    digitalWrite(i+2, state[i]); // LED OFF
  }
}
void loop() {
  int key = 0;
  int x = analogRead(A0); // A0-A7
  key = x>900 ? 1 : (x>700 ? 2 : (x>500 ? 3 : (x>300 ? 4 : key))); // 0,1,2,3,4
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
<br>
<br>

## Zadatak 3
### RGB LED treba mijenjati plavu i bijelu boju s periodom od 1 s. Pritisak na tipku povećava razinu svjetline za 10 %

```cpp

volatile int brightness;
volatile bool bInterrupt, state;

void setup() {
  pinMode(2, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2),tipka,RISING);
  brightness = 50;
  state = 0;
}
void postaviBoju(){
  analogWrite( 9, state ? brightness : 0); // r
  analogWrite(10, state ? brightness : 0); // g
  analogWrite(11, brightness); // b
}
void tipka() {
  if(!bInterrupt) {  // spriječava poziv INT tijekom INT
    bInterrupt = true;
  }
}
void loop() {
  if(bInterrupt) {
    brightness = brightness*1.1<255 ? brightness*1.1 : 255;
    delay(100); // debounce
    bInterrupt = false;
  }
  postaviBoju();
  delay(1000);
  state = !state;
}

```

---
<br>
<br>

## Zadatak 4
### Analogna tipkala s vrijednostima      {0, 400, 600,  800, 1000} ± 10 postavlja 5 razina svjetla na jednoj LED (0, 25,   50,   75,  100) %
pwm   0  64   127   190   255  
napon   0V 1.2V  2.5V  3.7V  5V

```cpp
#define pwm_25 0.25*255

void setup() {
  Serial.begin(9600); // 115200 baudrate
}
void loop() {
  int key = 0;
  int x = analogRead(A0); // A0-A7
  key = x>900 ? 1 : (x>700 ? 2 : (x>500 ? 3 : (x>300 ? 4 : key))); // 0,1,2,3,4
  
  int pwm_value = map(key,0,4,0,255); // 0..4 --> 0..255
  analogWrite( 9, pwm_value); // pinovi 5 i 6 su za Timer0 koji koristimo za delay()
  
  Serial.println(key);
  delay(100);
}
```
---

<br>
<br>

## Zadatak 5
### LDR mjeri razinu osvjetljenja i ovisno o osvjetljenju svijetli od 0 do 8 LED sa zajedničkom katodom
```cpp

void setup() {
  Serial.begin(9600);
  for(int i=0; i<8; i++) { // index i=0..7
    pinMode(i+2, OUTPUT); // LED
    digitalWrite(i+2, LOW); // LEDs OFF
  }
}
void loop() {
  int x = analogread(A0); // A0-A7 --> 0..1023
  int n = map(x,330,1000,0,8); // 0..1023 --> 0..8.  broj upaljenih LED
  // 0..8 LED OFF
  for(int i=0; i<8; i++) { // index i=0..7
    digitalWrite(i+2, LOW); // LEDs OFF.           sve ugasi
  }
  // 0..n LED ON
  for(int i=0; i<n; i++) { // index i=0..7
    digitalWrite(i+2, HIGH); // LED ON             upali prvih n
  }
  delay(200); //                                   čekaj 200 ms
}


```
---

<br>
<br>

## Zadatak 6
### Pritiskom na jednu od 2 tipke povećajte ili smanjite brzinu istosmjernog motora za 10 % svakih 222 ms
```cpp
int brzina;
void setup() {
  Serial.begin(9600);
  pinMode(2, INPUT_PULLUP);
  pinMode(3, INPUT_PULLUP);
  brzina = 70;
}
void loop() {
  if(digitalRead(2)==0) // stisnuta tipka na pinu 2
    brzina = (brzina*1.1 > 255) ? 255 : brzina*1.1;
  if(!digitalRead(3)) // stisnuta tipka na pinu 3
    brzina = (brzina*0.9 < 1) ? 1 : brzina*0.9;
  analogWrite(9, brzina); // 0..255
  delay(222); // Timer0: ili delay ili analogWrite(5 ili 6, pwm_value)
}

// Rješenje koristeći interrupt


volatile int brzina;
void setup() {
  Serial.begin(9600);
  pinMode(2, INPUT_PULLUP);
  pinMode(3, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2),tipka2,RISING);
  attachInterrupt(digitalPinToInterrupt(3),tipka3,RISING);
  brzina = 70;
}
void tipka2() {
  brzina = (brzina*1.1 > 255) ? 255 : brzina*1.1;
}
void tipka3() {
  brzina = (brzina*0.9 < 1) ? 1 : brzina*0.9;
}
void loop() {
  analogWrite(9, brzina); // 0..255
  delay(222); // Timer0: ili delay ili analogWrite(5 ili 6, pwm_value)
}


```
---

<br>
<br>

## Zadatak 7
### Jedna os joysticka PWM upravlja s RGB LED. U lijevo svijetli plavo, u desno bijelo proporcionalno poziciji joysticka
```cpp
void setup() { }

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
<br>
<br>

## Zadatak 8
### Izmjerite proteklo vrijeme između dva pritiska jedne tipke. Prikaz vremena treba biti u sekundama na ekranu računala
```cpp
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
<br>
<br>

## Zadatak 9
### Koristeći prekid, treba izmjeriti brzinu istosmjernog motora RPM (okretaja u minuti) i ispisati je na ekranu računala
```cpp
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
<br>
<br>

## Zadatak 10
### Realizirajte jednu igraću kocku pomoću tipke za prekid ispisivanjem broja od 1 do 6 na ekranu računala, bez `rand()`
```cpp
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
---
<br>
<br>

## Zadatak 11
### Realizirajte 2 igraće kocke pomoću tipke za prekid ispisivanjem 2 broja od 1 do 6 na ekranu računala, bez `rand()`
```cpp
void setup() {
  pinMode(2, INPUT_PULLUP);
  Serial.begin(9600);
}
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
<br>
<br>

## Zadatak 12
### Vrijeme treba mjeriti pomoću biblioteke `TimerOne.h`. Ispišite vrijeme na zaslonu računala u formatu hh-mm-ss
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
---
<br>
<br>

## Zadatak 13
### Vrijeme treba mjeriti pomoću funkcije `millis()`. Ispišite vrijeme na zaslonu računala u formatu hh-mm-ss
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
---
<br>
<br>

## Zadatak 14
### Vrijeme treba mjeriti pomoću funkcije `delay()`. Ispišite vrijeme na zaslonu računala u formatu hh-mm-ss
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
<br>
<br>

## Zadatak 15
### Koristite `delay()` za trćeće svjetlo s 8 LED. Period kretanja od 5 s i promjena smjera svakih 5 minuta
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
---
<br>
<br>

## Zadatak 16
### Koristite `millis()` za trćeće svjetlo s 8 LED. Period kretanja od 5 s i promjena smjera svakih 5 minuta
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
---
<br>
<br>

## Zadatak 17
### Koristite `delay()` za uključivanje grijača vode od 21:00-7:00, pretpostavite da je mikroupravljač uključen u 21:00 sat
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
```cpp
#define HOUR 60 * 60 * 1000  // sati u ms
int h;
long ms;
void setup() {
  pinMode(13, OUTPUT);
  digitalWrite(13, HIGH);
  h = 21;
}
void loop() {
  delay(HOUR); // ms
  h = h < 23 ? h + 1 : 0;  // h = h % 23;
  digitalWrite((h >= 21 || h < 7) ? HIGH : LOW);
}

```
---
<br>
<br>

## Zadatak 18
### Koristite `millis()` za uključivanje grijača vode od 21:00-7:00, pretpostavite da je mikroupravljač uključen u 21:00 sat
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
---
<br>
<br>

## Zadatak 19
### Koristite Timer1 za uključivanje grijača vode od 21:00-7:00, pretpostavite da je mikroupravljač uključen u 21:00 sat
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

<br>
<br>

## Zadatak 20
### Znakovi sa UART-a se trebaju brojati pomoću tri brojača znakova, ovisno o vrsti znaka: broj, malo slovo, veliko slovo
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
      // Serial.print(x);
      // Serial.print(", ");
      // Serial.print(y);
      // Serial.print(", ");
      // Serial.print(z);
      // Serial.print("\n");
    }
  }
}
```
---
<br>
<br>

## Zadatak 21
### UART postavlja razinu svjetla RGB LED. Prima se redni broj LED od 1 do 3 te razina svjetline od 0 do 100 %

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
---
<br>
<br>

## Zadatak 22
### Kada se sa UART-a primi niz znakova FERIT i znak za kraj niza, na zaslonu računala treba ispisati tekst OK
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

