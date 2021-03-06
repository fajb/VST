# HOW TO BUILD:

1. Make sure you have the right version of Coq.  
   ```sh
   grep ^COQVERSION Makefile
   ```
   will tell you which versions are compatible.

2. Make sure you have the right version of CompCert.
   VST 2.0 uses CompCert 3.2 for Coq 8.7.1 (or Coq 8.7 should work).

### METHOD A [recommended]

This method bases the VST on a copy of certain CompCert specification files
distributed with VST, located in `VST/compcert`.

1. Execute this command:
   ```
   make
   ```  
   (or, if you have a multicore computer,  `make -j`)
2. *optional, only if you're going to run "clightgen"*  
    Unpack CompCert in the location of your choice (not under VST/), and in that
    directory,  
    ```
    ./configure -clightgen x86_32-linux
    make
    ```

Use x86_32-macosx if you're on a Mac, x86_32-cygwin on Windows.
You may also use x86_64, but VST has not been as heavily tested in
64-bit configurations.  You may also use other back ends besides x86,
but VST has not been much tested on those.


### METHOD B [alternate]

This method bases the VST on the same specification files
that the CompCert compiler is built upon (in contrast to method A,
which uses verbatim copies of them).

1. Unpack CompCert in a sibling directory to VST;  
   in that directory, build CompCert according to the instructions
   ```sh
    ./configure -clightgen x86_32-linux;
    make
    ```
(Use x86_32-macosx if you're on a Mac, etc.)

2. In the VST directory, create a file `CONFIGURE` containing exactly the text:  
   ```
   COMPCERT=../CompCert   # or whatever is your path to compcert
   ```
3. In the VST directory,  
   ```sh
   make
   ```

Note on the Windows (cygwin) installation of CompCert:
To build CompCert you'll need an up to date version of the
menhir parser generator: http://gallium.inria.fr/~fpottier/menhir/
To work around a cygwin incompatibility in the menhir build,
`touch src/.versioncheck` before doing `make`.

### METHOD A64, METHOD B64:   Sixty-four-bit (64 bit) build:
CompCert works with 64-bit architectures as well as 32-bit,
and VST now works with 64-bit or 32-bit CompCert.

Using method A, put  BITSIZE=64   (and nothing else)
in your CONFIGURE file, (do a fresh "make depend"),
and you'll get a 64-bit (x86_64) Verifiable C.

Using method B, put COMPCERT=your-compcert-directory  in your CONFIGURE file,
and in your-compcert-directory build with "./configure" specifying
a 64-bit architecture; and you'll get the corresponding 64-bit Verifiable C.
No need to specify BITSIZE in your CONFIGURE file.

In the standard VST distribution, in the progs/ and sha/ directories
there are .v files built by clightgen from .c files.  These are built
with a 32-bit clightgen, and will not be portable to 64-bit mode;
that is, they work if "make floyd" has been done with BITSIZE=32.

The progs64/ directory contains a subset of the .c files from progs/,
compiled in 64-bit mode to the corresponding .v files.  The files
progs64/verif_*.v are copied from progs/verif*.v with no change except
to replace "Import VST.progs.XXX" with "Import VST.progs64.XXX",
and will build only if "make floyd" has been done with BITSIZE=64.


--------------------------------------------------------------------------------

# ORGANIZATION:

The Verified Software Toolchain is organized into separate subprojects,
each in a separate directory:

- `msl` -   Mechanized Software Library
- `examples` - examples of how to use the msl (not ported to Coq 8.6)
- `compcert` -   front end of the CompCert compiler, specification of C light
- `sepcomp` - the theory and practice of how to specify shared-memory interaction
- `veric` -  program logic (and soundness proof) for Verifiable C
- `floyd` -  tactics for applying the separation logic
- `progs` -  sample programs, with their verifications

The dependencies are:

- `msl`:   _no dependency on other directories_
- `examples`: msl
- `compcert`: _no dependency on other directories_
- `sepcomp`: compcert
- `veric`:  msl compcert sepcomp
- `floyd`: msl sepcomp compcert veric
- `progs`: msl sepcomp compcert veric floyd

In general, we Import using `-Q` (qualified) instead of `-R`
(recursive).  This means modules need to be named using qualified names.
Thus, in `veric/expr.v` we write `Require Import msl.msl_standard`
instead of `Require Import msl_standard`.  To make this work, the loadpaths
need to be set up properly; the file `_CoqProject` (built by `make _CoqProject`)
shows what -I includes to use.

## USING VST:

To use either of these interactive development environments you will
need to have the right load path.  This can be done by command-line
arguments to coqide or coqtop.  The precise command-line arguments
to use when running CoqIDE or coqc are constructed automatically when
when you do "make", in the following files:

- `_CoqProject-export`: For VST users, running the IDE outside the VST directory
- `_CoqProject` : For VST developers, running the IDE in the VST directory

#### WITH COQIDE

From the VST root directory, run `./coqide` to run coqide with recommended options.
(Read the script for more info.)

#### WITH PROOF GENERAL

Use the `_CoqProject` file generated by the Makefile
   (Yes, we know, normally it's the other way 'round, normally one generates
    a Makefile from the `_CoqProject`.)

## NEW DIRECTORIES:

If you add a new directory, you will probably want to augment the loadpath
so that qualified names work right.  Edit the `OTHERDIRS` or `VSTDIRS` lines of
the `Makefile`.

## EXTERNAL COMPCERT:

The VST imports from the CompCert verified C compiler, the definition
of C light syntax and operational semantics.  For the convenience of
VST users, the `VST/compcert` directory is a copy (with permission) of
the front-end portions of compcert.  
You may choose to ignore the `VST/compcert` directory and have
the VST import from a build of compcert that you have installed in
another directory, for example,  `../CompCert`.

**This has not been tested recently, as of August 2017.**  
To do this, create a file `CONFIGURE` containing a definition such as,
  `COMPCERT=../CompCert`  
Make sure that you have the right version of CompCert!  Check
the file `VST/compcert/VERSION` to be sure.

## COMPCERT_NEW:
Starting in July 2018, for a limited period of (we hope) only a few months,
there is an experimental alternate CompCert basis in compcert_new.
To use this, define a CONFIGURE file containing  COMPCERT=compcert_new,
and make sure to do a "make depend" and "make clean" before (re)building.
WARNING:  When using compcert_new, the file veric/Clight_core.v
is not active; instead concurrency/shim/Clight_core.v is bound to the
module path VST.veric.Clight_core.
