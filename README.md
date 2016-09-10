This is my personal mirror of the FreeType project. It mostly contains changes
to the build system to bring it more in-line with other tools I use and
integrate more cleanly into my other projects and build pipeline.

I don't expect this to be useful to anyone else and suggest using the upstream
repository instead.

# FreeType - Open Source Font Engine

FreeType is a freely available software library to render fonts.

It is written in C, designed to be small, efficient, highly customizable, and
portable while capable of producing high-quality output (glyph images) of most
vector and bitmap font formats.

* [FreeType Homepage](http://www.freetype.org)
* [Online Documentation](http://www.freetype.org/freetype2/documentation.html)

## Building

We're using CMake for managing the build. There are a number of CMake properties
that can be set to customize the build. Rather than documenting the various
flags here, I suggest using ccmake or cmake-gui to tweak them.

Build steps are bog-standard CMake and so a minimal example would be:

```
mkdir build && cd build
cmake ..
make
```

## Installation

You can install the project with just the basic `make install` after building,
or you can `make package` to generate an archive for the project that you can
install manually or distribute. There are CMake flags you can enable to create
more advanced installers as well such as Debian packages or Windows .MSI
installers.

