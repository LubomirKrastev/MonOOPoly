<div align="center">

# 🎲 MonOOPoly

### A console-based Monopoly built in modern C++ as an exercise in object-oriented design.

![C++](https://img.shields.io/badge/C%2B%2B-MSVC%20v143-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![Visual Studio](https://img.shields.io/badge/Visual%20Studio-2022-5C2D91?style=for-the-badge&logo=visualstudio&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows%20x86%20%7C%20x64-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![No STL Containers](https://img.shields.io/badge/STL%20containers-not%20used-orange?style=for-the-badge)

<img src="https://github.com/user-attachments/assets/b5080b21-e36b-4bd1-b100-20d3dba51ce4" alt="Monopoly Banner" width="600"/>

</div>

---

## 📖 About

**MonOOPoly** is a faithful, console-driven implementation of the classic **Monopoly** board game on the **US edition** (40 fields, Mediterranean Avenue → Boardwalk). The capitalisation in the name highlights the project's real focus: **OOP** — inheritance, polymorphism, encapsulation, abstraction, the Singleton pattern, and a hand-written generic container, all without relying on the STL containers or `std::string`.

Players take turns rolling dice, moving around the board, buying properties, developing them with houses and hotels, paying rent, drawing Chance and Community Chest cards, and trying to bankrupt their opponents. The last solvent player wins.

---

## ✨ Features

- 👥 **2 – 6 players** in a single session, each starting with **$1500**
- 🎲 **Automatic dice rolling** with full doubles logic (3 doubles in a row → JAIL)
- 🏠 **Buying, building, mortgaging** — contextual prompts when you land on a property:
  - Unowned + affordable → **Buy? (y/n)**
  - Owned by you, monopoly complete → **build houses / hotels** (with even-build rule enforced)
  - Owned and mortgaged → **Unmortgage? (y/n)**
- 🚂 **Railroads** — Reading, Pennsylvania, B&O, Short Line
- ⚡ **Utilities** — Electric Company & Water Works (rent depends on dice roll; doubles when both owned)
- 💰 **Taxes** — Income Tax ($200) and Luxury Tax ($100)
- 🃏 **Chance & Community Chest** decks with multiple card categories (see [Card hierarchy](#-card-hierarchy))
- 🚔 **Jail mechanics** — Get Out of Jail Free cards, doubles roll, or pay
- 📉 **Mortgage / unmortgage** — half-price mortgage, 10 % interest to redeem
- 🧮 **Debt resolution flow** — when a player can't afford a payment, an interactive *Property Management* menu lets them sell buildings, mortgage properties, or unmortgage to raise funds before being declared bankrupt
- ⚰️ **Bankruptcy & asset transfer** — net-worth check, automatic transfer of all properties to the creditor (or back to the bank if the debt was to it)
- 💾 **Save / Load** — full game state to a plain text file
- 🏆 **Win detection** — game ends as soon as only one solvent player remains

---

## 🧠 OOP Design Highlights

### Field hierarchy

`Field` is an abstract base with a pure virtual `onPlayerLanding(Player&, int diceRoll)`. The main game loop never branches on the concrete type — it just calls the virtual method.

```
Field (abstract)
├── Property        // streets, railroads, utilities, taxes, GO, Free Parking, Jail (visiting)
├── CardField       // Chance / Community Chest squares
└── GoToJailField   // sends the player straight to jail
```

### Card hierarchy

`Card` is an abstract base with a pure virtual `applyEffect(Player&)`. Four concrete card kinds are implemented:

```
Card (abstract)
├── SpecialCard        // ADVANCE_TO_GO / ILLINOIS / BOARDWALK / RAILROAD / UTILITY / ST_CHARLES,
│                      // GO_TO_JAIL, GET_OUT_OF_JAIL, GO_BACK_3_SPACES
├── PaymentCard        // bank pays you / you pay the bank a fixed amount
├── GroupPaymentCard   // collect / pay a fixed amount from / to every other player
└── MovePositionCard   // move N spaces relative to current position
```

### Other design decisions

- **Singleton** — `Bank::getInstance()` guarantees a single bank with controlled construction and a `cleanup()` for tear-down.
- **Composition** — `Monopoly` owns the `Board`, the two `CardDeck`s, and the `Vector<Player*>`.
- **Custom generic container** — `Vector<T>` template with the rule of five (copy ctor, copy assign, move ctor, move assign, dtor) and explicit capacity management.
- **Custom string** — `MyString` replaces `std::string` and implements `operator+`, `operator+=`, `operator==`, stream `<<` / `>>`, etc.
- **Encapsulation** — every class hides state behind getters / setters; copy operations are explicitly deleted on classes that must not be copied (`Monopoly`, `Board`, `Bank`).
- **Extensible rent modifiers** — a small `Mortgage` base class with `Cottage` (×1.15) and `Castle` (×1.50) subclasses ready to plug into `Property::updateRent()`.
- **No STL containers** — the project deliberately avoids `std::vector`, `std::string`, and friends to demonstrate manual resource management.

---

## 🛠️ Tech Stack

| Layer       | Choice                       |
|-------------|------------------------------|
| Language    | C++ (MSVC v143 toolset)      |
| Build       | MSBuild (`.sln` / `.vcxproj`)|
| IDE         | Visual Studio 2022           |
| Platforms   | Windows x86 (Win32) & x64    |
| Configs     | Debug & Release              |

---

## 📁 Project Structure

```
MonOOPoly/
├── MonOOPoly/
│   ├── main.cpp                  # Entry point — New Game / Load Game menu
│   ├── Monopoly.{h,cpp}          # Game orchestrator + main loop
│   ├── Board.{h,cpp}             # Builds the 40-field US board and 8 color groups
│   ├── Player.{h,cpp}            # Balance, position, jail state, GOJF cards
│   ├── Bank.{h,cpp}              # Singleton; starting money, payments, bankruptcies
│   │
│   ├── Field.{h,cpp}             # Abstract base for any board square
│   ├── Property.{h,cpp}          # Streets / railroads / utilities / taxes / corners
│   ├── CardField.{h,cpp}         # Chance / Community Chest squares
│   ├── GoToJailField.{h,cpp}     # "Go to Jail" corner
│   ├── ColorGroup.{h,cpp}        # Color group + monopoly detection + house/hotel costs
│   │
│   ├── Card.{h,cpp}              # Abstract base for any deck card
│   ├── SpecialCard.{h,cpp}       # Move-to-named-field / jail / etc.
│   ├── PaymentCard.{h,cpp}       # Fixed receive-from / pay-to-bank
│   ├── GroupPaymentCard.{h,cpp}  # Collect / pay every other player
│   ├── MovePositionCard.{h,cpp}  # Relative move
│   ├── CardDeck.{h,cpp}          # Card container with shuffle / draw
│   │
│   ├── Mortgage.{h,cpp}          # Rent multipliers (Cottage / Castle)
│   ├── Trade.{h,cpp}             # Two-player property + cash trade model
│   │
│   ├── Vector.hpp                # Hand-rolled generic container (rule of 5)
│   ├── MyString.{h,cpp}          # Hand-rolled string class
│   │
│   └── MonOOPoly.vcxproj         # Visual Studio project
└── MonOOPoly.sln                 # Visual Studio solution
```

---

## 🚀 Build & Run

### Prerequisites

- **Windows 10 / 11**
- **Visual Studio 2022** with the *Desktop development with C++* workload

### Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/LubomirKrastev/MonOOPoly.git
   cd MonOOPoly
   ```
2. Open `MonOOPoly.sln` in Visual Studio 2022.
3. Pick a platform (**x86** or **x64**) and a configuration (**Debug** or **Release**).
4. Press **F5** to build and run, or **Ctrl + Shift + B** to build only.

The executable is produced under `x64/Debug/MonOOPoly.exe` (or the matching folder for your chosen config).

> 💡 Command-line build with MSBuild:
> ```bash
> msbuild MonOOPoly.sln /p:Configuration=Release /p:Platform=x64
> ```

---

## 🎮 How to Play

When you launch the executable you'll see:

```
1. New Game
2. Load Game
Choose option:
```

- Pick **1** → enter the number of players (2 – 6) and a name for each.
- Pick **2** → enter the name of a previously saved game file.

From that point, the game is **fully interactive prompt-driven** — there is no command parser. Each turn:

1. The current player's name, balance, and position are printed.
2. The dice are rolled automatically.
3. Depending on where you land, the game asks you the right question — for example:
   - On an unowned property: **`Buy? (y/n)`**
   - On a monopolised group: **`How many houses would you like to build? (0–N):`**
   - On a fully-developed group: **`Build a hotel for $X? (y/n)`**
   - On your own mortgaged property: **`Unmortgage for $X? (y/n)`**
4. Doubles grant another roll (third double in a row sends you to jail).
5. At the end of the round, the full game state is printed and you press **Enter** to continue.

### When you can't afford a payment

A dedicated **Property Management** sub-menu opens automatically:

```
1. Sell buildings
2. Mortgage property
3. Unmortgage property
0. Done
```

If your net worth is still insufficient after using these, you're declared bankrupt and your assets are transferred to the creditor (or the bank).

### Ending the game

When only one solvent player remains, the game prints a winner banner and then asks:

```
Would you like to save the game? (y/n):
```

## 👤 Author

**Lyubomir Krastev**

- GitHub: [@LubomirKrastev](https://github.com/LubomirKrastev)

---

## 📄 License & Trademarks

This project is a non-commercial, educational implementation made for studying object-oriented programming in C++.

*Monopoly* is a registered trademark of **Hasbro, Inc.** This project is not affiliated with, endorsed by, or sponsored by Hasbro.
