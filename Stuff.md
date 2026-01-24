# Core ideas

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

---

# Repeating ideas in his codes

These are some repeating things in his codes. Instead of trying to memorize each code, learning these patterns may be more beneficial.

---

## TimerOne.h

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

## Speeding up and slowing down logic
```cpp

```

## 
```cpp

```

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

```cpp

```
