# smart-railway-announcement-controller
An embedded systems project using ARM7 LPC2148 to manage railway platform time display and train arrival/departure announcements using RTC, LCD, keypad, interrupt handling, LEDs, and buzzer.
🚆 Smart Railway Platform Controller

An embedded-systems project that simulates a smart railway platform information and announcement system using the LPC2148 (ARM7) microcontroller. It combines a real-time clock (RTC), a 16×2 LCD, a 4×4 keypad, an administrator menu, train schedule data, status LEDs, a buzzer, and an external-interrupt-driven admin switch.

Project type: Embedded C / ARM7
Target: LPC2148 / LPC21xx family
Development environment: Keil µVision with ARMCC and the LPC21xx device header
Display: 16×2 character LCD

Table of Contents

Overview

Features

System Block Diagram

System Operation Flowchart

Train Announcement Logic

Hardware and Interface Summary

Software Architecture

Repository Structure

How It Works

Build and Run

Configuration Notes

Possible Improvements

Overview

The controller displays the current time, day, and date on the LCD, then switches to train-announcement information. Train details are maintained in a database in firmware. The scheduler compares the RTC time against each train's updated arrival and departure times and chooses the information to display.

An administrator can use the external-interrupt switch to open a protected configuration dashboard. The keypad is used to enter the PIN and navigate the menu. Depending on the train status and schedule data, the controller operates green, yellow, and red indicator LEDs and can pulse the buzzer when a train is at the platform.

This is a microcontroller-based prototype. It does not connect to a live railway database or railway signalling network.

Features

Displays RTC time (HH:MM:SS), day, and date (DD/MM/YYYY).

Alternates between the RTC screen and train-information screen.

Stores train number, train name, destination, schedule, platform, delay, and status in a firmware train database.

Scrolls train name and destination text on the LCD.

Uses schedule comparison to identify upcoming, arrived, and departed trains.

Provides a keypad-operated administrator dashboard.

Uses an external interrupt on the admin switch to enter the dashboard.

Includes menu functions for editing RTC values, train information, and the admin PIN.

Controls status LEDs and a buzzer based on schedule/status logic.

System Block Diagram

flowchart TB
    PWR[Power Supply] --> MCU[LPC2148 ARM7 Microcontroller]

    RTC[On-chip Real-Time Clock] <--> MCU
    KP[4x4 Matrix Keypad] <--> MCU
    SW[Admin Switch] -->|EINT1 interrupt| MCU

    MCU --> LCD[16x2 LCD]
    MCU --> LEDG[Green LED]
    MCU --> LEDY[Yellow LED]
    MCU --> LEDR[Red LED]
    MCU --> BZ[Buzzer]

    DB[(Train Schedule Database<br/>in Firmware)] <--> MCU
    subgraph Firmware[Embedded C Firmware]
        APP[Main Application]
        SCH[Train Scheduler]
        ADM[Admin Dashboard]
        APP --> SCH
        APP --> ADM
    end
    MCU --- Firmware

System Operation Flowchart

flowchart TD
    A([Power ON / Reset]) --> B[Initialize RTC, LCD, keypad,<br/>LEDs, buzzer, scheduler and EINT1]
    B --> C[Show startup message]
    C --> D{{Main loop}}
    D --> E{Admin interrupt flag set?}
    E -- Yes --> F[Clear flag and open admin dashboard]
    F --> G[Return to normal display]
    E -- No --> H[Read RTC time, date and day]
    G --> H
    H --> I[Compare RTC time with train schedules]
    I --> J[Determine train status and indicator states]
    J --> K{Any train arrived?}
    K -- Yes --> L[Pulse buzzer]
    K -- No --> M[Continue]
    L --> M
    M --> N{Display mode}
    N -- RTC mode --> O[Show time, day and date]
    N -- Train mode --> P[Show train announcement and status]
    O --> Q[Short delay]
    P --> Q
    Q --> D

Train Announcement Logic

The scheduler uses the current RTC time and each train's updated arrival/departure time. The display routine also uses time windows to decide whether to cycle through trains or lock the display to a particular train.

flowchart TD
    A[Read current RTC time] --> B[Scan train database]
    B --> C{Within 5 minutes before<br/>a train arrival?}
    C -- Yes --> D[Window 2:<br/>Lock display to upcoming train]
    C -- No --> E{Train currently between<br/>arrival and 30 seconds<br/>after departure?}
    E -- Yes --> F[Window 3:<br/>Lock display to active train]
    E -- No --> G[Window 1:<br/>Cycle through train records]
    D --> H[Prepare LCD lines and status]
    F --> H
    G --> H
    H --> I[Update LCD announcement]

Status and indicator intent

Train condition

Firmware status / indicator intent

Before the updated arrival time

STATUS_ON_TIME

From arrival through departure

STATUS_ARRIVED

After departure

STATUS_DEPARTED

One or more records have delayMinutes > 0

Delayed indicator is enabled

A train is currently arrived

Arrival indicator and a short buzzer pulse are enabled

The exact electrical ON/OFF level depends on the board wiring and LED/buzzer circuit.

Hardware and Interface Summary

The following pin mapping is taken from the project's all_peripheral_defines.h. Confirm it against your actual board and schematic before wiring.

Peripheral

MCU interface in the source

Purpose

LCD data bus

P0.6–P0.13

Sends commands and characters to the 16×2 LCD

LCD RS

P0.17

Selects command or data register

LCD EN

P0.18

Latches LCD data

LCD RW

P0.19

LCD read/write control

Admin switch

P0.3 / EINT1

Requests entry to the admin dashboard

Keypad rows

P1.16–P1.19

Row scanning

Keypad columns

P1.20–P1.23

Key detection

Status LEDs

P1.27, P1.28, P1.29

On-time/approaching/delayed indication

Buzzer

P1.26

Audible train-arrival indication

RTC

LPC2148 internal RTC peripheral

Time and date reference

Hardware check: Some source comments and older project files may describe different pins. Use the actual definitions, schematic, and final wiring as the source of truth. Also verify that LCD and other peripheral pins do not conflict with your chosen board configuration.

Software Architecture

flowchart LR
    MAIN[application.c<br/>Main loop] --> RTC[rtc.c / rtc.h<br/>RTC driver]
    MAIN --> SCHED[scheduler.c / scheduler.h<br/>Train timing and display logic]
    MAIN --> DB[trainDB.c / trainDB.h<br/>Train records]
    MAIN --> LCD[lcd.c / lcd.h<br/>LCD driver]
    MAIN --> KPM[kpm.c / kpm.h<br/>Keypad driver]
    MAIN --> LED[led_buzz.c / led_buzz.h<br/>Indicators and buzzer]
    MAIN --> EINT[eint1_sw.c / eint1_sw.h<br/>Admin interrupt]
    MAIN --> MENU[menu.c / menu.h<br/>Admin configuration]
    MAIN --> DELAY[delay.c / delay.h<br/>Timing utilities]
    MENU --> KPM
    MENU --> LCD
    MENU --> RTC
    MENU --> DB

Important source modules

File

Responsibility

application.c

Initializes peripherals and runs the main application loop

scheduler.c, scheduler.h

Compares RTC time with train schedules and updates the announcement display

trainDB.c, trainDB.h

Defines the train structure, status constants, and train records

rtc.c, rtc.h

Initializes and accesses the LPC2148 RTC

lcd.c, lcd.h

LCD initialization, commands, text, and numeric output

kpm.c, kpm.h

Keypad scanning and numeric/PIN entry

menu.c, menu.h

Password check and admin configuration menus

eint1_sw.c, eint1_sw.h

EINT1 setup and interrupt flag handling

led_buzz.c, led_buzz.h

Status LED and buzzer control

delay.c, delay.h

Delay routines

all_peripheral_defines.h

Register bit definitions, macros, pin mappings, and clock constants

types.h

Project-specific type aliases

How It Works

Initialization: The firmware initializes the RTC, LCD, keypad, status outputs, scheduler, and EINT1.

Startup message: A welcome message is shown on the LCD.

RTC update: The application reads the current time, date, and day.

Schedule comparison: The scheduler checks each train's updated arrival and departure times.

Status outputs: The main loop evaluates train status and delay fields, then updates the LEDs and buzzer.

LCD modes: The LCD alternates between the clock/date screen and the train-announcement screen.

Admin access: An EINT1 interrupt sets a flag. The main loop detects it and calls the admin dashboard.

Configuration: After PIN verification, the admin menu can update supported RTC and train details.

Repository Structure

Project_No_0A/
├── application.c
├── all_peripheral_defines.h
├── delay.c
├── delay.h
├── eint1_sw.c
├── eint1_sw.h
├── kpm.c
├── kpm.h
├── lcd.c
├── lcd.h
├── led_buzz.c
├── led_buzz.h
├── menu.c
├── menu.h
├── rtc.c
├── rtc.h
├── scheduler.c
├── scheduler.h
├── trainDB.c
├── trainDB.h
├── types.h
└── Startup.s

The supplied project archive also contains Keil project files and generated build artifacts. Keep the source files and the project configuration needed to rebuild the firmware. Generated .o, .d, .crf, .axf, and similar build outputs are usually unnecessary in a source repository unless you specifically want to publish compiled artifacts.

Build and Run

Install Keil µVision with the appropriate ARM7/ARMCC toolchain and LPC2148 device support.

Open the supplied .uvproj project file in Keil µVision.

Verify the target device, startup assembly file, include paths, and source-file list.

Check that all project modules are included in the build.

Build the project and resolve any compiler/linker errors.

Connect the LPC2148 board to the LCD, keypad, admin switch, LEDs, and buzzer according to the final pin definitions and circuit schematic.

Flash the generated firmware to the board and power it on.

Confirm the RTC configuration, LCD output, keypad input, admin interrupt, and indicator behavior.

This repository uses device-specific registers and headers, so it is not expected to compile as a standard desktop C program with a generic GCC command.

Configuration Notes

The source currently defines a default admin PIN in application.c (save_pass). Change it before demonstrating or deploying the project.

Review RTC prescaler and clock settings for your oscillator and target board.

Validate the train entries and time format in trainDB.c.

Confirm the admin interrupt pin and keypad/LCD wiring before testing.

Review the status logic if you want to distinguish “approaching” from “arrived” more precisely.

The project is a prototype and should not be used for real railway operational or safety-critical control.

Possible Improvements

Store train schedules in non-volatile memory instead of relying only on compiled-in records.

Add robust calendar validation, including month lengths and leap years.

Add keypad debouncing and stronger PIN handling.

Separate delay/status policy from display formatting for easier testing.

Add unit tests for time-window boundaries and train selection.

Add a wiring schematic, board photographs, and an LCD demonstration video.

Add a .gitignore for generated build files.
