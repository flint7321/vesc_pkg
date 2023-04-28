# VESC Packages

This repository contains packages that show up in the package store in VESC Tool. A VESC Package consists of:

* A description, which can include formatted text and images.
* An optional LispBM-script with optional includes.
	- The includes can also contain one or more compiled C libraries.
* An optional QML-file.

Packages can be pulled and updated from VESC Tool on demand, so their development does not have to be synchronized with the VESC Tool development. This is an advantage for package developers and maintainers as they can control how and when they make their releases. There is also no need for users to update their VESC Tool and firmware, they can just refresh the package store in VESC Tool and reinstall the updated package.

## Package Types

There are two types of packages:

* Applications
* Libraries

At the moment they are implemented in the same way, the only difference is the naming and how you are supposed to use them. If you create a package or library you should make that clear in your description.

### Applications

Application packages are intended to be installed and used either without configuration, or with a configuration GUI that they provide. That GUI can either be a QML-file or a custom configuration page in VESC Tool.

### Libraries

Library packages are intended to be building blocks to be used from LisbBM-scripts. All libraries in this repository can be imported in LispBM-scripts without the need for installation.

To declare a package as a library it should be placed in a directory that starts with the name **lib_**. That is enough to tell VESC Tool to list it under libraries and to disable the install button when it is selected.

Libraries work by providing one or more files that can be imported in LispBM-scripts. These files can be anything, but are typically either LispBM-scripts or compiled native code that can be loaded after importing it. To make these files available they need a LispBM-script that imports all files meant to be provided in the VESC-package. Other LispBM-scripts can then import the files imported in the VESC-package.

**Example**

The WS2812-library wants to provide the file **c_lib/ws2812/ws2812.bin**. Therefore it contains a lisp-script with the line

```clj
(import "c_lib/ws2812/ws2812.bin" 'ws2812)
```

which means that it imports that file under the label ws2812. Once the VESC Package is made this file is included in the package and thus available to other lisp-scripts that want to use it.

Other lisp-scripts can then import that file by addressing the label ws2812 in that package:

```clj
(import "pkg::ws2812@://vesc_packages/lib_ws2812/ws2812.vescpkg" 'ws2812)
```

Here the syntax means

**pkg::** - Import a file from a VESC Package  
**ws2812** - The label in that package is ws2812  
**@://vesc_packages/lib_ws2812/ws2812.vescpkg** - The path of that VESC Package

The **@**-sign is followed by a path to a package which can be any path on the local file system. The packages in this git-repository can be accessed from the base path **://vesc_packages/** (because they are loaded as a Qt resource there), which is what we have done here.

Once the file is imported in your lisp-script you can use it as usual, e.g. in this case by loading it as a native library:

```clj
(load-native-lib ws2812) ; Loading this library provides some extensions to control ws2812 addressable LEDs
```

**Note**

VESC Tool does not automatically refresh the packages, it uses the ones that are downloaded and cached from the archive the last time. To update the archive you can use the **Update Archive**-button from the VESC Packages-page in VESC Tool.

## How to include your package

To include your package in VESC Tool you can make a pull request to this repository. Your pull request should include:

* A description of what your package does.
* Instructions on how to use it.
* If it contains compiled C libraries their source code must be included in the pull request under the GPL license (see the examples).

### Notes

* Make sure that you build a VESC Package file (\*.vescpkg) in the main directory of your package (e.g. euc/euc.vescpkg)
* Make sure to add your package to res_all.qrc in the root of the repository.
* Test your .vescpkg-file before making a pull request!
* How to generate code from settings.xml:

vesc_tool --xmlConfToCode settings.xml
