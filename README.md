# endless-sky-cmake

This repository contains a CMake build system for Endless Sky (https://endless-sky.github.io/), an open source space trading and combat game.

I set this up becuase I like to hack on Endless Sky and I am also a fan of the [CLion IDE](https://www.jetbrains.com/clion/), which uses CMake natively.

Currently, it is compatible with Endless Sky v0.9.8+.

## How To Use

  1. Clone Endless Sky: `git clone https://github.com/endless-sky/endless-sky.git`
  2. Download this repository and copy `CMakeLists.txt` and `cmake_modules` into the Endless Sky folder created in step 1.
  3. Build with CMake
  
## Dependencies

Endless Sky requires a number of dependencies. On Mac, I recommend installing these with [Homebrew](https://brew.sh/). The CMake configuration should locate them auto-magically. On Windows, appropraite development libraries and SDKs can be downloaded and that location should be specified in `CMakeLists.txt`. Please see the [Endless Sky Wiki](https://github.com/endless-sky/endless-sky/wiki/BuildInstructions) for more information on dependencies. 
