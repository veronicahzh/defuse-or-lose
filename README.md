# Defuse or Lose

A multiplayer bomb-defusal game about communication and teamwork, built at cmd-f 2026.

Every game spawns a randomly generated bomb with wires, toggles, buttons and a password. Each player privately gets a different piece of the solution, so nobody sees the full picture. One player might know which wire to cut, another knows the button code, and a third knows which wire must never be touched. You have to talk it through and defuse the bomb together before the timer runs out. One wrong cut and it's over.

## How to play

1. On the home screen, pick the number of players and a game mode, then tap START GAME.
2. Pass the phone around. Each player holds to reveal their secret rule(s), then taps NEXT PLAYER and hands it off. Don't let anyone else see your screen.
3. On the bomb console, work together to get everything right:
   - Wires: cut the right ones (e.g. "Cut wire 3", "Never cut wire 1", "Cut all red wires")
   - Toggles: flip them on or off (e.g. "hot should be on")
   - Buttons: enter the code in order (e.g. "Password is: 3142")
4. You win if every wire, toggle and the password match the solution. You lose if someone cuts a wire that wasn't supposed to be cut, or if the timer hits zero.

A wrong password resets the buttons automatically, and you can also press Reset to clear the code yourself.

## How it's built

The game logic is written in C++17 (`bomb`, `wire`, `player`, `solution`, `game_master`), and the app is React Native with Expo. We designed the UI in Figma.

The two halves talk through a small Node server in `bomb-mobile/backend/server.js`. When it starts, it compiles the C++ code into a program called `backend_bridge` and runs it in the background. Whenever you do something in the app, like cutting a wire, the app sends a request to the Node server, which passes it to the C++ program as a simple text command (`CUT 2`). The C++ side updates the game and sends back the new state as JSON, which the app then displays.

So the flow is just: app → Node server → C++ game logic → back to the app.

## Getting started

You'll need:

- Node.js 18+ and npm
- clang++ with C++17 support (on macOS, run `xcode-select --install`)
- Expo Go on your phone ([iOS](https://apps.apple.com/app/expo-go/id982107779) / [Android](https://play.google.com/store/apps/details?id=host.exp.exponent)), or an iOS Simulator / Android emulator

Clone the repo and install dependencies:

```bash
git clone https://github.com/veronicahzh/defuse-or-lose.git
cd defuse-or-lose/bomb-mobile
npm install
```

Start the backend:

```bash
# from bomb-mobile/
npm run backend
```

This compiles the C++ code and starts the server on `http://localhost:5050`.

Then, in a second terminal, start the app:

```bash
# from bomb-mobile/
npx expo start
```

From there you can scan the QR code with Expo Go (your phone needs to be on the same Wi-Fi as your computer), or press `i` for the iOS Simulator, `a` for Android, or `w` for web. The app should find the backend on its own.

## Running the C++ game on its own

You can also play from the terminal without the app:

```bash
# from the repo root
clang++ -std=c++17 -Wall -Wextra -pedantic \
  backend_bridge.cpp bomb.cpp solution.cpp wire.cpp player.cpp game_master.cpp \
  -o backend_bridge
./backend_bridge
```

Then type one command per line:

```
INIT 180 3      # new game: 180-second timer, 3 players
CUT 2           # cut wire #2
TOGGLE hot      # flip the "hot" toggle
BUTTON 4        # press button 4
RESET           # clear the entered password
STATE           # print current state
QUIT
```

To run the tests:

```bash
clang++ -std=c++17 game_master_test.cpp game_master.cpp bomb.cpp solution.cpp wire.cpp player.cpp -o game_master_test
./game_master_test
```

## Project structure

```
.
├── bomb.cpp / .h            # Bomb state: wires, toggles, buttons, entered password
├── wire.cpp / .h            # A single wire (color, cut or not)
├── player.cpp / .h          # Player name and assigned rules
├── solution.cpp / .h        # Generates the random rules/solution and checks win/lose
├── game_master.cpp / .h     # Runs the game: timer, players, status
├── backend_bridge.cpp       # Lets the Node server talk to the C++ code
├── game_master_test.cpp     # Tests for game_master
├── Makefile                 # Builds the terminal version (make, or make DEBUG=1)
└── bomb-mobile/             # The Expo / React Native app
    ├── app/                 # Screens
    ├── contexts/            # Shared game state
    ├── lib/gameApi.ts       # Talks to the backend
    └── backend/server.js    # Node server that runs the C++ code
```

`bomb-front/`, `bomb_front/` and `bomb-mobile/server/` are early prototypes from before the C++ connection was working. The actual game is in `bomb-mobile/`.

## What we learned

We chose C++ for the game logic on purpose, because we wanted real practice with object-oriented design instead of doing another frontend-only hackathon project. The hardest part was connecting everything. For a long time we had a fully working terminal game and a polished Figma design that had nothing to do with each other. Getting a wire cut on the phone to actually update the C++ game and show up in the UI was the moment it all came together, right in time for judging.

Our biggest takeaway: figure out how the frontend and backend will talk to each other early.

## Team

Built at cmd-f 2026 by:

- Veronica Zhang Harvey ([@veronicahzh](https://github.com/veronicahzh))
- Annie Wu ([@aw0521](https://github.com/aw0521))
- Jade Tao ([@jadetao25](https://github.com/jadetao25))
- Daxton Lee ([@DaxtonL](https://github.com/DaxtonL))
