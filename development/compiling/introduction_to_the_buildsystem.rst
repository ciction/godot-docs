.. _doc_introduction_to_the_buildsystem:

Introduction to the buildsystem
===============================

.. highlight:: shell

Scons
-----

Godot uses `Scons <http://www.scons.org>`__ to build. We love it, we are
not changing it for anything else. We are not even sure other build
systems are up to the task of building Godot. We constantly get requests
to move the build system to CMake, or Visual Studio, but this is not
going to happen. There are many reasons why we have chosen SCons over
other alternatives and are listed as follows:

-  Godot can be compiled for a dozen different platforms. All PC
   platforms, all mobile platforms, many consoles, and many web-based
   platforms (such as HTML5 and Chrome PNACL).
-  Developers often need to compile for several of the platforms **at
   the same time**, or even different targets of the same platform. They
   can't afford reconfiguring and rebuilding the project each time.
   SCons can do this with no sweat, without breaking the builds.
-  SCons will *never* break a build no matter how many changes,
   configurations, additions, removals etc. You have more chances to die
   struck by lightning than needing to clean and rebuild in SCons.
-  Godot build process is not simple. Several files are generated by
   code (binders), others are parsed (shaders), and others need to offer
   customization (plugins). This requires complex logic which is easier
   to write in an actual programming language (like Python) rather than
   using a mostly macro-based language only meant for building.
-  Godot build process makes heavy use of cross compiling tools. Each
   platform has a specific detection process, and all these must be
   handled as specific cases with special code written for each.

So, please get at least a little familiar with it if you are planning to
build Godot yourself.

Platform selection
------------------

Godot's build system will begin by detecting the platforms it can build
for. If not detected, the platform will simply not appear on the list of
available platforms. The build requirements for each platform are
described in the rest of this tutorial section.

Scons is invoked by just calling ``scons``.

However, this will do nothing except list the available platforms, for
example:

::

    user@host:~/godot$ scons
    scons: Reading SConscript files ...
    No valid target platform selected.
    The following were detected:
        android
        server
        javascript
        windows
        x11

    Please run scons again with argument: platform=<string>
    scons: done reading SConscript files.
    scons: Building targets ...
    scons: `.' is up to date.
    scons: done building targets.

To build for a platform (for example, x11), run with the ``platform=`` (or just
``p=`` to make it short) argument:

::

    user@host:~/godot$ scons platform=x11

This will start the build process, which will take a while. If you want
scons to build faster, use the ``-j <cores>`` parameter to specify how many
cores will be used for the build. Or just leave it using one core, so you
can use your computer for something else :)

Example for using 4 cores:

::

    user@host:~/godot$ scons platform=x11 -j 4

Resulting binary
----------------

The resulting binaries will be placed in the bin/ subdirectory,
generally with this naming convention:

::

    godot.<platform>.[opt].[tools/debug].<architecture>[extension]

For the previous build attempt the result would look like this:

::

    user@host:~/godot$ ls bin
    bin/godot.x11.tools.64

This means that the binary is for X11, is not optimized, has tools (the
whole editor) compiled in, and is meant for 64 bits.

A Windows binary with the same configuration will look like this.

::

    C:\GODOT> DIR BIN/
    godot.windows.tools.64.exe

Just copy that binary to wherever you like, as it self-contains the
project manager, editor and all means to execute the game. However, it
lacks the data to export it to the different platforms. For that the
export templates are needed (which can be either downloaded from
`godotengine.org <http://godotengine.org>`, or you can build them yourself).

Aside from that, there are a few standard options that can be set in all
build targets, and will be explained as follows.

Tools
-----

Tools are enabled by default in al PC targets (Linux, Windows, OSX),
disabled for everything else. Disabling tools produces a binary that can
run projects but that does not include the editor or the project
manager.

::

    scons platform=<platform> tools=yes/no

Target
------

Target controls optimization and debug flags. Each mode means:

-  **debug**: Build with C++ debugging symbols, runtime checks (performs
   checks and reports error) and none to little optimization.
-  **release_debug**: Build without C++ debugging symbols and
   optimization, but keep the runtime checks (performs checks and
   reports errors). Official binaries use this configuration.
-  **release**: Build without symbols, with optimization and with little
   to no runtime checks. This target can't be used together with
   tools=yes, as the tools require some debug functionality and run-time
   checks to run.

::

    scons platform=<platform> target=debug/release_debug/release

This flag appends ".debug" suffix (for debug), or ".tools" (for debug
with tools enabled). When optimization is enabled (release) it appends
the ".opt" suffix.

Bits
----

Bits is meant to control the CPU or OS version intended to run the
binaries. It works mostly on desktop platforms and ignored everywhere
else.

-  **32**: Build binaries for 32 bits platform.
-  **64**: Build binaries for 64 bits platform.
-  **default**: Built whatever the build system feels is best. On Linux
   this depends on the host platform (if not cross compiling), while on
   Windows and Mac it defaults to produce 32 bits binaries unless 64
   bits is specified.

::

    scons platform=<platform> bits=default/32/64

This flag appends ".32" or ".64" suffixes to resulting binaries when
relevant.

Export templates
----------------

Official export templates are downloaded from the Godot Engine site:
`godotengine.org <http://godotengine.org>`. However, you might want
to build them yourself (in case you want newer ones, you are using custom
modules, or simply don't trust your own shadow).

If you download the official export templates package and unzip it, you
will notice that most are just optimized binaries or packages for each
platform:

::

    android_debug.apk
    android_release.apk
    javascript_debug.zip
    javascript_release.zip
    linux_server_32
    linux_server_64
    linux_x11_32_debug
    linux_x11_32_release
    linux_x11_64_debug
    linux_x11_64_release
    osx.zip
    version.txt
    windows_32_debug.exe
    windows_32_release.exe
    windows_64_debug.exe
    windows_64_release.exe

To create those yourself, just follow the instructions detailed for each
platform in this same tutorial section. Each platform explains how to
create its own template.

If you are working for multiple platforms, OSX is definitely the best
host platform for cross compilation, since you can cross-compile for
almost every target (except for winrt). Linux and Windows come in second
place, but Linux has the advantage of being the easier platform to set
this up.
