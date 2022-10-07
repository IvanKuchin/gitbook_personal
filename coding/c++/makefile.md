# Makefile

## Do smth if process run

```makefile
ifeq ($(shell pgrep chat > /dev/null && echo 1 || echo 0), 1)
	@echo "process ran"
else
	@echo "process stopped"
endif
```

## Suppress warnings from set of include files

There are two options:

1\.       Define header as a “system include”:

> g++  -I  custom\_librabies  my\_source.cpp
>
> replace with
>
> g++  <mark style="background-color:yellow;">-isystem</mark> custom\_librabies  my\_source.cpp

or the same via pragma

> \#ifndef ROOTHEADERS\_HPP\_INCLUDED
>
> \#define ROOTHEADERS\_HPP\_INCLUDED
>
> \#ifdef \_\_GNUC\_\_
>
> // Avoid tons of warnings with root code
>
> <mark style="background-color:yellow;">#pragma GCC system\_header</mark>
>
> \#endif
>
> \#include "TH1F.h"
>
> \#include "TApplication.h"
>
> \#include "TGraph.h"
>
> \#include "TGraph2D.h"
>
> \#include "TCanvas.h"
>
> \#endif

{% hint style="warning" %}
This will change type of included files all way down till end of file.
{% endhint %}

2\.   Use pragma to filter diagnostic messages from single include file

```
#pragma GCC diagnostic push 
#pragma GCC diagnostic ignored "-Wunused-local-typedefs"
#include “codecvt/mbwcvt.hpp”
#pragma GCC diagnostic pop
```
