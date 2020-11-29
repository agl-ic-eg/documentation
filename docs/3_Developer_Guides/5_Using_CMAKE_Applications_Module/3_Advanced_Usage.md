---
title: Advanced Usage
---

This topic describes some advanced ways of using the CMake templates.

## Building a Widget

To build a widget, you need a `config.xml` file that describes
your application (widget) and how the Application Framework launches it.
Your repository contains a simple default file named
`config.xml.in` that should work for simple applications and should
not require interactions with other bindings.

It is also recommended that you use the sample configuration
file that you can find in the location.
This file is named `config.xms.in.sample` and is more complete.
Copy the sample file to your `conf.d/wgt` directory and name it
`config.xml.in`.
Once you have your copy, edit the file to fit your needs.

**CAUTION:** The default file is only meant to be used for a
simple widget application.
For more complicated applications that need to export
their API, or ship several applications in one widget
need to use the provided `config.xml.in.sample` file, which has
all new Application Framework features explained and provides
examples.

## Using CMake Template Macros

To leverage all CMake template features, you must specify properties
on your targets.
Some macros do not work unless you specify the target type.
If you do not specify a type (e.g. a custom target such as an
HTML5 application), the macro uses the `LABELS` property to
determine the target type.

Aside from those values, the following optional values can be
assigned to the `LABELS` property.
These values define the resource types that make up your test materials:

- **TEST-CONFIG**: JSON configuration files used by the `afb-test`
 binding.
 These files execute the tests.
- **TEST-DATA**: Resources used to test your binding.
 Minimally, you need a test plan.
 You should also consider fixtures and any files required by your tests.
 These required files appear as part of a separate test widget.
- **TEST-PLUGIN**: A shared library used as a binding plugin.
 A binding loads the library as a plugin to extend the binding's functionality.
 You should use a special file extension when you name the library
 by using the `SUFFIX` CMake target property.
 If you do not choose an extension, `.ctlso` is used by default.
- **TEST-HTDOCS**: The root directory of a web application.
 This target has to build its directory and put its files in
 the `${CMAKE_CURRENT_BINARY_DIR}/${TARGET_NAME}` directory.
- **TEST-EXECUTABLE**: The entry point of your application executed by the AGL
 Application Framework.
- **TEST-LIBRARY**: An external third-party library bundled with the binding
 for its own purpose.
 The platform does not provide this library.

Following is a mapping between `LABELS` and directories where files reside in
the widget:

- **EXECUTABLE** : `<wgtrootdir>/bin`
- **BINDING-CONFIG** : `<wgtrootdir>/etc`
- **BINDING** | **BINDINGV2** | **BINDINGV3** | **LIBRARY** : `<wgtrootdir>/lib`
- **PLUGIN** : `<wgtrootdir>/lib/plugins`
- **HTDOCS** : `<wgtrootdir>/htdocs`
- **BINDING-DATA** : `<wgtrootdir>/var`
- **DATA** : `<wgtrootdir>/var`

Following is a mapping between test-dedicated `LABELS` and directories where
files reside in the widget:

- **TEST-EXECUTABLE** : `<wgtrootdir>/bin`
- **TEST-CONFIG** : `<TESTwgtrootdir`>/etc`
- **TEST-PLUGIN** : `<wgtrootdir>/lib/plugins`
- **TEST-HTDOCS** : `<wgtrootdir>/htdocs`
- **TEST-DATA** : `<TESTwgtrootdir>/var`

**TIP:** Use the prefix `afb-` (Application Framework Binding)
with your **BINDING** targets.

Following is an example that sets the `LABELS` and `OUTPUT_NAME` properties:

```cmake
SET_TARGET_PROPERTIES(${TARGET_NAME} PROPERTIES
		LABELS "HTDOCS"
		OUTPUT_NAME dist.prod
	)
```

**NOTE**: You do not need to specify an **INSTALL** command for these
  targets.
  Installation is handled by the template and installs using the
  following path : **${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}**

  Also, if you want to set and use `rpath` with your target, you should use
  and set the target property `INSTALL_RPATH`.

## Adding an External Third-Party Library

You can add an external third-party library that is built, linked,
and shipped with the project.
Or, you can link and ship the library only with the project.

### Building, Linking, and Shipping an External Library with the Project

If you need to include an external library that is not shipped
with the project, you can bundle the required library in the
`lib` widget directory.

Templates includes facilities to help you work with external
libraries.
A standard method is to declare as many CMake ExternalProject
modules as you need to match the number of needed libraries.

An ExternalProject module is a special CMake module that lets you define how
to download, update, patch, configure, build, and install an external project.
The project does not need to be a CMake project.
Additionally, you can provide custom steps to account for special
needs using ExternalProject step.
See the CMake
[ExternalProject documentation site](https://cmake.org/cmake/help/v3.5/module/ExternalProject.html?highlight=externalproject)
for more information.

Following is an example that includes the `mxml` library for the
[unicens2-binding](https://github.com/iotbzh/unicens2-binding)
project:

```cmake
set(MXML external-mxml)
set(MXML_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/mxml)
ExternalProject_Add(${MXML}
    GIT_REPOSITORY https://github.com/michaelrsweet/mxml.git
    GIT_TAG release-2.10
    SOURCE_DIR ${MXML_SOURCE_DIR}
    CONFIGURE_COMMAND ./configure --build x86_64 --host aarch64
    BUILD_COMMAND make libmxml.so.1.5
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND ""
)

PROJECT_TARGET_ADD(mxml)

add_library(${TARGET_NAME} SHARED IMPORTED GLOBAL)

SET_TARGET_PROPERTIES(${TARGET_NAME} PROPERTIES
    LABELS LIBRARY
    IMPORTED_LOCATION ${MXML_SOURCE_DIR}/libmxml.so.1
    INTERFACE_INCLUDE_DIRECTORIES ${MXML_SOURCE_DIR}
)

add_dependencies(${TARGET_NAME} ${MXML})
```

The example defines an external project that drives the building of the library.
The example also defines a new CMake target whose type is **IMPORTED**.
The **IMPORTED** target type indicates the target has yet to be built using
CMake but is available at the location defined using the **IMPORTED_LOCATION**
target property.

You might want to build the library as **SHARED** or **STATIC** depending on your needs
and goals.
Next, the example only has to modify the external project configure step and change
the filename used by **IMPORTED** library target defined after external project.

The target's **LABELS** property is set to **LIBRARY** to ship it in the widget.

In this example, the Unicens project also needs header
information from this library.
Consequently, the **INTERFACE_INCLUDE_DIRECTORIES** target property
is used.
Setting that property when another target links to that imported target
allows access to included directories.

Finally, the example binds the target to the external project
by using a CMake dependency.

The target can now be linked and used like any other CMake target.

### Link and Ship an External Library with the Project

If you already have a binary version of the library that you want to use and you
cannot or do not want to build the library, you can use the **IMPORTED**
library target.

To illustrate, consider the same example in the previous section.
Following are the relevant modifications:

```cmake
PROJECT_TARGET_ADD(mxml)

add_library(${TARGET_NAME} SHARED IMPORTED GLOBAL)

SET_TARGET_PROPERTIES(${TARGET_NAME} PROPERTIES
    LABELS LIBRARY
    IMPORTED_LOCATION /path_to_library/libmxml.so.1
    INTERFACE_INCLUDE_DIRECTORIES /path_to_mxml/include/dir
)
```

In the previous example, notice the changes to the
`IMPORTED_LOCATION` and `INTERFACE_INCLUDE_DIRECTORIES` statements.
These locate the binary version of the library.

Finally, you can link any other library or executable target with this imported
library just as you would for any other target.

## Macro Reference

Following are several macros that are useful for advanced CMake usage.

### PROJECT_TARGET_ADD

This macro adds the target to your project.
Following is a typical example that adds the target to your project.
You need to provide the name of your target as the macro's parameter:

Example:

```cmake
PROJECT_TARGET_ADD(low-can-demo)
```

The macro makes the variable `${TARGET_NAME}` available and it is defined
using the specified name (e.g. `low-can-demo`).
The variable changes each time the `PROJECT_TARGET_ADD` macro is called.

### project_subdirs_add

This macro searches within specified subfolders of the CMake project for
any `CMakeLists.txt` file.
If the file is found, it is added to your project.
You can use this macro in a hybrid application (e.g. where the binding
exists in a subfolder).

The following example searches within all subfolders:

Usage :

```cmake
project_subdirs_add()
```

You might want to restrict the subfolders being searched.
If so, you can specify a
[globbing](https://en.wikipedia.org/wiki/Glob_(programming)) pattern
as the argument.
Doing so effectively creates a search filter.

Following is an example that specifies all directories that begin
with a number, are followed by the dash character, and then followed
by any characters:

```cmake
project_subdirs_add("[0-9]-*")
```

### set_openapi_filename

This macro is used with a **BINDINGV2** target and defines the
binding definition filename.
You can use it to also define a relative path to
the current `CMakeLists.txt` file.

If you do not use this macro to specify the name of your definition file,
the default one is used, which is `${OUTPUT_NAME}-apidef` and uses
**OUTPUT_NAME** as the [target property].

**CAUTION** When specifying the binding definition filename,
you must not use the file's extension as part of the name.
Following is an example:

```cmake
set_openapi_filename('binding/mybinding_definition')
```

[target property]: https://cmake.org/cmake/help/v3.6/prop_tgt/OUTPUT_NAME.html "OUTPUT_NAME property documentation"

### add_input_files

This macro creates a custom target dedicated for HTML5 and data resource files.
The macro provides syntax and schema verification for different languages that
include LUA, JSON and XML.

Alongside the macro are tools used to check files.
You can configure the tools by setting the
following variables:

- XML_CHECKER: Uses **xmllint** that is provided with major linux distributions.
- LUA_CHECKER: Uses **luac** that is provided with major linux distributions.
- JSON_CHECKER: Currently, not used by any tools.

Following is an example:

```cmake
add_input_file("${MY_FILES_LIST}")
```

**NOTE**: If an issue occurs during the "check" step of the macro,
the build halts.