## How to build documentation

The procedure is validated on Windows and Ubuntu operation systems.

## Table of content:

* Installing dependencies
* Building documentation

## Installing dependencies

In addition to usual build dependencies `doxygen` and `latex` must be installed:

### Windows

* [miktex](https://miktex.org/)
* [doxygen](http://doxygen.nl/files/doxygen-1.8.20-setup.exe)
* [Graphviz](https://graphviz.org/download/)

### Ubuntu 18.04

* Latex, goxygen and graphviz

```sh
apt-get install texlive-full graphviz doxygen
```

## Building documentation

You should run `cmake` as usual and it will find all dependencies automatically on Ubuntu systems, while on Windows we still need to specify paths to the installed dependencies:

```sh
cmake -DLATEX_COMPILER="C:/Program Files/MiKTeX/miktex/bin/x64/latex.exe" \
      -DDOXYGEN_DOT_EXECUTABLE="C:/Program Files (x86)/Graphviz2.38/bin/dot.exe" \
      -DDOXYGEN_EXECUTABLE="C:/Program Files/doxygen/bin/doxygen.exe" \
```

Once the dependencies are found, the project must generated using CMake. The target `openvino_docs` must be built to generate doxygen documentation, the generated files can be found at `<binary dir>/docs/html/index.html`