# physx-js-webidl
Javascript bindings for Nvidia PhysX 4.1.2 based on WebIDL.

This library is based on the awesome work made by [prestomation/PhysX](https://github.com/prestomation/PhysX) and
[ashconnell/physx-js](https://github.com/ashconnell/physx-js) and provides emscripten / WebIDL based
javascript bindings for [NVIDIAGameWorks/PhysX](https://github.com/NVIDIAGameWorks/PhysX) (with much greater library coverage
than the original projects).

Looking for pre-built binaries / build instructions? See [below](#pre-built-binaries)

## Getting started
There is a very basic [hello world example](dist/helloworld.html): No fancy graphics, only console output but it should get you started.

## Documentation
The API is very close to the original PhysX C++ API, so you can simply use the official
[PhysX API documentation](https://gameworksdocs.nvidia.com/PhysX/4.1/documentation/physxapi/files/index.html) and
[PhysX User's Guide](https://gameworksdocs.nvidia.com/PhysX/4.1/documentation/physxguide/Manual/Index.html).

## Demos
I also use this lib in my engine [kool](https://github.com/fabmax/kool) and have a few demos in place:
- [Ragdolls](https://fabmax.github.io/kool/kool-js/?demo=phys-ragdoll): A simple ragdoll demo.
- [Vehicle](https://fabmax.github.io/kool/kool-js/?demo=phys-vehicle): Basic vehicle demo with a few obstacles.
- [Joints](https://fabmax.github.io/kool/kool-js/?demo=phys-joints): A chain running over two gears.
- [Collision](https://fabmax.github.io/kool/kool-js/?demo=physics): The obligatory box (and other shapes) collision physics demo.

However, these are written in kotlin, not javascript.

## Pre-built binaries
This is published as a npm package:
```
npm i physx-js-webidl
```
Alternatively you can grab the pre-built binaries (.wasm + support .js) from the `dist` directory. In case you wanna dive deep, there
are also binaries from the profile and debug builds in corresponding `dist-profile` / `dist-debug` directories.

## What's the difference to physx-js?
Short answer: Much greater library coverage.

Long answer: Emscripten offers two methods to define javascript bindings for native projects: Embind (used by physx-js) and
WebIDL (used by this project). Embind basically is a C++ framework, which requires the javascript interfaces to be defined in hand-written C++.
WebIDL on the other hand uses an Interface Definition Language to generate the javascript interfaces automatically.

I initially tried to expose additional PhysX classes in the original Embind based project and found that quite tedious
and error-prone. In the end I decided to do a clean start and gave WebIDL a shot (this was also a bit inspired by
[ammo.js](https://github.com/kripken/ammo.js), a WebIDL based port of Bullet physics).

So far my expriences with WebIDL are:
- Much less hand-written C++ binding code (although it's not possible to omit that entirely)
- The .idl file used to define the bindings has a simple syntax and is easy to extend
- Speed seems to be the same as with Embind (I tested this quite a bit and really didn't see any difference)

However, there also are a few minor issues:
- Enums, used as configuration flags everywhere in PhysX, are a bit troublesome (see this
    [issue](https://github.com/emscripten-core/emscripten/issues/13243))
- WebIDL does not support top-level functions, so I had to wrap those in a class. This makes the bulk of the hand-written code.
- Overloaded functions (multiple functions with the same name but different parameters) are not supported.
- References to primititve types used as function output parameters are not supported (e.g. something like `void someFunction(int& output) { ... }`).
    

## Building
To build this you need the [emscripten SDK](https://emscripten.org/docs/getting_started/downloads.html). However,
with recent emscripten versions, box-box collisions sometimes result in weird behavior: Even for very low-speed collisions
the involved boxes bounce away from eacht other at very high speed (almost as if they would explode, at a larger scale
it looks a bit like popcorn). This only happens sometimes and only for box shapes, moreover only the relase build is
affected, so this is probably some compiler optimzation problem. The latest emscripten version not affected by this seems
to be 2.0.7. So make sure to use that one (or the docker method from below which also uses emscripten 2.0.7).
```
# Clone this repo
git clone https://github.com/fabmax/physx-js-webidl

# Enter that directory
cd physx-js-webidl

# Download submodule containing the PhysX code
git submodule update --init

# Generate build-scripts
./generate.sh

# Build
./make.sh
```

To add bindings to additional PhysX interfaces you only have to edit the
[PhysXJs.idl](https://github.com/fabmax/PhysX/blob/emscripten_webidl_wip/physx/source/physxwebbindings/src/PhysXJs.idl)
file located in `PhysX/physx/source/physxwebbindings/src/`.


## Build Types

It is also possible to generate Typescript bindings out of the idl file:

```
npx @milkshakeio/webidl2ts -e -d -n PhysX -i PhysX/physx/source/physxwebbindings/src/PhysXJs.idl -o dist/physx-js-webidl.wasm.d.ts
```

## Build with Docker

```
# Build the image
docker build . -t physx-js-builder

# Generate build-scripts
docker run --rm -it -v $(pwd):/src physx-js-builder /bin/bash -c ./generate.sh

# Build Release
docker run --rm -it -v $(pwd):/src physx-js-builder /bin/bash -c ./make.sh

# Build Profile
docker run --rm -it -v $(pwd):/src physx-js-builder /bin/bash -c ./make-profile.sh

# Build Debug
docker run --rm -it -v $(pwd):/src physx-js-builder /bin/bash -c ./make-debug.sh
```
