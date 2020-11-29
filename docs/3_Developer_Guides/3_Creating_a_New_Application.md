---
title: Creating a New Application
---

This tutorial provides instructions on **Creating a New Application** from scratch.

### 1. Create new project development directory

   ```sh
   $ cd ~/agl-app
   $ mkdir newapp/
   $ cd newapp/
   ```

### 2. Source the SDK environment setup

  ```sh
  $ source ~/agl-app/agl-sdk/environment-setup-corei7-64-agl-linux
  ```

### 3. Copy initial CMAKE configuration templates

  ```sh
  $ mkdir -p conf.d/cmake
  $ cp ${OECORE_NATIVE_SYSROOT}/usr/share/doc/CMakeAfbTemplates/samples.d/config.cmake.sample conf.d/cmake/config.cmake
  $ cp ${OECORE_NATIVE_SYSROOT}/usr/share/doc/CMakeAfbTemplates/samples.d/CMakeLists.txt.sample CMakeLists.txt
  ```

### 4. Edit CMAKE configuration template

  ```sh
  $ vim ~/agl-app/newapp/conf.d/cmake/config.cmake

    ###########################################################################
    # Copyright 2020 # INSERT YEAR
    #
    # author: John Doe <john.doe@example.com> # INSERT AUTHOR DETAILS
    #
    # Licensed under the Apache License, Version 2.0 (the "License");
    # you may not use this file except in compliance with the License.
    # You may obtain a copy of the License at
    #
    #     http://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.
    ###########################################################################

    # Project Info
    # ------------------
    set(PROJECT_NAME hellocount)    # INSERT NEW PROJECT NAME
    set(API_NAME hellocount)    #INSERT NEW API NAME
    set(PROJECT_PRETTY_NAME "Example helloworld count app") #INSERT PRETTY NAME
    set(PROJECT_DESCRIPTION "AGL hellocount application example")   # INSERT CONCISE PROJECT DESCRIPTION
    set(PROJECT_URL "https://git.somewhere.org/foo")
    set(PROJECT_ICON "icon.png")
    set(PROJECT_AUTHOR "John Doe")    # INSERT AUTHOR NAME
    set(PROJECT_AUTHOR_MAIL "john.doe@example.com")   # INSERT AUTHOR EMAIL
    set(PROJECT_LICENSE "APL2.0")
    set(PROJECT_LANGUAGES "C")
    set(PROJECT_VERSION "1.0.0")    # INSERT PROJECT VERSION

    # Which directories inspect to find CMakeLists.txt target files
    # set(PROJECT_SRC_DIR_PATTERN "*")

    # Where are stored the project configuration files
    # relative to the root project directory
    set(PROJECT_CMAKE_CONF_DIR "conf.d")

    # Compilation Mode (DEBUG, RELEASE, COVERAGE or PROFILING)
    # ----------------------------------
    set(BUILD_TYPE "RELEASE") # SELECT BUILD TYPE
    #set(USE_EFENCE 1)

    # Kernel selection if needed. You can choose between a
    # mandatory version to impose a minimal version.
    # Or check Kernel minimal version and just print a Warning
    # about missing features and define a preprocessor variable
    # to be used as preprocessor condition in code to disable
    # incompatibles features. Preprocessor define is named
    # KERNEL_MINIMAL_VERSION_OK.
    #
    # NOTE*** FOR NOW IT CHECKS KERNEL Yocto environment and
    # Yocto SDK Kernel version.
    # -----------------------------------------------
    #set (kernel_mandatory_version 4.8)
    #set (kernel_minimal_version 4.8)

    # Compiler selection if needed. Impose a minimal version.
    # -----------------------------------------------
    set (gcc_minimal_version 4.9)

    # PKG_CONFIG required packages
    # -----------------------------
    set (PKG_REQUIRED_LIST
      json-c
      afb-daemon
    )

    # You can also consider to include libsystemd
    # -----------------------------------
    #list (APPEND PKG_REQUIRED_LIST libsystemd>=222)

    # Prefix path where will be installed the files
    # Default: /usr/local (need root permission to write in)
    # ------------------------------------------------------
    #set(INSTALL_PREFIX /opt/AGL CACHE PATH "INSTALL PREFIX PATH")

    # Customize link option
    # -----------------------------
    #list(APPEND link_libraries -an-option)

    # Compilation options definition
    # Use CMake generator expressions to specify only for a specific language
    # Values are prefilled with default options that is currently used.
    # Either separate options with ";", or each options must be quoted separately
    # DO NOT PUT ALL OPTION QUOTED AT ONCE , COMPILATION COULD FAILED !
    # ----------------------------------------------------------------------------
    #set(COMPILE_OPTIONS
    # -Wall
    # -Wextra
    # -Wconversion
    # -Wno-unused-parameter
    # -Wno-sign-compare
    # -Wno-sign-conversion
    # -Werror=maybe-uninitialized
    # -Werror=implicit-function-declaration
    # -ffunction-sections
    # -fdata-sections
    # -fPIC
    # CACHE STRING "Compilation flags")
    #set(C_COMPILE_OPTIONS "" CACHE STRING "Compilation flags for C language.")
    #set(CXX_COMPILE_OPTIONS "" CACHE STRING "Compilation flags for C++ language.")
    #set(PROFILING_COMPILE_OPTIONS
    # -g
    # -O0
    # -pg
    # -Wp,-U_FORTIFY_SOURCE
    # CACHE STRING "Compilation flags for PROFILING build type.")
    #set(DEBUG_COMPILE_OPTIONS
    # -g
    # -ggdb
    # CACHE STRING "Compilation flags for DEBUG build type.")
    #set(COVERAGE_COMPILE_OPTIONS
    # -g
    # -O0
    # --coverage
    # CACHE STRING "Compilation flags for COVERAGE build type.")
    #set(RELEASE_COMPILE_OPTIONS
    # -O2
    # -pipe
    # -D_FORTIFY_SOURCE=2
    # -fstack-protector-strong
    # -Wformat -Wformat-security
    # -Werror=format-security
    # -feliminate-unused-debug-types
    # -Wl,-O1
    # -Wl,--hash-style=gnu
    # -Wl,--as-needed
    # -fstack-protector-strong
    # -Wl,-z,relro,-z,now
    # CACHE STRING "Compilation flags for RELEASE build type.")

    # Location for config.xml.in template file.
    #
    # If you keep them commented then it will build with a default minimal widget
    # template which is very simple and it is highly probable that it will not suit
    # to your app.
    # -----------------------------------------
    #set(WIDGET_ICON "conf.d/wgt/${PROJECT_ICON}" CACHE PATH "Path to the widget icon")
    set(WIDGET_CONFIG_TEMPLATE "${CMAKE_CURRENT_SOURCE_DIR}/conf.d/wgt/config.xml.in" CACHE PATH "Path to widget config file template (config.xml.in)")   # UNCOMMENT WIDGET_CONFIG_TEMPLATE
    #set(TEST_WIDGET_CONFIG_TEMPLATE "${CMAKE_CURRENT_SOURCE_DIR}/conf.d/wgt/test-config.xml.in" CACHE PATH "Path to the test widget config file template (test-config.xml.in)")

    # Mandatory widget Mimetype specification of the main unit
    # --------------------------------------------------------------------------
    # Choose between :
    #- text/html : HTML application,
    #	content.src designates the home page of the application
    #
    #- application/vnd.agl.native : AGL compatible native,
    #	content.src designates the relative path of the binary.
    #
    # - application/vnd.agl.service: AGL service, content.src is not used.
    #
    #- ***application/x-executable***: Native application,
    #	content.src designates the relative path of the binary.
    #	For such application, only security setup is made.
    #
    set(WIDGET_TYPE application/vnd.agl.service)    # UNCOMMENT WIDGET_TYPE

    # Mandatory Widget entry point file of the main unit
    # --------------------------------------------------------------
    # This is the file that will be executed, loaded,
    # at launch time by the application framework.
    #
    set(WIDGET_ENTRY_POINT hellocount)    # UNCOMMENT WIDGET_ENTRY_POINT

    # Optional dependencies order
    # ---------------------------
    #set(EXTRA_DEPENDENCIES_ORDER)

    # Optional Extra global include path
    # -----------------------------------
    #set(EXTRA_INCLUDE_DIRS)

    # Optional extra libraries
    # -------------------------
    #set(EXTRA_LINK_LIBRARIES)

    # Optional force binding Linking flag
    # ------------------------------------
    # set(BINDINGS_LINK_FLAG LinkOptions )

    # Optional force package prefix generation, like widget
    # -----------------------------------------------------
    # set(PKG_PREFIX DestinationPath)

    # Optional Application Framework security token
    # and port use for remote debugging.
    #------------------------------------------------------------
    #set(AFB_TOKEN   ""     CACHE PATH "Default binder security token")   # COMMENT AFB_TOKEN
    #set(AFB_REMPORT "1234" CACHE PATH "Default binder listening port")   # COMMENT AFB_REMPORT

    # Print a helper message when every thing is finished
    # ----------------------------------------------------
    set(CLOSING_MESSAGE "Typical binding launch: cd ${CMAKE_BINARY_DIR}/package \\&\\& afb-daemon --port=${AFB_REMPORT} --workdir=. --ldpaths=lib --roothttp=htdocs  --token=\"${AFB_TOKEN}\" --tracereq=common --verbose")
    set(PACKAGE_MESSAGE "Install widget file using in the target : afm-util install ${PROJECT_NAME}.wgt")

    # Optional schema validator about now only XML, LUA and JSON
    # are supported
    #------------------------------------------------------------
    #set(LUA_CHECKER "luac" "-p" CACHE STRING "LUA compiler")
    #set(XML_CHECKER "xmllint" CACHE STRING "XML linter")
    #set(JSON_CHECKER "json_verify" CACHE STRING "JSON linter")

    include(CMakeAfbTemplates)
  ```

### 5. Copy WGT configuration template

  ```sh
  $ mkdir -p conf.d/wgt
  $ cp ${OECORE_NATIVE_SYSROOT}/usr/share/doc/CMakeAfbTemplates/samples.d/config.xml.in.sample conf.d/wgt/config.xml.in
  ```

### 6. Run CMAKE and Generate autobuild

  ```sh
  $ mkdir build/
  $ cd build/
  $ cmake ..
  $ make autobuild
  ```

### 7. Create app directory

  ```sh
  $ cd ..
  $ mkdir app/
  $ cd app/
  ```
  This `app` directory holds the C++, qrc, qml code & CMakeLists.txt ;

  - Add `main.cpp` file which contains the functional C code :

    ```sh
    $ vim main.cpp

     /*
      * Copyright (C) 2016 The Qt Company Ltd. # INSERT YEAR & ORG
      *               2020 The Linux Foundation
      *
      * Licensed under the Apache License, Version 2.0 (the "License");
      * you may not use this file except in compliance with the License.
      * You may obtain a copy of the License at
      *
      *      http://www.apache.org/licenses/LICENSE-2.0
      *
      * Unless required by applicable law or agreed to in writing, software
      * distributed under the License is distributed on an "AS IS" BASIS,
      * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      * See the License for the specific language governing permissions and
      * limitations under the License.
      */

      #include <QGuiApplication>
      #include <QtCore/QCommandLineParser>
      #include <QtCore/QUrlQuery>
      #include <QtGui/QGuiApplication>
      #include <QtQml/QQmlContext>
      #include <QtQml/QQmlApplicationEngine>
      #include <QtQml/qqml.h>
      #include <QDebug>

      int main(int argc, char *argv[])
      {
              QGuiApplication app(argc, argv);
              app.setDesktopFileName("Hellocount");

          QQmlApplicationEngine engine;
          QQmlContext *context = engine.rootContext();

          QCommandLineParser parser;
          parser.addPositionalArgument("port", app.translate("main", "port for binding"));
          parser.addPositionalArgument("secret", app.translate("main", "secret for binding"));
          parser.addHelpOption();
          parser.addVersionOption();
          parser.process(app);
          QStringList positionalArguments = parser.positionalArguments();

          if (positionalArguments.length() == 2) {
        int port = positionalArguments.takeFirst().toInt();
        QString secret = positionalArguments.takeFirst();

        QUrl bindingAddress;
        QUrlQuery query;

        bindingAddress.setScheme(QStringLiteral("ws"));
        bindingAddress.setHost(QStringLiteral("localhost"));
        bindingAddress.setPort(port);
        bindingAddress.setPath(QStringLiteral("/api"));


        query.addQueryItem(QStringLiteral("token"), secret);
        bindingAddress.setQuery(query);
        context->setContextProperty(QStringLiteral("bindingAddress"), bindingAddress);
          }

          engine.load(QUrl(QStringLiteral("qrc:/Hellocount.qml")));
              return app.exec();

      }
    ```

  - Add `Hellocount.qml` file which contains the functional qml code :

    ```sh
    $ vim Hellocount.qml

      /*
      * Copyright 2020 The Linux Foundation # INSERT YEAR & ORG
      *
      * Licensed under the Apache License, Version 2.0 (the "License");
      * you may not use this file except in compliance with the License.
      * You may obtain a copy of the License at
      *
      *      http://www.apache.org/licenses/LICENSE-2.0
      *
      * Unless required by applicable law or agreed to in writing, software
      * distributed under the License is distributed on an "AS IS" BASIS,
      * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      * See the License for the specific language governing permissions and
      * limitations under the License.
      */

      import QtQuick 2.6
      import QtQuick.Layouts 1.1
      import QtQuick.Controls 2.0
      import AGL.Demo.Controls 1.0

      import QtQuick.Window 2.13

      ApplicationWindow {

          // ----- Setup
          id: root
          width: Window.width * roles.scale
          height: Window.height * roles.scale

          // ----- Childs
          Label {
        id: title
        font.pixelSize: 48
        text: "Hello World Counter"
        anchors.horizontalCenter: parent.horizontalCenter
          }

      }
    ```

- Add `Hellocount.qrc` file :

    ```sh
    $ vim Hellocount.qrc

      <RCC>
          <qresource prefix="/">
              <file>Hellocount.qml</file>
          </qresource>
      </RCC>
    ```

- Add `CMakeLists.txt` file :

    ```sh
    $ vim CMakeLists.txt

      ###########################################################################
      # Copyright 2020 The Linux Foundation # INSERT YEAR & ORG
      #
      # Licensed under the Apache License, Version 2.0 (the "License");
      # you may not use this file except in compliance with the License.
      # You may obtain a copy of the License at
      #
      #     http://www.apache.org/licenses/LICENSE-2.0
      #
      # Unless required by applicable law or agreed to in writing, software
      # distributed under the License is distributed on an "AS IS" BASIS,
      # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      # See the License for the specific language governing permissions and
      # limitations under the License.
      ###########################################################################

      set(CMAKE_INCLUDE_CURRENT_DIR ON)
      set(CMAKE_AUTOMOC ON)
      set(CMAKE_AUTORCC ON)
      set(CMAKE_CXX_STANDARD 14)
      set(CMAKE_CXX_STANDARD_REQUIRED ON)
      set(OE_QMAKE_PATH_EXTERNAL_HOST_BINS $ENV{OE_QMAKE_PATH_HOST_BINS})

      find_package(Qt5 COMPONENTS Core Gui QuickControls2 QuickWidgets WebSockets REQUIRED)

      PROJECT_TARGET_ADD(hellocount)

      add_executable(hellocount
          "main.cpp"
          "Hellocount.qrc"
      )

      set_target_properties(hellocount PROPERTIES
          LABELS "EXECUTABLE"
          PREFIX ""
          COMPILE_FLAGS " -DFOR_AFB_BINDING"
          LINK_FLAGS "${BINDINGS_LINK_FLAG}"
          OUTPUT_NAME "${TARGET_NAME}"
      )

      target_link_libraries(hellocount
          Qt5::WebSockets
          Qt5::QuickWidgets
          Qt5::QuickControls2
          json-c
          libafb-helpers-qt.a
      )
    ```

### 8. Build and Package `wgt` using autobuild

  ```sh
  $ cd ..

  $ ./autobuild/agl/autobuild build

    make[1]: Entering directory '/home/boron/agl-app/newapp/build'
    make[2]: Entering directory '/home/boron/agl-app/newapp/build'
    make[3]: Entering directory '/home/boron/agl-app/newapp/build'
    make[3]: Leaving directory '/home/boron/agl-app/newapp/build'
    [100%] Built target autobuild
    make[2]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[1]: Leaving directory '/home/boron/agl-app/newapp/build'


  $ ./autobuild/agl/autobuild package

    make[1]: Entering directory '/home/boron/agl-app/newapp/build'
    make[2]: Entering directory '/home/boron/agl-app/newapp/build'
    make[3]: Entering directory '/home/boron/agl-app/newapp/build'
    make[3]: Leaving directory '/home/boron/agl-app/newapp/build'
    [100%] Built target autobuild
    make[2]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[1]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[1]: Entering directory '/home/boron/agl-app/newapp/build'
    make[2]: Entering directory '/home/boron/agl-app/newapp/build'
    make[3]: Entering directory '/home/boron/agl-app/newapp/build'
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    Scanning dependencies of target prepare_package
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    [  6%] Generating package
    [ 12%] Generating package/bin
    [ 18%] Generating package/etc
    [ 25%] Generating package/lib
    [ 31%] Generating package/htdocs
    [ 37%] Generating package/var
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    [ 37%] Built target prepare_package
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    Scanning dependencies of target prepare_package_test
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    [ 43%] Generating package-test
    [ 50%] Generating package-test/bin
    [ 56%] Generating package-test/etc
    [ 62%] Generating package-test/lib
    [ 68%] Generating package-test/htdocs
    [ 75%] Generating package-test/var
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    [ 75%] Built target prepare_package_test
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    Scanning dependencies of target populate
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    [ 75%] Built target populate
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    Scanning dependencies of target widget_files
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    [ 81%] Generating package/icon.png
    [ 87%] Generating package/config.xml
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    [ 93%] Built target widget_files
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    Scanning dependencies of target widget
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[4]: Entering directory '/home/boron/agl-app/newapp/build'
    [100%] Generating hellocount.wgt
    NOTICE: -- PACKING widget hellocount.wgt from directory /home/boron/agl-app/newapp/build/package
    ++ Install widget file using in the target : afm-util install hellocount.wgt
    make[4]: Leaving directory '/home/boron/agl-app/newapp/build'
    [100%] Built target widget
    make[3]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[2]: Leaving directory '/home/boron/agl-app/newapp/build'
    make[1]: Leaving directory '/home/boron/agl-app/newapp/build'
  ```

  - The `hellocount.wgt` file is packaged and available at `~/agl-app/newapp/build`.

### 9. Install the service

  - Local System :
    ```sh
    # Copy hellocount.wgt to remote AGL system
    $ scp ~/agl-app/newapp/build/hellocount.wgt root@<ip-address>:/home/0/
    ```

  - Remote AGL System :
    ```sh
    # Install using the service using afm-util
    $ afm-util install ./hellocount.wgt
    # Reboot the system
    $ reboot -f
    ```