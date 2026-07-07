# **Pi Game V1 - Raspberry Pi Pico Retro Game Console Setup Guide**

This guide covers the hardware wiring, library installations, and firmware flashing steps for the Raspberry Pi Pico 2 (RP2350) based handheld retro gaming console project using the Arduino IDE.

🚀 **Project Vision & Next Steps:** > For the next episode, I'll be upgrading to a 16MB flash board and adding a MAX98357A I2S amplifier for proper audio support. I'm also designing a custom PCB called **Pi Game V1**, which will be released as a fully open-source project so anyone can build their own handheld console from scratch! Stay tuned!

📺 Watch the Full Project Video on YouTube: <https://youtu.be/pmxH-TPf2Zc>

## **1\. Hardware Requirements**

- **Microcontroller:** Raspberry Pi Pico 2 (RP2350)
- **Display:** 2.8" ILI9341 TFT LCD (320x240 Resolution, SPI0)
- **Audio Amplifier:** MAX98357A I2S Digital Audio Amplifier
- **Storage:** Built-in or external Micro SD Card Module located on the back of the display (SPI1)
- **Buttons:** 8 directional and action buttons (configured with 10k Pull-Up resistors or directly via enabled internal pull-ups)

## **2\. Hardware Pin Mapping Table**

Refer to the table below when wiring your hardware components to the Pico 2 board.

⚠️ **Important Note:** "GPIO No" refers to the pin numbers defined within the software (code). "Physical Pin No" refers to the actual physical pin layout/sequence on the Pico 2 board (DIP package layout).

| **Module**              | **Pin Definition** | **GPIO No (Software)** | **Physical Pin No** | **Description**             |
| ----------------------- | ------------------ | ---------------------- | ------------------- | --------------------------- |
| **ILI9341 TFT Display** | PIN_TFT_CS         | 17                     | 22                  | Display Chip Select         |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_TFT_DC         | 20                     | 26                  | Display Data/Command Select |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_TFT_RST        | 21                     | 27                  | Display Reset               |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_TFT_MOSI       | 19                     | 25                  | SPI0 MOSI                   |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_TFT_SCK        | 18                     | 24                  | SPI0 Clock                  |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_TFT_LED        | 22                     | 29                  | Backlight Brightness (PWM)  |
| ---                     | ---                | ---                    | ---                 | ---                         |
| **Micro SD Card**       | PIN_SD_SCK         | 10                     | 14                  | SPI1 Clock                  |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_SD_MOSI        | 11                     | 15                  | SPI1 MOSI                   |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_SD_MISO        | 12                     | 16                  | SPI1 MISO                   |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_SD_CS          | 13                     | 17                  | SD Card Chip Select         |
| ---                     | ---                | ---                    | ---                 | ---                         |
| **MAX98357A Audio**     | PIN_I2S_BCLK       | 26                     | 31                  | I2S Bit Clock               |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_I2S_WS         | 27                     | 32                  | I2S Word Select (LRCLK)     |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_I2S_DIN        | 28                     | 34                  | I2S Data Input              |
| ---                     | ---                | ---                    | ---                 | ---                         |
| **Buttons**             | PIN_BTN_LEFT       | 0                      | 1                   | Left D-Pad Button           |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_RIGHT      | 1                      | 2                   | Right D-Pad Button          |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_UP         | 2                      | 4                   | Up D-Pad Button             |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_DOWN       | 3                      | 5                   | Down D-Pad Button           |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_START      | 4                      | 6                   | Start Button                |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_SELECT     | 5                      | 7                   | Select Button               |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_A          | 6                      | 9                   | Action Button A             |
| ---                     | ---                | ---                    | ---                 | ---                         |
|                         | PIN_BTN_B          | 7                      | 10                  | Action Button B             |
| ---                     | ---                | ---                    | ---                 | ---                         |

##

## **3\. Arduino IDE Software Setup**

### **Step 3.1: Adding the Additional Boards Manager URL**

We will use the Earle Philhower core to add RP2350 (Pico 2) support to the Arduino IDE.

- Open the **Arduino IDE**.
- Navigate to **File** > **Preferences** from the top menu.
- Paste the following URL into the **Additional Boards Manager URLs** field:
- <https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json>
- Click **OK** to save the settings.

### **Step 3.2: Installing the Pico 2 Board Package**

- Click the **Boards Manager** icon on the left sidebar (or go to **Tools** > **Board** > **Boards Manager**).
- Type Pico into the search bar.
- Locate the **"Raspberry Pi Pico/RP2040/RP2350"** (by Earle F. Philhower) package and click **Install** to get the latest version.

### **Step 3.3: Installing Required Libraries**

Open the **Library Manager** tab on the left sidebar, search for, and install the following libraries:

- **SdFat** (by Bill Greiman): Type SdFat into the search bar and install the latest version. _(Note: Choose the original Bill Greiman version, not the Adafruit fork)._
- **TFT_eSPI** (by Bodmer): Search for TFT_eSPI and install the package authored by Bodmer.

## **4\. TFT_eSPI Library Configuration**

To make the TFT_eSPI library compatible with the Pico 2 hardware SPI configuration, you must modify the User_Setup.h file located within the library directory.

- Go to your local Arduino libraries directory on your computer (Typically on Windows: Documents\\Arduino\\libraries\\TFT_eSPI\\).
- Open the User_Setup.h file using a text editor (Notepad, VS Code, etc.).
- Delete all the existing text inside the file, paste the configuration below, and save the file:

**C++**

# define USER_SETUP_LOADED

# define ILI9341_DRIVER

# define TFT_MISO -1 // MISO pin is not used for the display

# define TFT_MOSI 19

# define TFT_SCLK 18

# define TFT_CS 17

# define TFT_DC 20

# define TFT_RST 21

# define LOAD_GLCD

# define LOAD_FONT2

# define LOAD_FONT4

# define LOAD_FONT6

# define LOAD_FONT7

# define LOAD_FONT8

# define LOAD_GFXFF

# define SPI_FREQUENCY 60000000 // Stable 60 MHz SPI speed for Pico 2

# define SPI_READ_FREQUENCY 20000000

## **5\. Arduino IDE Tools Menu Settings**

Before flashing the firmware to your Pico 2, make sure the following parameters are correctly selected under the **Tools** menu:

- **Board:** Raspberry Pi Pico 2 (RP2350)
- **Flash Size:** 4MB (Sketch: 1MB, FS: 3MB) _(or choose 16MB depending on your specific Pico 2 module variation)_
- **CPU Speed:** 150 MHz _(you can select 200 MHz for a more stable/overclocked emulation performance)_
- **Optimize:** Optimize Even More (-O3) or Small (-Os) _(Selecting -O3 is highly recommended for stable 60 FPS gameplay)_
- **Port:** Select the matching COM port assigned to your device after connecting the Pico 2 via USB.

## **6\. Flashing the Firmware**

- Open the main code file with the .ino extension inside the Arduino IDE.
- Ensure that the companion dependency files pin_config.h and the emulator core file walnut_cgb.h are located in the exact same project folder directory.
- Click the **Verify** checkmark button in the upper left corner to compile the source code.
- Click the **Upload** arrow button to begin flashing the code to the hardware.

💡 **Upload Note:** If the Arduino IDE fails to auto-detect the COM port, press and hold down the **BOOTSEL** button on your Pico 2 while plugging the USB cable into your PC. Once the computer recognizes the board as a mass storage flash drive, click the upload button to flash the firmware. The COM port will automatically resolve itself during future code uploads.
