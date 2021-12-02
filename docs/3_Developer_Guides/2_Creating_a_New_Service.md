---
title: Creating a New Service
---

This tutorial provides instructions on **Creating a New Service** from scratch.

### 1. Create new project development directory

   ```sh
   $ cd ~/agl-app
   $ mkdir newservice/
   $ cd newservice/
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
  $ vim ~/agl-app/newservice/conf.d/cmake/config.cmake

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
    set(API_NAME "hellocount")    #INSERT NEW API NAME
    set(PROJECT_PRETTY_NAME "Example")
    set(PROJECT_DESCRIPTION "AGL hellocount application example")   # INSERT CONCISE PROJECT DESCRIPTION
    set(PROJECT_URL "https://gerrit.automotivelinux.org/gerrit/apps/cmake-apps-module")
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
    set(PROJECT_CMAKE_CONF_DIR "conf.d/cmake")

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
      afb-helpers
    )

    # You can also consider to include libsystemd
    # -----------------------------------
    #list (APPEND PKG_REQUIRED_LIST libsystemd>=222)

    if(IS_DIRECTORY $ENV{HOME}/opt/afb-monitoring)
    set(MONITORING_ALIAS "--alias=/monitoring:$ENV{HOME}/opt/afb-monitoring")
    endif()

    # Print a helper message when every thing is finished
    # ----------------------------------------------------
    set(CLOSING_MESSAGE "Debug from afb-daemon --port=1234 ${MONITORING_ALIAS} --ldpaths=package --workdir=. --roothttp=../htdocs --token= --verbose ")
    set(PACKAGE_MESSAGE "Install widget file using in the target : afm-util install ${PROJECT_NAME}.wgt")



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

    # (BUG!!!) as PKG_CONFIG_PATH does not work [should be an env variable]
    # ---------------------------------------------------------------------
    set(INSTALL_PREFIX $ENV{HOME}/opt)
    set(CMAKE_PREFIX_PATH ${CMAKE_INSTALL_PREFIX}/lib64/pkgconfig ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)
    set(LD_LIBRARY_PATH ${CMAKE_INSTALL_PREFIX}/lib64 ${CMAKE_INSTALL_PREFIX}/lib)

    # Location for config.xml.in template file.
    #
    # If you keep them commented then it will build with a default minimal widget
    # template which is very simple and it is highly probable that it will not suit
    # to your app.
    # -----------------------------------------
    #set(WIDGET_ICON "conf.d/wgt/${PROJECT_ICON}" CACHE PATH "Path to the widget icon")
    set(WIDGET_ICON ${PROJECT_APP_TEMPLATES_DIR}/wgt/${PROJECT_ICON})
    set(WIDGET_CONFIG_TEMPLATE ${CMAKE_SOURCE_DIR}/conf.d/wgt/config.xml.in CACHE PATH "Path to widget config file template (config.xml.in)")   # UNCOMMENT WIDGET_CONFIG_TEMPLATE
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
    set(WIDGET_ENTRY_POINT config.xml)    # UNCOMMENT WIDGET_ENTRY_POINT

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
  - Edit `config.xml` file
    ```sh
    $ vim conf.d/wgt/config.xml.in

      <?xml version="1.0" encoding="UTF-8"?>
      <widget xmlns="http://www.w3.org/ns/widgets" id="@PROJECT_NAME@" version="@PROJECT_VERSION@">
        <name>@PROJECT_NAME@</name>
        <icon src="@PROJECT_ICON@"/>
        <content src="@WIDGET_ENTRY_POINT@" type="@WIDGET_TYPE@"/>
        <description>@PROJECT_DESCRIPTION@</description>
        <author>@PROJECT_AUTHOR@ &lt;@PROJECT_AUTHOR_MAIL@&gt;</author>
        <license>@PROJECT_LICENSE@</license>
        <feature name="urn:AGL:widget:required-permission">
           <param name="urn:AGL:permission::partner:scope-platform" value="required" />
        </feature>
        <feature name="urn:AGL:widget:provided-api">
           <param name="hellocount" value="ws" />
        </feature>
        <feature name="urn:AGL:widget:required-binding">
           <param name="lib/afb-hellocount.so" value="local" />
        </feature>
      </widget>

    ```


### 6. Run CMAKE and Generate autobuild

  ```sh
  $ mkdir build/
  $ cd build/
  $ cmake ..
  $ make autobuild
  ```

### 7. Create service directory

  ```sh
  $ cd ..
  $ mkdir service/
  $ cd service/
  ```
  This `service` directory holds the C code & CMakeLists.txt ;

  - Add `hellocount-service.c` file which contains the functional C code :

    ```sh
    $ vim hellocount-service.c

      /*
       * Copyright 2020 The Linux Foundation # INSERT YEAR & ORG
       *
       * Licensed under the Apache License, Version 2.0 (the "License");
       * you may not use this file except in compliance with the License.
       * You may obtain a copy of the License at
       *
       *   http://www.apache.org/licenses/LICENSE-2.0
       *
       * Unless required by applicable law or agreed to in writing, software
       * distributed under the License is distributed on an "AS IS" BASIS,
       * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
       * See the License for the specific language governing permissions and
       * limitations under the License.
       */

      #define _GNU_SOURCE
      #include <stdio.h>
      #include <string.h>
      #include <json-c/json.h>

      #define AFB_BINDING_VERSION 3
      #include <afb/afb-binding.h>

      static void pingSample(afb_req_t request)
      {
        static int pingcount = 0;

        afb_req_success_f(request, json_object_new_int(pingcount), "Ping count = %d", pingcount);

        AFB_API_NOTICE(afbBindingV3root, "Verbosity macro at level notice invoked at ping invocation count = %d", pingcount);

        pingcount++;
      }

      static void count(afb_req_t request)
      {
        static int counter = 0;

        afb_req_success_f(request, json_object_new_int(counter), "Counter = %d", counter);

        AFB_API_NOTICE(afbBindingV3root, "Verbosity macro at level notice invoked at count invocation count = %d", counter);

        counter++;
      }



      static const afb_verb_t verbs[] =
      {
        /*Without security*/
        {.verb = "ping", .session = AFB_SESSION_NONE, .callback = pingSample, .auth = NULL},
        {.verb = "count", .session = AFB_SESSION_NONE, .callback = count, .auth = NULL},
        {NULL}
      };

      const afb_binding_t afbBindingExport =
      {
        .api = "hellocount",
        .specification = "HelloCount API",
        .verbs = verbs,
        .preinit = NULL,
        .init = NULL,
        .onevent = NULL,
        .userdata = NULL,
        .provide_class = NULL,
        .require_class = NULL,
        .require_api = NULL,
        .noconcurrency = 0
      };
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

      # Add target to project dependency list
      PROJECT_TARGET_ADD(hellocount)
          # Define project Targets
          ADD_LIBRARY(${TARGET_NAME} MODULE hellocount-service.c)

          # Binder exposes a unique public entry point
          SET_TARGET_PROPERTIES(${TARGET_NAME} PROPERTIES
          PREFIX "afb-"
          LABELS "BINDING"
          OUTPUT_NAME ${TARGET_NAME}
          )
    ```

### 8. Build and Package `wgt` using autobuild

  ```sh
  $ cd ..

  $ ./autobuild/agl/autobuild build

    make[1]: Entering directory '/home/boron/agl-app/newservice/build'
    make[2]: Entering directory '/home/boron/agl-app/newservice/build'
    make[3]: Entering directory '/home/boron/agl-app/newservice/build'
    make[3]: Leaving directory '/home/boron/agl-app/newservice/build'
    [100%] Built target autobuild
    make[2]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[1]: Leaving directory '/home/boron/agl-app/newservice/build'

  $ ./autobuild/agl/autobuild package

    make[1]: Entering directory '/home/boron/agl-app/newservice/build'
    make[2]: Entering directory '/home/boron/agl-app/newservice/build'
    make[3]: Entering directory '/home/boron/agl-app/newservice/build'
    make[3]: Leaving directory '/home/boron/agl-app/newservice/build'
    [100%] Built target autobuild
    make[2]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[1]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[1]: Entering directory '/home/boron/agl-app/newservice/build'
    make[2]: Entering directory '/home/boron/agl-app/newservice/build'
    make[3]: Entering directory '/home/boron/agl-app/newservice/build'
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    Scanning dependencies of target prepare_package
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    [  6%] Generating package
    [ 12%] Generating package/bin
    [ 18%] Generating package/etc
    [ 25%] Generating package/lib
    [ 31%] Generating package/htdocs
    [ 37%] Generating package/var
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    [ 37%] Built target prepare_package
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    Scanning dependencies of target prepare_package_test
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    [ 43%] Generating package-test
    [ 50%] Generating package-test/bin
    [ 56%] Generating package-test/etc
    [ 62%] Generating package-test/lib
    [ 68%] Generating package-test/htdocs
    [ 75%] Generating package-test/var
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    [ 75%] Built target prepare_package_test
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    Scanning dependencies of target populate
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    [ 75%] Built target populate
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    Scanning dependencies of target widget_files
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    [ 81%] Generating package/icon.png
    [ 87%] Generating package/config.xml
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    [ 93%] Built target widget_files
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    Scanning dependencies of target widget
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[4]: Entering directory '/home/boron/agl-app/newservice/build'
    [100%] Generating hellocount.wgt
    NOTICE: -- PACKING widget hellocount.wgt from directory /home/boron/agl-app/newservice/build/package
    ++ Install widget file using in the target : afm-util install hellocount.wgt
    make[4]: Leaving directory '/home/boron/agl-app/newservice/build'
    [100%] Built target widget
    make[3]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[2]: Leaving directory '/home/boron/agl-app/newservice/build'
    make[1]: Leaving directory '/home/boron/agl-app/newservice/build'
  ```

  - The `hellocount.wgt` file is packaged and available at `~/agl-app/newservice/build`.

### 9. Install the service

  - Local System :
    ```sh
    # Copy hellocount.wgt to remote AGL system
    $ scp ~/agl-app/newservice/build/hellocount.wgt root@<ip-address>:/home/0/
    ```

  - Remote AGL System :
    ```sh
    # Install using the service using afm-util
    $ afm-util install ./hellocount.wgt
    # Reboot the system
    $ reboot -f
    ```
