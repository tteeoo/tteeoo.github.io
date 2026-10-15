# software

## soundkit

A musical instrument consisting of
a liberating series of sound tools for a UNIX environment.

Compile from minimal C code to small, standalone binaries.
Construct signal chains with pipes on the command line.
Compose rhythms with signal chains in text files.

Needs GNU Gengetopt to compile.

Public domain.

[Hosted on GitHub.](https://github.com/tteeoo/soundkit)

### modules

Sources:
* adsrgen
* decode
* fmsynth
* noise
* wave

Filters
* coeff
* compress
* delay
* envelope
* hpf
* lpf
* negate

Pass-throughs:
* encode
* view

Hosts:
* newrcomp
* playback

### State of the toolchain

2026

soundkit set out with the almost silly but uniquely intuitive and flexible UI
constraint of implementing modularity through UNIX pipes.

This led it, upon culminating into rhythm composition, to have horrible
performance on mediocre hardware when using higher resolution subdivisions
and multiple instruments. Tens of processes would have to be spawned and killed
with an equally number of pipes, read from, in precise 1/64th-second intervals 
in order to sound legible.

A more practical design, preserving the UI, would be to detect the piping into 
the rhythm composer, and in that case, use the pipes to communicate the module
set-up of each instrument and then operate such a signal chain within the
rhythm composer as a thread.
