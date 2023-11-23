---
title: Configure PCH in Visual Studio
author: ipa
date: 2023-10-09
categories: [Miscellaneous]
tags: [tools]
---

## Getting started with PCH in a VC++ project

1. Backup your `src` dir and any other directories that contain source code ... (you'll need this in case anything goes wrong)

2. Create 2 files in your project, `Globals.cpp` and `Globals.h` .. (choose any name but stick to it)

3. Right click `Globals.cpp` and open **Properties**, choose Configuration > **All configurations**

4. Go to C/C++ | Precompiled Header, and fill these in:

   - **Precompiled Header** : Create (/Yc)
   - **Precompiled Header File** : Globals.h

5. Open `Globals.cpp` and add this one line in, and *nothing more*: `#include "Globals.h"`

6. Right click your VC++ project and open **Properties**, choose Configuration > **All configurations**

7. Go to C/C++ | Precompiled Header, and fill these in:

   - **Precompiled Header** : Use (/Yu)
   - **Precompiled Header File** : Globals.h

8. Open all the `.h` and `.cpp` files in your project, and add this at the very top: `#include "Globals.h"`. If you DONOT want to include every file manually, you can use [the Force Include /FI name](https://stackoverflow.com/questions/41803417/how-to-include-an-header-automatically-in-all-cpp-files-in-visual-studio/41803602#41803602).

9. Open `Globals.h` and add the following in: (its very important you have `#pragma once` at the top)

   ```cpp
     #pragma once
   
     #include <stdio.h>
     #include <stdlib.h>
     #include <stdarg.h>
     #include <stddef.h>
     #include <memory>
     #include <string.h>
     #include <limits.h>
     #include <float.h>
     #include <time.h>
     #include <ctype.h>
     #include <wchar.h>
     #include <wctype.h>
     #include <malloc.h>
     #include <locale.h>
     #include <math.h>
   
     // Windows SDK
     #define _WIN32_WINNT 0x0501     // _WIN32_WINNT_WINXP
     #include <SDKDDKVer.h>
   
     // Windows API
     #define WIN32_LEAN_AND_MEAN
     #include <windows.h>
   ```

- These includes are typical candidates for your PCH file
- Remove the includes that you're not using
- Go through your project and collect any more header files that **do not change often**

1. Using find and replace, search for each of the `#include`'s in your PCH file, and remove them from all the `.h` and `.cpp` files in your project.

2. Do a full compile and ensure everything is working okay. Here are some solutions for common problems you'll encounter:

   1. **PCH file includes itself**

      Your PCH file is including a header that includes the PCH header file again, creating a kind of circular dependency. Double click the error to take you to the offending file, and simply remove the line that says `#include "Globals.h"`

   2. **Undefined symbol X**
   
      Although all your project files can include the PCH header, the files included *inside* the PCH header cannot include the PCH header! (as stated above) so you'll need to add back any imports that were previously in the file. (diff the file with the backup version)
   
   3. **Cannot find symbol logf**
   
      Sometimes the global PCH file does not behave as expected, and breaks compiling with crazy errors that are impossible to solve. You can then turn off PCH for individual source code files.
   
      1. Right click your `.cpp` file and open **Properties**, choose Configuration > **All configurations**
   
      2. Go to C/C++ | Precompiled Header, and fill these in:
         - **Precompiled Header** : Not Using Precompiled Headers
   
      3. Remove the line `#include "Globals.h"` in your `.cpp` file
   
      4. Add back whatever imports the file originally had. (diff the file with the backup version)

source: [c++ - Getting started with PCH in a VC++ project - Stack Overflow](https://stackoverflow.com/a/16746939)
