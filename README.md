# Automatic-Pill-dispenser


📂 CODE BREAKDOWN
1. Initialization & Setup (setup() function)
cpp
Copy
Edit
void setup() {
    ...
    servo.attach(servoPin);
    servo.write(0); // Initial angle is 0°
    lcd.begin(16, 2); // Start the 16x2 LCD
    rtc.halt(false); // Start RTC if it was halted
    rtc.writeProtect(false); // Allow time changes
}
The RTC module is initialized to provide real-time.

The servo motor starts at 0°.

The LCD is turned on.

Uncomment the rtc.setTime(...) and rtc.setDate(...) lines once to set time, then comment again.

2. Main Loop (loop() function)
cpp
Copy
Edit
void loop() {
    Time now = rtc.time();  // Get current time
    updateLCD(now);         // Display time on LCD

    // Check if it's time to dispense
    if (millis() - lastDispenseTime >= dispenseInterval) {
        lastDispenseTime = millis();  // Reset timer
        dispensePill();               // Start dispensing process
    }

    delay(500); // Reduce CPU usage
}
The system displays current time.

Every 1 minute (60000 ms), it calls dispensePill() to rotate the servo by 45° (one step), beep, light up LED, and wait for button press before moving further.

3. LCD Time Display (updateLCD() function)
cpp
Copy
Edit
void updateLCD(Time now) {
    lcd.setCursor(0, 0);
    lcd.print("Time: ");
    ...
}
Displays time in HH:MM:SS format on the LCD.

4. Pill Dispensing Logic (dispensePill() function)
If servo is not at 180°, it moves forward by 45°, triggers alert, and waits for user to press the button.

When it reaches 180°, it prepares to return in reverse.

cpp
Copy
Edit
void dispensePill() {
    ...
    if (!moveBackward) {
        currentAngle += stepAngle; // +45°
        if (currentAngle > 180) currentAngle = 180;
        servo.write(currentAngle);
        activateAlert(); // Buzzer + LED
        waitForButtonPress();
        if (currentAngle == 180) moveBackward = true;
    } else {
        returnServoToZero();
    }
}
5. Alerts & Button Waiting
cpp
Copy
Edit
void activateAlert() {
    digitalWrite(buzzerPin, HIGH);
    digitalWrite(ledPin, HIGH);
    lcd.setCursor(0, 1);
    lcd.print("TAKE THE PILL");
}

void waitForButtonPress() {
    while (digitalRead(buttonPin) == HIGH) {
        delay(50); // Wait for press
    }
    delay(200); // Debounce
    digitalWrite(buzzerPin, LOW);
    digitalWrite(ledPin, LOW);
    lcd.clear();
}
Alerts the user to take the pill with LED and buzzer.

Waits until the button is pressed before allowing further movement.

6. Returning Servo to Zero
cpp
Copy
Edit
void returnServoToZero() {
    while (currentAngle > 0) {
        currentAngle -= stepAngle;
        if (currentAngle < 0) currentAngle = 0;
        servo.write(currentAngle);
        activateAlert();
        waitForButtonPress();
    }
    moveBackward = false;
}
The servo goes backwards step-by-step to 0°, prompting user confirmation at each step.

Resets flag to allow forward movement in the next cycle.

🔁 Flow of Operation
Every 60 seconds:

The servo moves by 45°, beeps, flashes LED.

LCD prompts "TAKE THE PILL".

Waits for button press before moving further.

After 4 steps (180°):

Starts reverse movement step-by-step.

Each step again alerts and waits for acknowledgment.

Stops at 0° and resets cycle.

✅ STRENGTHS
Efficient servo control using step angles.

Real-time tracking via DS1302.

Feedback mechanism using buzzer, LED, LCD, and button.

Bi-directional movement handled cleanly.

⚠️ ISSUES TO FIX
Pin Conflicts:

ledPin (8) conflicts with CLK (8) of DS1302.

buttonPin (9) conflicts with DAT (9) of DS1302.

➤ Fix: Use unused digital pins (e.g., 13, A0–A5) for LED and button.

Dispense Timing:

Currently based on millis() (every 60 seconds), not real clock time.

➤ Improvement: Trigger pill dispense at fixed times (e.g., 9 AM, 1 PM, 6 PM) using now.hr and now.min.

