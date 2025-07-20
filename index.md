# Button Piano
I created a button piano that bears the most similarities to an actual piano. This, of course, meant incorporating actual pedaling with your feet, as well as modifications like reverb with a speaker. Building the actual piano itself was not the real challenge; rather, the code and trying to mimic the functionality of an actual piano/keyboard, which you would play as an instrument, were the real challenges.

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Roger Z | Wilcox High School | Electrical Engineering | Incoming Senior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="1617" height="640" src="https://www.youtube.com/embed/IZ7pv20wpT0" title="Roger Z Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

In the first milestone, my instructor and I ordered mini speakers, a SD card, a DFPlayer, and wires to further enhance my project. By implementing speakers instead of piezo buzzers, I figured out that I could produce more robust sounds while not being limited to only one frequency. This meant I was also able to achieve another goal of adding sound effects alongside playing single notes. There is a limit of only being able to play 3 notes at once due to the Arduino Uno kit's limitation, so adding sound effects was a way around that. I was not able to add reverb yet since one of the biggest challenges I had this week was actually getting the DFPlayer installed, and also using an external computer to add mp3 files, since my main computer couldn't detect SD cards. A pedal system is currently being organized, and my code reflects that by this milestone. However, the hardware itself will be shown next week due to new parts coming in (wires not being long enough).

# First Milestone

<iframe width="1617" height="640" src="https://www.youtube.com/embed/IZ7pv20wpT0" title="Roger Z Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

The button piano base kit consists of mechanical buttons that, when pressed, produce a distinct sound through the Arduino Uno Board that can be modified inside the Arduino IDE. Right now, not all of the code can be shown physically working, due to the temporary use of a passive buzzer instead of a piezo buzzer. A piezo buzzer uses something called the piezoelectric effect, which uses crystals to generate an electric charge that will cause sound to play. A piezo buzzer has been ordered so you will hopefully see a piezo buzzer working in other milestone videos. I also asked my instructor to order mini speakers that will allow me to add reverb, and maybe even turn my button piano into some kind of soundboard in the future. In my second milestone, I hope to code more functionality to my button piano like reverb, be able to assemble a well-designed breadboard, and debug as much of the code as possible. 

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code

```c++
#define KEY_C 262    // (Middle C)
#define KEY_D 294     
#define KEY_E 330    
#define KEY_F 350    
#define KEY_G 392    
#define KEY_A 440    
#define KEY_B 494    
#define KEY_C1 523   // (One octave above middle C)
#define KEY_D1 587    

const int NUM_KEYS = 9;

const int INPUT_BUTTON_PINS[NUM_KEYS] = {2, 3, 4, 5, 6, 7, 8, 9, 10};
const int NOTE_FREQUENCIES[NUM_KEYS] = {
  KEY_C, KEY_D, KEY_E, KEY_F, KEY_G, KEY_A, KEY_B, KEY_C1, KEY_D1
};
const char* NOTE_NAMES[NUM_KEYS] = {
  "C", "D", "E", "F", "G", "A", "B", "C1", "D1"
};

const int OUTPUT_PIEZO_PIN = 11;
const int OUTPUT_LED_PIN = LED_BUILTIN;

const boolean _buttonsAreActiveLow = true;
const int DEBOUNCE_WINDOW = 10; // milliseconds

int _prevRawButtonVals[NUM_KEYS];
int _debouncedButtonVals[NUM_KEYS];
unsigned long _buttonStateChangeTimestamps[NUM_KEYS];

int currentNoteIndex = -1;
int lastPrintedNoteIndex = -1;
unsigned long lastNotePrintTime = 0;
int heldNoteCount = 0;

void setup() {
  Serial.begin(9600);
  Serial.println("Starting piano debug...");

  for (int i = 0; i < NUM_KEYS; i++) {
    pinMode(INPUT_BUTTON_PINS[i], INPUT_PULLUP);
    _prevRawButtonVals[i] = digitalRead(INPUT_BUTTON_PINS[i]);
    _debouncedButtonVals[i] = _prevRawButtonVals[i];
    _buttonStateChangeTimestamps[i] = millis();
  }

  pinMode(OUTPUT_PIEZO_PIN, OUTPUT);
  pinMode(OUTPUT_LED_PIN, OUTPUT);
}

void loop() {
  bool anyKeyPressed = false;
  int activeNoteIndex = -1;

  for (int i = 0; i < NUM_KEYS; i++) {
    int rawVal = digitalRead(INPUT_BUTTON_PINS[i]);

    if (rawVal != _prevRawButtonVals[i]) {
      _buttonStateChangeTimestamps[i] = millis();
    }

    if ((millis() - _buttonStateChangeTimestamps[i]) >= DEBOUNCE_WINDOW) {
      _debouncedButtonVals[i] = rawVal;
    }

    _prevRawButtonVals[i] = rawVal;

    bool pressed = (_buttonsAreActiveLow && _debouncedButtonVals[i] == LOW) ||
                   (!_buttonsAreActiveLow && _debouncedButtonVals[i] == HIGH);

    if (pressed) {
      activeNoteIndex = i;
      anyKeyPressed = true;
      break;  // Only play one note at a time
    }
  }

  if (anyKeyPressed && activeNoteIndex != -1) {
    tone(OUTPUT_PIEZO_PIN, NOTE_FREQUENCIES[activeNoteIndex]);
    digitalWrite(OUTPUT_LED_PIN, HIGH);

    if (activeNoteIndex == currentNoteIndex) {
      heldNoteCount++;
    } else {
      heldNoteCount = 1;
      currentNoteIndex = activeNoteIndex;
    }
    // Debugging purposes and testing debouncing buttons
    if (currentNoteIndex != lastPrintedNoteIndex || millis() - lastNotePrintTime > 300) {
      Serial.print("Playing note: ");
      Serial.print(NOTE_NAMES[currentNoteIndex]);
      Serial.print(" [");
      Serial.print(heldNoteCount);
      Serial.println("]");
      lastPrintedNoteIndex = currentNoteIndex;
      lastNotePrintTime = millis();
    }

  } else {
    noTone(OUTPUT_PIEZO_PIN);
    digitalWrite(OUTPUT_LED_PIN, LOW);
    currentNoteIndex = -1;
    heldNoteCount = 0;
  }
}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
