Python iOS Support
==================

This is a meta-package for building a version of Python that can be embedded
into an iOS project.

It works by downloading, patching, and building a fat binary static libffi.a
and libPython.a, and packaging them both in iOS Framework format.

The ``site-packages`` has the `Rubicon Objective-C`_ library pre-installed.
This library enables you to have direct access to the iOS system libraries
from within the Python environment.

The binaries support the ``$(ARCHS_STANDARD_32_BIT)`` set - that is, armv7 and
armv7s. This should enable the code to run on:

* iPhone
    - iPhone 3GS,
    - iPhone 4
    - iPhone 4s
    - iPhone 5
    - iPhone 5s
* iPad
    - iPad 2
    - iPad (3rd gen)
    - iPad (4th gen)
    - iPad Air
* iPad Mini
    - iPad Mini (1st gen)
    - iPad Mini (2nd gen)
* iPod Touch
    - iPod Touch (4th gen)
    - iPod Touch (5th gen)

This repository branch builds a packaged version of **Python 2.7.1**.
Other Python versions are available by cloning other branches of the main
repository.

Pre-built Frameworks
--------------------

Pre-built versions of the frameworks can be downloaded_, and added to your iOS project.

.. _downloaded: https://github.com/pybee/Python-iOS-support/releases/download/2.7.1-b3/Python-2.7.1-iOS-support.b3.tar.gz

Building Frameworks
-------------------

Alternatively, to build the frameworks on your own, download/clone this repository, and then in the root directory, and run:

    $ make

This should:

1. Download the original source packages
2. Patch them as required for iOS compatibility
3. Build the packages as iOS frameworks.

The build products will be in the `build` directory. You'll need to add
**all** these frameworks (not just Python.framework) to your project.

Installation
------------

The Python Interpreter can be installed into your iOS project by following these steps:

1. Build/download the frameworks as mentioned above.
2. Add the frameworks built/downloaed (Python.framework & ffi.framework) to your project by dragging them from Finder into your xCode Project (check "Copy items if needed".)
3. In your project organiser, right click the Python.framework and select "Show in Finder". Navigate to Versions > 2.7 > Resources. Drag the two files ("include" & "lib") into your project (**uncheck** "Copy files if needed", **uncheck** your project as a target)
4. In Build Phases in your project settings link the following libraries:
- Python.framework
- ffi.framework
- libsqlite3.dylib
- libz.dylib
5. In Build Phases in your project settings add a New Run Script Phase. Leave the shell as `/bin/sh` and paste the following script in:

::
  rsync -pvtrL --exclude .hg --exclude .svn --exclude .git $PROJECT_DIR/Python.framework/Resources/lib $BUILT_PRODUCTS_DIR/$CONTENTS_FOLDER_PATH
  rsync -pvtrL --exclude .hg --exclude .svn --exclude .git $PROJECT_DIR/Python.framework/Resources/include $BUILT_PRODUCTS_DIR/$CONTENTS_FOLDER_PATH
  rsync -pvtrL --exclude .hg --exclude .svn --exclude .git $PROJECT_DIR/app $BUILT_PRODUCTS_DIR/$CONTENTS_FOLDER_PATH
  rsync -pvtrL --exclude .hg --exclude .svn --exclude .git $PROJECT_DIR/app_packages $BUILT_PRODUCTS_DIR/$CONTENTS_FOLDER_PATH

This script copies the necessary Python files into your app binary. If you find that your app compiles/archives, but fails to run correctly on devices, ensure that the paths above are correct for your project.

6.  In main.m, add the following imports:

.. code:: objc
  #import <Python/Python.h>
  #include <dlfcn.h>

7. Update your main function in main.m as follows:
.. code:: objc
  @autoreleasepool {
          int ret = 0;

  #if TARGET_IPHONE_SIMULATOR
          putenv("TARGET_IPHONE_SIMULATOR=1");
  #else
          putenv("TARGET_IPHONE=1");
  #endif

          //Setting the Python environment
          Py_SetProgramName(argv[0]);

          NSString * resourcePath = [[NSBundle mainBundle] resourcePath];

          Py_SetPythonHome((char *)[resourcePath UTF8String]);

          Py_Initialize();
          PySys_SetArgv(argc, argv);

          // If other modules are using thread, we need to initialize them before.
          PyEval_InitThreads();

          @try
          {
              // Start the Python app
              ret = UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
          }
          @catch (NSException *exception)
          {
              NSLog(@"Error running Python application: %@", exception.reason);
          }

          Py_Finalize();
          return ret;
      }

Running
-------

- To execute a Python script use `PyRun_SimplString();` or similar in your code.
- If you embed the scripts in the app you can add the scripts to your project ("/app/ProjectName/main.py" in this case), and then use this code to run the script:

.. code:: objc
  const char * prog = [[[NSBundle mainBundle] pathForResource:@"app/ProjectName/main" ofType:@"py"] cStringUsingEncoding:NSUTF8StringEncoding];
  FILE* fd = fopen(prog, "r");
  if (fd) {
      PyRun_SimpleFileEx(fd, prog, 1);
  }

Acknowledgements
----------------

This work draws on the groundwork provided by `Kivy's iOS packaging tools.`_

The approach to framework packaging is drawn from `Jeff Verkoeyen`_, and
`Ernesto García's`_ tutorials.

.. _Kivy's iOS packaging tools.: https://github.com/kivy/kivy-ios
.. _Jeff Verkoeyen: https://github.com/jverkoey/iOS-Framework
.. _Ernesto García's: http://www.raywenderlich.com/41377/creating-a-static-library-in-ios-tutorial
.. _Rubicon Objective-C: http://github.com/pybee/rubicon-objc
