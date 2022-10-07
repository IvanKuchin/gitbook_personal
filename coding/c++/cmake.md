# CMake

## Tutorial

{% embed url="https://cmake.org/cmake/help/latest/guide/tutorial/index.html" %}

## Deny in-source build

```cmake
if ( ${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR} )
    message( FATAL_ERROR "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there. You may need to remove CMakeCache.txt." )
endif()
```

## Do your best in static linking

Add following to `CMakeLists.txt`

```cmake
set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++")
```

Check if `libgcc` and `libstdc++` is in ELF's relocation table

```
readelf -d book.cgi  | grep Shared
```

Output:

```
 0x0000000000000001 (NEEDED)             Shared library: [libmysqlclient.so.21]
 0x0000000000000001 (NEEDED)             Shared library: [libcrypto.so.1.1]
 0x0000000000000001 (NEEDED)             Shared library: [libGraphicsMagick++-Q16.so.12]
 0x0000000000000001 (NEEDED)             Shared library: [libcurl.so.4]
 0x0000000000000001 (NEEDED)             Shared library: [libmaxminddb.so.0]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [ld-linux-x86-64.so.2]
```

## Build options



| **Build type**                                                | **cmake  parameter**                | **Comment**                                 |
| ------------------------------------------------------------- | ----------------------------------- | ------------------------------------------- |
| <p><strong>release</strong><br><strong></strong>(default)</p> | -DCMAKE\_BUILD\_TYPE=release        | <p>Debug - off</p><p>optimization – on</p>  |
| **relwithdebinfo**                                            | -DCMAKE\_BUILD\_TYPE=relwithdebinfo | <p>Debug – on</p><p>optimization – on</p>   |
| **plain**                                                     | -DCMAKE\_BUILD\_TYPE=plain          | <p>Debug – off</p><p>Optimization – off</p> |
| **debug**                                                     | -DCMAKE\_BUILD\_TYPE=debug          | <p>Debug – on</p><p>Optimization - off</p>  |

## RPATH

More info here ([https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling)).

Linker joins obj-files to bundle. During run-time ELF-file must have pointers to all dynamic linked libraries. Dynamic libraries must load once executable file run. This is the job of the linker, usually ld.so:

Dynamic link libraries search list

* env LD\_LIBRARY\_PATH=/x/x/x; export LD\_LIBRARY\_PATH
* RPATH - a list of directories which is linked into the executable, supported on most UNIX systems. It is ignored if RUNPATH is present.
* LD\_LIBRARY\_PATH - an environment variable which holds a list of directories (
* RUNPATH - same as RPATH, but searched after LD\_LIBRARY\_PATH, supported only on most recent UNIX systems, e.g. on most current Linux systems
* /etc/ld.so.conf - configuration file for ld.so which lists additional library directories (/etc/ld.so.conf -> sudo ldconfig  (check: ldconfig -p))
* builtin directories - basically /lib and /usr/lib

&#x20;To configure RPATH configured in ELF-file during link process following options available. Makefile syntax:

```
-Wl,-rpath,../../lib
-Wl,-rpath,$(XL_LIBRARY_DIR)
```

CMakeLists.txt:

1\)      configure rpath in built file.

`set(CMAKE_INSTALL_RPATH ${XL_LIBRARY_DIR})`

2\)      keep RPATH once file INSTALL’ed by CMake.

`set_target_properties(agency.cgi PROPERTIES INSTALL_RPATH_USE_LINK_PATH TRUE)`

&#x20;Default CMake behavior:

{% hint style="warning" %}
By default, if you don't change any RPATH related settings, CMake will link the executables and shared libraries with full RPATH to all used libraries in the build tree. When installing, it will clear the RPATH of these targets, so they are installed with an empty RPATH.
{% endhint %}

&#x20;How to check:

Before installation:

> ikuchin@dev:\~/src/dev.timecard.su$ readelf -d <mark style="background-color:yellow;">build/src/pd/agency.cgi</mark>  &#x20;
>
> Dynamic section at offset 0x4f7d88 contains 34 entries:
>
> &#x20; Tag        Type                         Name/Value
>
> 0x0000000000000001 (NEEDED)             Shared library: \[libgcc\_s.so.1]
>
> &#x20;0x0000000000000001 (NEEDED)             Shared library: \[libc.so.6]
>
> &#x20;0x000000000000000f <mark style="background-color:yellow;">(RPATH)              Library rpath: \[/usr/local/lib:/usr/local/share/timecard.su/libxl/lib64:]</mark>
>
> &#x20;0x000000000000000c (INIT)               0x408de8
>
> &#x20;0x000000000000000d (FINI)               0x825760
>
> &#x20;0x0000000000000019 (INIT\_ARRAY)         0xaf7cb8

> &#x20;$ ldd build/src/pd/agency.cgi
>
> &#x20;       libmaxminddb.so.0 => /usr/local/lib/libmaxminddb.so.0 (0x00007f3d7410a000)
>
> &#x20;       libhpdf-2.2.1.so => /usr/lib/libhpdf-2.2.1.so (0x00007f3d73e53000)
>
> &#x20;       <mark style="background-color:yellow;">libxl.so => /usr/local/share/timecard.su/libxl/lib64/libxl.so</mark> (0x00007f3d72476000)
>
> &#x20;       libstdc++.so.6 => /usr/lib/x86\_64-linux-gnu/libstdc++.so.6 (0x00007f3d720f4000)
>
> &#x20;       libm.so.6 => /lib/x86\_64-linux-gnu/libm.so.6 (0x00007f3d71deb000)

After installation must be the same:

> $ readelf -d /home/httpd/dev.timecard.su/cgi-bin/agency.cgi  &#x20;
>
> Dynamic section at offset 0x4f7d88 contains 34 entries:
>
> &#x20; Tag        Type                         Name/Value
>
> 0x0000000000000001 (NEEDED)             Shared library: \[libgcc\_s.so.1]
>
> &#x20;0x0000000000000001 (NEEDED)             Shared library: \[libc.so.6]
>
> &#x20;0x000000000000000f <mark style="background-color:yellow;">(RPATH)              Library rpath: \[/usr/local/lib:/usr/local/share/timecard.su/libxl/lib64:]</mark>
>
> &#x20;0x000000000000000c (INIT)               0x408de8
>
> &#x20;0x000000000000000d (FINI)               0x825760
>
> &#x20;0x0000000000000019 (INIT\_ARRAY)         0xaf7cb8
>
> &#x20;
>
> ikuchin@dev:\~/src/dev.timecard.su$ ldd /home/httpd/dev.timecard.su/cgi-bin/agency.cgi
>
> &#x20;       libmaxminddb.so.0 => /usr/local/lib/libmaxminddb.so.0 (0x00007f3d7410a000)
>
> &#x20;       libhpdf-2.2.1.so => /usr/lib/libhpdf-2.2.1.so (0x00007f3d73e53000)
>
> &#x20;       <mark style="background-color:yellow;">libxl.so => /usr/local/share/timecard.su/libxl/lib64/libxl.so (0x00007f3d72476000)</mark>
>
> &#x20;       libstdc++.so.6 => /usr/lib/x86\_64-linux-gnu/libstdc++.so.6 (0x00007f3d720f4000)
>
> &#x20;       libm.so.6 => /lib/x86\_64-linux-gnu/libm.so.6 (0x00007f3d71deb000)

If installation removes RPATH most probably target property INSTALL\_RPATH\_USE\_LINK\_PATH set to FALSE. To get more information check build/src/pd/cmake\_install.cmake

> ikuchin@dev:\~/src/dev.timecard.su$ more build/src/pd/cmake\_install.cmake
>
> file(INSTALL DESTINATION "/home/httpd/dev.timecard.su/cgi-bin" TYPE EXECUTABLE FILES "/home/ikuchin/src/dev.timecard.su/build/src/pd/<mark style="background-color:yellow;">approver.cgi</mark>")
>
> &#x20; if(EXISTS "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/approver.cgi" AND
>
> &#x20;    NOT IS\_SYMLINK "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/approver.cgi")
>
> &#x20;   file(RPATH\_CHANGE
>
> &#x20;        FILE "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/approver.cgi"
>
> &#x20;        <mark style="background-color:yellow;">OLD\_RPATH "/usr/local/lib:/usr/local/share/timecard.su/libxl/lib64:"</mark>
>
> &#x20;        <mark style="background-color:yellow;">NEW\_RPATH ""</mark>)
>
> &#x20;   if(CMAKE\_INSTALL\_DO\_STRIP)
>
> &#x20;     execute\_process(COMMAND "/usr/bin/strip" "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/approver.cgi")
>
> &#x20;   endif()
>
> &#x20; endif()
>
> endif()
>
> &#x20;
>
> …… skip ……
>
> &#x20;
>
> file(INSTALL DESTINATION "/home/httpd/dev.timecard.su/cgi-bin" TYPE EXECUTABLE FILES "/home/ikuchin/src/dev.timecard.su/build/src/pd/<mark style="background-color:yellow;">agency.cgi</mark>")
>
> &#x20; if(EXISTS "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/agency.cgi" AND
>
> &#x20;    NOT IS\_SYMLINK "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/agency.cgi")
>
> &#x20;   file(RPATH\_CHANGE
>
> &#x20;        FILE "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/agency.cgi"
>
> &#x20;      <mark style="background-color:yellow;">OLD\_RPATH "/usr/local/lib:/usr/local/share/timecard.su/libxl/lib64:"</mark>
>
> &#x20;        <mark style="background-color:yellow;">NEW\_RPATH "/usr/local/lib:/usr/local/share/timecard.su/libxl/lib64"</mark>)
>
> &#x20;   if(CMAKE\_INSTALL\_DO\_STRIP)
>
> &#x20;     execute\_process(COMMAND "/usr/bin/strip" "$ENV{DESTDIR}/home/httpd/dev.timecard.su/cgi-bin/agency.cgi")
>
> &#x20;   endif()
>
> &#x20; endif()
>
> endif()

## **Ldconfig**

1. Copy library to `/usr/lib`&#x20;
2. Update ldd-cache: `sudo ldconfig`
3. Check: `ldconfig -p | grep xl`. Expect to see full path to the library
4. Compile ELF-file normally with dynamic library `-lxl`
5. Check that ELF-file has library linked
   1. `ldd agency.cgi | grep xl`
   2. `readelf -d agency.cgi | grep xl`&#x20;

## **Libra**r**y order in static linking**

Original white paper is here ([https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking](https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking))

* The linker maintains a symbol table. This symbol table does a bunch of things, but among them is keeping two lists:
  * A list of symbols exported by all the objects and libraries encountered so far.
  * A list of undefined symbols that the encountered objects and libraries requested to import and were not found yet.
* When the linker encounters a new object file, it looks at:
  * The symbols it exports: these are added to the list of exported symbols mentioned above. If any symbol is in the undefined list, it's removed from there because it has now been found. If any symbol has already been in the exported list, we get a "multiple definition" error: two different objects export the same symbol, and the linker is confused.
  * The symbols it imports: these are added to the list of undefined symbols, unless they can be found in the list of exported symbols.
* When the linker encounters a new library, things are a bit more interesting. The linker goes over all the objects in the library. For each one, it first looks at the symbols it exports.
  * If any of the symbols it exports are on the undefined list, the object is added to the link and the next step is executed. Otherwise, the next step is skipped.
  * If the object has been added to the link, it's treated as described above - its undefined and exported symbols get added to the symbol table.
  * Finally, if any of the objects in the library has been included in the link, the library is rescanned again - it's possible that symbols imported by the included object can be found in other objects within the same library.

Linker keep functions “symbol table” with

* Undefined – functions defined somwhere else
* Exported – functions exported form .o or .a (static lib)
* Internal – static functions

&#x20;

Symbol must be inquired (by application) and then reference must be resolved (via external obj or library), then application should always precede libraries in linker process.

`g++ application.o -lmylib1 -lmylib2`

&#x20;

To build library

```
$ gcc -c simple_func.c -o simple_func.o
$ ar r libsimple_func.a ./simple_func.o
ar: creating libsimple_func.a
$ ranlib ./libsimple_func.a
```

To check library content

```
$ ar t libsimple_func.a
simple_func.o
```

To check symbols exported by .o

```
$ nm x.o
000000000000000e T exported
                 U imported
0000000000000000 t internal
```

&#x20;Frequent problem is circular link. It could be resolved by one of the following options:

1. Most feasible: If two object files or .a (static libs) have hard links to each other, it is better to merge it to a single library
2. Makefile:\
   \--start-group archives --end-group - The specified archives are searched repeatedly until no new undefined references are created.\
   `$ gcc simplemain.o -L. -Wl,--start-group -lbar_dep -lfunc_dep -Wl,--end-group`
3. CMake\
   `target_link_libraries  (${target_executable} ${lib_1} ${lib_2} ${lib_1})`
4. CMake\
   Use LINK\_INTERFACE\_MULTIPLICITY - Repetition count for STATIC libraries with cyclic dependencies.\
   Or IMPORTED\_LINK\_INTERFACE\_MULTIPLICITY.\
   These target properties must be configured on \_libraries\_ , not on final application.\
   `set_target_properties(${_lib} PROPERTIES IMPORTED_LINK_INTERFACE_MULTIPLICITY 3)`

&#x20;
