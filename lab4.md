# CPE 301L - Laboratory 4: LCD & Keypad

## Complete Solutions with Code, Wiring, and Expected Results

---

## Hardware Wiring Guide

### UNLV ECE Development Board Users (Recommended)

If you are using the **UNLV ECE Development Board** with built-in LCD header, the LCD connections are pre-routed on the PCB. Connect your SparkFun ADM1602K LCD directly to the **LCD Header** (Location 1 on the board).

**UNLV Board LCD Connections:**

| Board Feature | Location | Description |
|---------------|----------|-------------|
| LCD Header | Location 1 | 16-pin header for LCD module |
| CONTRAST Potentiometer | Location 2 | Built-in contrast adjustment (no external pot needed) |
| DATA Header | Location 3 | LCD data pins connection |
| Control Pins (RS, RW, EN) | Location 21 | LCD control signal header |

**To use the LCD on UNLV Board:**
1. Insert the ADM1602K LCD into the LCD header (Location 1) - ensure pin 1 alignment
2. Adjust contrast using the built-in potentiometer (Location 2)
3. The LCD data and control pins are pre-wired to specific microcontroller PORTs

**Verify your board's PORT mappings by checking:**
- The board silkscreen labels near the DATA header (Location 3)
- The UNLV 328P Board Manual for exact pin-to-PORT mapping
- Typical configuration: Data on PORTB, Control on PORTC

**Based on the original lab document, the UNLV board uses:**
- **Data PORT**: PORTB (PB0-PB7 for LCD D0-D7)
- **RS (Register Select)**: PC0
- **RW (Read/Write)**: PC1
- **E (Enable)**: PC2

> **Important**: Before running the code, verify these connections match your specific board revision by checking the labels on the DATA header (Location 3) and Control header (Location 21).

---

### Manual Wiring (SparkFun ADM1602K - 16x2 Character LCD)

The **ADM1602K** is a standard 16x2 character LCD module with LED backlight, compatible with the HD44780 controller. The E349456 is a batch/lot number.

**LCD Specifications:**
- Display: 16 characters x 2 lines
- Controller: HD44780 compatible
- Interface: 8-bit or 4-bit parallel
- Backlight: LED (yellow-green)
- Operating Voltage: 5V DC

| LCD Pin | Name | Connection to ATmega328P | Description |
|---------|------|--------------------------|-------------|
| 1 | VSS | GND | Ground (0V) |
| 2 | VDD | +5V | Power Supply (+5V) |
| 3 | V0 | Potentiometer Wiper | Contrast Control (10k pot: ends to VCC/GND, wiper to V0) |
| 4 | RS | PC0 | Register Select (0=Command, 1=Data) |
| 5 | R/W | PC1 | Read/Write (0=Write, 1=Read) |
| 6 | E | PC2 | Enable (data latched on falling edge) |
| 7 | D0 | PB0 | Data Bit 0 (LSB) |
| 8 | D1 | PB1 | Data Bit 1 |
| 9 | D2 | PB2 | Data Bit 2 |
| 10 | D3 | PB3 | Data Bit 3 |
| 11 | D4 | PB4 | Data Bit 4 |
| 12 | D5 | PB5 | Data Bit 5 |
| 13 | D6 | PB6 | Data Bit 6 |
| 14 | D7 | PB7 | Data Bit 7 (MSB) |
| 15 | A (LED+) | +5V (via 220Ω resistor) | Backlight Anode (+) |
| 16 | K (LED-) | GND | Backlight Cathode (-) |

**Important Notes for ADM1602K:**
- Pin 1 is on the LEFT when viewing the LCD from the front (text facing you) with pins at the top
- The contrast potentiometer is critical - without proper adjustment, you may see nothing or black boxes
- For 8-bit mode, all data pins (D0-D7) must be connected
- The backlight requires a current-limiting resistor (220Ω typical)

### Keypad Wiring (4x4 Matrix)

#### UNLV Board Users
The UNLV development board has a **built-in keypad header** at Location 5. The keypad pins (C1-C4, R1-R4) are pre-routed to the microcontroller.

**UNLV Board Keypad Connections:**
- Connect your 4x4 keypad to the keypad header at Location 5
- Check the board silkscreen or manual for the exact PORT assignments
- Typical configuration: Keypad on PORTD

#### Manual Wiring (for custom boards)

| Keypad | Connection to ATmega328P | Type |
|--------|--------------------------|------|
| Row 1 (R1) | PD0 | Output |
| Row 2 (R2) | PD1 | Output |
| Row 3 (R3) | PD2 | Output |
| Row 4 (R4) | PD3 | Output |
| Col 1 (C1) | PD4 | Input (with internal pull-up) |
| Col 2 (C2) | PD5 | Input (with internal pull-up) |
| Col 3 (C3) | PD6 | Input (with internal pull-up) |
| Col 4 (C4) | PD7 | Input (with internal pull-up) |

### Keypad Layout Reference

```
        C1    C2    C3    C4
    ┌─────┬─────┬─────┬─────┐
R1  │  1  │  2  │  3  │  A  │
    ├─────┼─────┼─────┼─────┤
R2  │  4  │  5  │  6  │  B  │
    ├─────┼─────┼─────┼─────┤
R3  │  7  │  8  │  9  │  C  │
    ├─────┼─────┼─────┼─────┤
R4  │  *  │  0  │  #  │  D  │
    └─────┴─────┴─────┴─────┘
```

---

## Experiment 1: Displaying on LCD Screen

### Objective
Display your name on the first row and the lab info on the second row of the LCD.

### Complete Code

```c
#define F_CPU 1000000UL
#include <avr/io.h>
#include <util/delay.h>

// LCD Control Pin Definitions
#define rs PC0   // Register Select: 0=Command, 1=Data
#define rw PC1   // Read/Write: 0=Write, 1=Read
#define en PC2   // Enable pin

// Data port for LCD
#define data PORTB
#define dataDir DDRB

// Function Prototypes
void lcd_init(void);
void lcd_cmd(char cmd_out);
void lcd_data(char data_out);
void lcd_str(char *str);
void lcd_clear(void);
void lcd_set_cursor(char row, char col);

int main(void)
{
    // Set Port B as output for LCD data (D0-D7)
    dataDir = 0xFF;
    // Set Port C pins 0,1,2 as output for LCD control
    DDRC |= (1 << rs) | (1 << rw) | (1 << en);

    // Initialize the LCD
    lcd_init();

    // Display on first row
    lcd_set_cursor(0, 0);  // Set cursor to row 0, column 0
    lcd_str("Doe, John");  // Replace with your name

    // Display on second row
    lcd_set_cursor(1, 0);  // Set cursor to row 1, column 0
    lcd_str("CPE 301L: Lab 4");

    while(1)
    {
        // Main loop - display remains static
    }

    return 0;
}

// Initialize LCD
void lcd_init(void)
{
    _delay_ms(15);          // Wait for LCD to power up
    lcd_cmd(0x38);          // Initialize to 2 lines & 5x7 font (8-bit mode)
    _delay_ms(1);
    lcd_cmd(0x0C);          // Display ON, Cursor OFF
    _delay_ms(1);
    lcd_cmd(0x06);          // Auto-increment cursor to the right
    _delay_ms(1);
    lcd_cmd(0x01);          // Clear display
    _delay_ms(2);
    lcd_cmd(0x80);          // Set cursor to beginning of first line
    _delay_ms(1);
}

// Send command to LCD
void lcd_cmd(char cmd_out)
{
    data = cmd_out;         // Send command to data port

    PORTC &= ~(1 << rs);    // RS = 0 (Command mode)
    PORTC &= ~(1 << rw);    // RW = 0 (Write mode)
    PORTC |= (1 << en);     // EN = 1 (Enable high)

    _delay_ms(10);          // Wait for command to process

    PORTC &= ~(1 << en);    // EN = 0 (Enable low - falling edge triggers LCD)

    _delay_ms(10);
}

// Send data (character) to LCD
void lcd_data(char data_out)
{
    data = data_out;        // Send data to data port

    PORTC |= (1 << rs);     // RS = 1 (Data mode)
    PORTC &= ~(1 << rw);    // RW = 0 (Write mode)
    PORTC |= (1 << en);     // EN = 1 (Enable high)

    _delay_ms(10);          // Wait for data to process

    PORTC &= ~(1 << en);    // EN = 0 (Enable low - falling edge triggers LCD)

    _delay_ms(10);
}

// Send string to LCD
void lcd_str(char *str)
{
    unsigned int i = 0;
    while(str[i] != '\0')
    {
        lcd_data(str[i]);
        i++;
    }
}

// Clear LCD display
void lcd_clear(void)
{
    lcd_cmd(0x01);          // Clear display command
    _delay_ms(2);
}

// Set cursor position
// row: 0 for first line, 1 for second line
// col: 0-15 for column position
void lcd_set_cursor(char row, char col)
{
    char address;
    if(row == 0)
        address = 0x80 + col;   // First line starts at 0x80
    else
        address = 0xC0 + col;   // Second line starts at 0xC0
    lcd_cmd(address);
}
```

### Expected Result
- **First Row**: "Doe, John" (your actual name)
- **Second Row**: "CPE 301L: Lab 4"
- The display should remain static with the text visible

---

## Experiment 2: Displaying Keypad Values on LCD

### Objective
Display the integer value of the buttons on the keypad when pressed. For example, when '4' is pressed, display "4 was pressed."

### Complete Code

```c
#define F_CPU 1000000UL
#include <avr/io.h>
#include <util/delay.h>

// LCD Control Pin Definitions
#define rs PC0
#define rw PC1
#define en PC2
#define data PORTB
#define dataDir DDRB

// Keypad Pin Definitions (Port D)
// Rows (Outputs): PD0-PD3
// Columns (Inputs): PD4-PD7
#define KEYBOARD_PORT PORTD
#define KEYBOARD_DDR  DDRD
#define KEYBOARD_PIN  PIND

// Function Prototypes
void lcd_init(void);
void lcd_cmd(char cmd_out);
void lcd_data(char data_out);
void lcd_str(char *str);
void lcd_clear(void);
void lcd_set_cursor(char row, char col);
char keypad_scan(void);
void keypad_init(void);

int main(void)
{
    char key;

    // Initialize LCD
    dataDir = 0xFF;
    DDRC |= (1 << rs) | (1 << rw) | (1 << en);
    lcd_init();

    // Initialize keypad
    keypad_init();

    // Display welcome message
    lcd_str("Press a Key...");

    while(1)
    {
        key = keypad_scan();    // Scan keypad for key press

        if(key != 0)            // If a key was pressed
        {
            lcd_clear();        // Clear display
            lcd_data(key);      // Display the key character
            lcd_str(" was pressed");
            _delay_ms(500);     // Debounce delay
        }
    }

    return 0;
}

// Initialize keypad
void keypad_init(void)
{
    // Set rows (PD0-PD3) as outputs, columns (PD4-PD7) as inputs
    KEYBOARD_DDR = 0x0F;
    // Enable internal pull-ups on columns and set rows high
    KEYBOARD_PORT = 0xFF;
}

// Scan keypad and return pressed key
// Returns 0 if no key pressed
char keypad_scan(void)
{
    // Keypad mapping for 4x4 matrix
    // Row 1: 1, 2, 3, A
    // Row 2: 4, 5, 6, B
    // Row 3: 7, 8, 9, C
    // Row 4: *, 0, #, D

    // Check Row 1 (PD0 = LOW)
    KEYBOARD_PORT = 0b11111110;  // Row 1 low, others high
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '1';  // Col 1
    if((KEYBOARD_PIN & 0b00100000) == 0) return '2';  // Col 2
    if((KEYBOARD_PIN & 0b01000000) == 0) return '3';  // Col 3
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'A';  // Col 4

    // Check Row 2 (PD1 = LOW)
    KEYBOARD_PORT = 0b11111101;  // Row 2 low, others high
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '4';  // Col 1
    if((KEYBOARD_PIN & 0b00100000) == 0) return '5';  // Col 2
    if((KEYBOARD_PIN & 0b01000000) == 0) return '6';  // Col 3
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'B';  // Col 4

    // Check Row 3 (PD2 = LOW)
    KEYBOARD_PORT = 0b11111011;  // Row 3 low, others high
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '7';  // Col 1
    if((KEYBOARD_PIN & 0b00100000) == 0) return '8';  // Col 2
    if((KEYBOARD_PIN & 0b01000000) == 0) return '9';  // Col 3
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'C';  // Col 4

    // Check Row 4 (PD3 = LOW)
    KEYBOARD_PORT = 0b11110111;  // Row 4 low, others high
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '*';  // Col 1
    if((KEYBOARD_PIN & 0b00100000) == 0) return '0';  // Col 2
    if((KEYBOARD_PIN & 0b01000000) == 0) return '#';  // Col 3
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'D';  // Col 4

    return 0;  // No key pressed
}

// LCD Functions (same as Experiment 1)
void lcd_init(void)
{
    _delay_ms(15);
    lcd_cmd(0x38);
    _delay_ms(1);
    lcd_cmd(0x0C);
    _delay_ms(1);
    lcd_cmd(0x06);
    _delay_ms(1);
    lcd_cmd(0x01);
    _delay_ms(2);
    lcd_cmd(0x80);
    _delay_ms(1);
}

void lcd_cmd(char cmd_out)
{
    data = cmd_out;
    PORTC &= ~(1 << rs);
    PORTC &= ~(1 << rw);
    PORTC |= (1 << en);
    _delay_ms(10);
    PORTC &= ~(1 << en);
    _delay_ms(10);
}

void lcd_data(char data_out)
{
    data = data_out;
    PORTC |= (1 << rs);
    PORTC &= ~(1 << rw);
    PORTC |= (1 << en);
    _delay_ms(10);
    PORTC &= ~(1 << en);
    _delay_ms(10);
}

void lcd_str(char *str)
{
    unsigned int i = 0;
    while(str[i] != '\0')
    {
        lcd_data(str[i]);
        i++;
    }
}

void lcd_clear(void)
{
    lcd_cmd(0x01);
    _delay_ms(2);
}

void lcd_set_cursor(char row, char col)
{
    char address;
    if(row == 0)
        address = 0x80 + col;
    else
        address = 0xC0 + col;
    lcd_cmd(address);
}
```

### Expected Result
- Initially displays: "Press a Key..."
- When you press '4': Display shows "4 was pressed"
- When you press '7': Display shows "7 was pressed"
- When you press 'A': Display shows "A was pressed"
- Works for all 16 keys on the keypad

---

## Experiment 3: Combination Lock System

### Objective
Create a combination lock that accepts a 4-digit code. Display "Lock Opened" for correct code, "Incorrect Sequence" for wrong code.

### Complete Code

```c
#define F_CPU 1000000UL
#include <avr/io.h>
#include <util/delay.h>

// LCD Control Pin Definitions
#define rs PC0
#define rw PC1
#define en PC2
#define data PORTB
#define dataDir DDRB

// Keypad Pin Definitions
#define KEYBOARD_PORT PORTD
#define KEYBOARD_DDR  DDRD
#define KEYBOARD_PIN  PIND

// Define your 4-digit combination lock code here
#define CODE_1 '1'
#define CODE_2 '2'
#define CODE_3 '3'
#define CODE_4 '4'
// 'D' button acts as Enter key

// Function Prototypes
void lcd_init(void);
void lcd_cmd(char cmd_out);
void lcd_data(char data_out);
void lcd_str(char *str);
void lcd_clear(void);
void lcd_set_cursor(char row, char col);
char keypad_scan(void);
char keypad_get_key(void);
void keypad_init(void);

int main(void)
{
    char key;
    char entered_code[4];
    int code_index = 0;
    int correct;

    // Initialize LCD
    dataDir = 0xFF;
    DDRC |= (1 << rs) | (1 << rw) | (1 << en);
    lcd_init();

    // Initialize keypad
    keypad_init();

    while(1)
    {
        // Reset for new entry
        code_index = 0;
        lcd_clear();
        lcd_str("Enter Code:");
        lcd_set_cursor(1, 0);
        lcd_str("____");  // Placeholder for 4 digits
        lcd_set_cursor(1, 0);

        // Get 4 digits from user
        while(code_index < 4)
        {
            key = keypad_get_key();  // Wait for a key press

            // Skip 'D' (Enter key) until 4 digits entered
            if(key == 'D')
            {
                continue;
            }

            // Store entered digit and display asterisk
            entered_code[code_index] = key;
            lcd_data('*');  // Show asterisk for privacy
            code_index++;
        }

        // Wait for Enter key (D)
        lcd_set_cursor(1, 5);
        lcd_str(" Press D");

        while(1)
        {
            key = keypad_get_key();
            if(key == 'D')
            {
                break;
            }
        }

        // Verify the code
        correct = 1;
        if(entered_code[0] != CODE_1) correct = 0;
        if(entered_code[1] != CODE_2) correct = 0;
        if(entered_code[2] != CODE_3) correct = 0;
        if(entered_code[3] != CODE_4) correct = 0;

        // Display result
        lcd_clear();
        if(correct)
        {
            lcd_str("Lock Opened!");
            // Optional: Turn on a green LED on PB4
            PORTB |= (1 << PB4);
        }
        else
        {
            lcd_str("Incorrect");
            lcd_set_cursor(1, 0);
            lcd_str("Sequence");
        }

        // Wait 3 seconds before allowing retry
        _delay_ms(3000);

        // Turn off LED if was on
        PORTB &= ~(1 << PB4);
    }

    return 0;
}

// Initialize keypad
void keypad_init(void)
{
    KEYBOARD_DDR = 0x0F;   // Rows output, columns input
    KEYBOARD_PORT = 0xFF;  // Pull-ups on columns, rows high
}

// Scan keypad and return pressed key (non-blocking)
char keypad_scan(void)
{
    // Check Row 1
    KEYBOARD_PORT = 0b11111110;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '1';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '2';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '3';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'A';

    // Check Row 2
    KEYBOARD_PORT = 0b11111101;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '4';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '5';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '6';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'B';

    // Check Row 3
    KEYBOARD_PORT = 0b11111011;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '7';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '8';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '9';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'C';

    // Check Row 4
    KEYBOARD_PORT = 0b11110111;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '*';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '0';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '#';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'D';

    return 0;
}

// Get key with wait (blocking) and debounce
char keypad_get_key(void)
{
    char key;

    // Wait until a key is pressed
    do {
        key = keypad_scan();
    } while(key == 0);

    // Wait until key is released (debounce)
    _delay_ms(50);
    while(keypad_scan() != 0);
    _delay_ms(50);

    return key;
}

// LCD Functions
void lcd_init(void)
{
    _delay_ms(15);
    lcd_cmd(0x38);
    _delay_ms(1);
    lcd_cmd(0x0C);
    _delay_ms(1);
    lcd_cmd(0x06);
    _delay_ms(1);
    lcd_cmd(0x01);
    _delay_ms(2);
    lcd_cmd(0x80);
    _delay_ms(1);
}

void lcd_cmd(char cmd_out)
{
    data = cmd_out;
    PORTC &= ~(1 << rs);
    PORTC &= ~(1 << rw);
    PORTC |= (1 << en);
    _delay_ms(10);
    PORTC &= ~(1 << en);
    _delay_ms(10);
}

void lcd_data(char data_out)
{
    data = data_out;
    PORTC |= (1 << rs);
    PORTC &= ~(1 << rw);
    PORTC |= (1 << en);
    _delay_ms(10);
    PORTC &= ~(1 << en);
    _delay_ms(10);
}

void lcd_str(char *str)
{
    unsigned int i = 0;
    while(str[i] != '\0')
    {
        lcd_data(str[i]);
        i++;
    }
}

void lcd_clear(void)
{
    lcd_cmd(0x01);
    _delay_ms(2);
}

void lcd_set_cursor(char row, char col)
{
    char address;
    if(row == 0)
        address = 0x80 + col;
    else
        address = 0xC0 + col;
    lcd_cmd(address);
}
```

### Expected Result

**Scenario 1: Correct Code (1-2-3-4 + D)**
```
Enter Code:
**** Press D
```
(After pressing D)
```
Lock Opened!
```

**Scenario 2: Incorrect Code (9-8-7-6 + D)**
```
Enter Code:
**** Press D
```
(After pressing D)
```
Incorrect
Sequence
```

---

## Experiment 4: Extra Credit - Cursor Control with Pushbuttons

### Objective
Use 4 pushbuttons to control cursor position (left, right, up, down). Keypad presses display characters at current cursor position.

### Wiring Additions for Pushbuttons

| Pushbutton | Connection | Function |
|------------|------------|----------|
| Button 1 | PA0 (with pull-up) | Cursor Left |
| Button 2 | PA1 (with pull-up) | Cursor Right |
| Button 3 | PA2 (with pull-up) | Cursor Up |
| Button 4 | PA3 (with pull-up) | Cursor Down |

Each button connects between the pin and GND (active low with internal pull-up).

### Complete Code

```c
#define F_CPU 1000000UL
#include <avr/io.h>
#include <util/delay.h>

// LCD Control Pin Definitions
#define rs PC0
#define rw PC1
#define en PC2
#define data PORTB
#define dataDir DDRB

// Keypad Pin Definitions
#define KEYBOARD_PORT PORTD
#define KEYBOARD_DDR  DDRD
#define KEYBOARD_PIN  PIND

// Pushbutton Pin Definitions (Port A)
#define BTN_PORT PORTA
#define BTN_DDR  DDRA
#define BTN_PIN  PINA
#define BTN_LEFT  PA0
#define BTN_RIGHT PA1
#define BTN_UP    PA2
#define BTN_DOWN  PA3

// Cursor position tracking
volatile char cursor_row = 0;  // 0 = first row, 1 = second row
volatile char cursor_col = 0;  // 0-15

// Function Prototypes
void lcd_init(void);
void lcd_cmd(char cmd_out);
void lcd_data(char data_out);
void lcd_str(char *str);
void lcd_clear(void);
void lcd_set_cursor(char row, char col);
void lcd_update_cursor(void);
char keypad_scan(void);
char keypad_get_key(void);
void keypad_init(void);
void buttons_init(void);
void handle_cursor_movement(void);

int main(void)
{
    char key;

    // Initialize LCD
    dataDir = 0xFF;
    DDRC |= (1 << rs) | (1 << rw) | (1 << en);
    lcd_init();

    // Initialize keypad
    keypad_init();

    // Initialize pushbuttons
    buttons_init();

    // Clear display and set cursor
    lcd_clear();
    lcd_set_cursor(0, 0);

    // Display instructions briefly
    lcd_str("Use buttons");
    lcd_set_cursor(1, 0);
    lcd_str("for cursor");
    _delay_ms(2000);
    lcd_clear();
    lcd_set_cursor(0, 0);

    while(1)
    {
        // Check for cursor movement buttons
        handle_cursor_movement();

        // Check for keypad input
        key = keypad_scan();
        if(key != 0)
        {
            // Display the pressed key at current cursor position
            lcd_data(key);

            // Update cursor position tracking
            cursor_col++;
            if(cursor_col > 15)
            {
                cursor_col = 0;
                cursor_row = !cursor_row;
                lcd_set_cursor(cursor_row, cursor_col);
            }

            // Debounce delay
            _delay_ms(200);
            while(keypad_scan() != 0);
            _delay_ms(50);
        }
    }

    return 0;
}

// Initialize pushbuttons with internal pull-ups
void buttons_init(void)
{
    // Set PA0-PA3 as inputs
    BTN_DDR &= ~((1 << BTN_LEFT) | (1 << BTN_RIGHT) | (1 << BTN_UP) | (1 << BTN_DOWN));
    // Enable internal pull-up resistors
    BTN_PORT |= (1 << BTN_LEFT) | (1 << BTN_RIGHT) | (1 << BTN_UP) | (1 << BTN_DOWN);
}

// Handle cursor movement based on button presses
void handle_cursor_movement(void)
{
    // Check Left button (move cursor left)
    if(!(BTN_PIN & (1 << BTN_LEFT)))
    {
        _delay_ms(50);  // Debounce
        if(!(BTN_PIN & (1 << BTN_LEFT)))
        {
            if(cursor_col > 0)
                cursor_col--;
            else if(cursor_row == 1)
            {
                cursor_row = 0;
                cursor_col = 15;
            }
            lcd_update_cursor();
            while(!(BTN_PIN & (1 << BTN_LEFT)));  // Wait for release
            _delay_ms(50);
        }
    }

    // Check Right button (move cursor right)
    if(!(BTN_PIN & (1 << BTN_RIGHT)))
    {
        _delay_ms(50);
        if(!(BTN_PIN & (1 << BTN_RIGHT)))
        {
            if(cursor_col < 15)
                cursor_col++;
            else if(cursor_row == 0)
            {
                cursor_row = 1;
                cursor_col = 0;
            }
            lcd_update_cursor();
            while(!(BTN_PIN & (1 << BTN_RIGHT)));
            _delay_ms(50);
        }
    }

    // Check Up button (move cursor up)
    if(!(BTN_PIN & (1 << BTN_UP)))
    {
        _delay_ms(50);
        if(!(BTN_PIN & (1 << BTN_UP)))
        {
            if(cursor_row == 1)
                cursor_row = 0;
            lcd_update_cursor();
            while(!(BTN_PIN & (1 << BTN_UP)));
            _delay_ms(50);
        }
    }

    // Check Down button (move cursor down)
    if(!(BTN_PIN & (1 << BTN_DOWN)))
    {
        _delay_ms(50);
        if(!(BTN_PIN & (1 << BTN_DOWN)))
        {
            if(cursor_row == 0)
                cursor_row = 1;
            lcd_update_cursor();
            while(!(BTN_PIN & (1 << BTN_DOWN)));
            _delay_ms(50);
        }
    }
}

// Update LCD cursor position based on tracking variables
void lcd_update_cursor(void)
{
    lcd_set_cursor(cursor_row, cursor_col);
}

// Initialize keypad
void keypad_init(void)
{
    KEYBOARD_DDR = 0x0F;
    KEYBOARD_PORT = 0xFF;
}

// Scan keypad
char keypad_scan(void)
{
    KEYBOARD_PORT = 0b11111110;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '1';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '2';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '3';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'A';

    KEYBOARD_PORT = 0b11111101;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '4';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '5';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '6';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'B';

    KEYBOARD_PORT = 0b11111011;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '7';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '8';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '9';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'C';

    KEYBOARD_PORT = 0b11110111;
    _delay_us(10);
    if((KEYBOARD_PIN & 0b00010000) == 0) return '*';
    if((KEYBOARD_PIN & 0b00100000) == 0) return '0';
    if((KEYBOARD_PIN & 0b01000000) == 0) return '#';
    if((KEYBOARD_PIN & 0b10000000) == 0) return 'D';

    return 0;
}

// LCD Functions
void lcd_init(void)
{
    _delay_ms(15);
    lcd_cmd(0x38);
    _delay_ms(1);
    lcd_cmd(0x0E);  // Display ON, Cursor ON (visible cursor)
    _delay_ms(1);
    lcd_cmd(0x06);
    _delay_ms(1);
    lcd_cmd(0x01);
    _delay_ms(2);
    lcd_cmd(0x80);
    _delay_ms(1);
}

void lcd_cmd(char cmd_out)
{
    data = cmd_out;
    PORTC &= ~(1 << rs);
    PORTC &= ~(1 << rw);
    PORTC |= (1 << en);
    _delay_ms(10);
    PORTC &= ~(1 << en);
    _delay_ms(10);
}

void lcd_data(char data_out)
{
    data = data_out;
    PORTC |= (1 << rs);
    PORTC &= ~(1 << rw);
    PORTC |= (1 << en);
    _delay_ms(10);
    PORTC &= ~(1 << en);
    _delay_ms(10);
}

void lcd_str(char *str)
{
    unsigned int i = 0;
    while(str[i] != '\0')
    {
        lcd_data(str[i]);
        i++;
    }
}

void lcd_clear(void)
{
    lcd_cmd(0x01);
    _delay_ms(2);
}

void lcd_set_cursor(char row, char col)
{
    char address;
    if(row == 0)
        address = 0x80 + col;
    else
        address = 0xC0 + col;
    lcd_cmd(address);
}
```

### Expected Result
1. Initial display shows instructions briefly, then clears
2. Visible cursor blinks at current position
3. Pressing Left/Right buttons moves cursor horizontally
4. Pressing Up/Down buttons moves cursor between rows
5. Pressing keypad buttons displays characters at cursor position
6. Cursor automatically advances after each character

---

## Wiring Diagram Summary

### Complete System Wiring (SparkFun ADM1602K LCD)

```
                    ATmega328P
                   +-----------+
    LCD D0 <------| PB0   PB7 |------> LCD D7
    LCD D1 <------| PB1   PB6 |------> LCD D6
    LCD D2 <------| PB2   PB5 |------> LCD D5
    LCD D3 <------| PB3   PB4 |------> LCD D4
                   |           |
    LCD RS <------| PC0       |
    LCD RW <------| PC1       |
    LCD E  <------| PC2       |
                   |           |
    Key R1 <------| PD0       |
    Key R2 <------| PD1       |
    Key R3 <------| PD2       |
    Key R4 <------| PD3       |
    Key C1 <------| PD4       |
    Key C2 <------| PD5       |
    Key C3 <------| PD6       |
    Key C4 <------| PD7       |
                   |           |
  Btn Left <------| PA0       |
  Btn Right<------| PA1       |
  Btn Up   <------| PA2       |
  Btn Down <------| PA3       |
                   +-----------+

LCD (ADM1602K) Power & Control Connections:
  ┌─────────────────────────────────────┐
  │ LCD Pin 1 (VSS)    ─────> GND       │
  │ LCD Pin 2 (VDD)    ─────> +5V       │
  │ LCD Pin 3 (V0)     ─────> 10k Pot   │
  │                        (wiper)      │
  │ LCD Pin 15 (A)     ─────> +5V       │
  │                        via 220Ω     │
  │ LCD Pin 16 (K)     ─────> GND       │
  └─────────────────────────────────────┘

Contrast Potentiometer Wiring:
      +5V ───┬───[10k Pot]───┬─── GND
             │               │
            ─┴─ (V0 to LCD) ─┴─

Pushbutton Connections (Active Low):
  Each button: Pin ───[Button]─── GND
  (Internal pull-up resistors enabled in code)
```

---

## Post-Lab Questions Answers

### Q1: How do you wire the LCD to the microcontroller?

The **SparkFun ADM1602K LCD** is wired using 8-bit parallel interface mode:

1. **Power Connections**:
   - Pin 1 (VSS) to GND
   - Pin 2 (VDD) to +5V

2. **Contrast Control**:
   - Pin 3 (V0) to the wiper of a 10k potentiometer
   - Potentiometer ends connect to VCC and GND
   - Adjust potentiometer to set display contrast (typically mid-position)

3. **Control Pins** (connect to PORTC):
   - Pin 4 (RS) to PC0 - Register Select
   - Pin 5 (R/W) to PC1 - Read/Write
   - Pin 6 (E) to PC2 - Enable

4. **Data Pins** (connect to PORTB):
   - Pin 7 (D0) to PB0
   - Pin 8 (D1) to PB1
   - Pin 9 (D2) to PB2
   - Pin 10 (D3) to PB3
   - Pin 11 (D4) to PB4
   - Pin 12 (D5) to PB5
   - Pin 13 (D6) to PB6
   - Pin 14 (D7) to PB7

5. **Backlight** (for ADM1602K with LED backlight):
   - Pin 15 (A/LED+) to +5V via 220Ω current-limiting resistor
   - Pin 16 (K/LED-) to GND

**Control Pin Functions**:
- **RS (Register Select)**: 0 = Command mode, 1 = Data mode
- **R/W (Read/Write)**: 0 = Write operation, 1 = Read operation
- **E (Enable)**: Data is latched on the falling edge (HIGH to LOW transition)

### Q2: How is keypad organized internally?

The 4x4 keypad is organized as a matrix of rows and columns:
- **4 rows** and **4 columns** create a grid of 16 intersection points
- Each button connects one row to one column when pressed
- **Scanning method**: The microcontroller sets one row LOW at a time while keeping others HIGH, then reads all columns. If a button is pressed on that row, the corresponding column reads LOW.
- **Pull-up resistors** on column pins keep them HIGH when no button is pressed
- This matrix configuration reduces the number of I/O pins needed from 16 (one per button) to just 8 (4 rows + 4 columns)

### Q3: What is the difference between 16x2 and 40x2 displays?

| Feature | 16x2 LCD | 40x2 LCD |
|---------|----------|----------|
| **Columns** | 16 characters per row | 40 characters per row |
| **Display size** | Smaller, compact | Larger, more text |
| **DDRAM addresses** | Row 1: 0x00-0x0F, Row 2: 0x40-0x4F | Row 1: 0x00-0x27, Row 2: 0x40-0x67 |
| **Visible area** | All characters visible | May require scrolling for full view |
| **Applications** | Simple status displays, small devices | Advanced UI, longer messages |
| **Command compatibility** | Same HD44780 commands | Same HD44780 commands |

Both use the same HD44780 controller and command set. The main difference is the memory addressing and visible character count.

---

## Troubleshooting Tips (SparkFun ADM1602K)

### LCD Issues

1. **LCD shows nothing (blank screen)**:
   - Check power connections (VSS to GND, VDD to +5V)
   - Adjust contrast potentiometer (most common issue!)
   - Verify backlight is working (should glow when powered)
   - Check all data and control pin connections

2. **LCD shows black boxes (solid rectangles) on top row**:
   - Contrast is set too high - adjust potentiometer
   - LCD not properly initialized - check initialization code
   - Check RS, R/W, and E pin connections

3. **LCD shows random/garbage characters**:
   - Verify data pin connections (D0-D7 to PB0-PB7)
   - Check for loose connections
   - Ensure proper timing delays in LCD functions
   - Verify 8-bit mode initialization (0x38 command)

4. **Backlight not working**:
   - Check LED+ (Pin 15) and LED- (Pin 16) connections
   - Verify 220Ω current-limiting resistor is in series with LED+
   - Some ADM1602K variants may have different backlight voltages

5. **Display flickers or is unstable**:
   - Increase delay times in LCD functions
   - Check power supply stability
   - Add decoupling capacitor (100µF) near LCD power pins

### Keypad Issues

6. **Keypad not responding**:
   - Check row connections (PD0-PD3) and column connections (PD4-PD7)
   - Verify internal pull-ups are enabled in code
   - Test each button with a multimeter

7. **Multiple/duplicate key presses detected**:
   - Add proper debounce delays (50-200ms)
   - Check for short circuits between row/column lines

8. **Wrong key detected**:
   - Verify row/column mapping matches code
   - Check keypad pinout against code assumptions
