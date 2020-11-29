---
title: Options of afb-daemon
---

The launch options for binder **afb-daemon** are:

```
 -v, --verbose           Verbose Mode, repeat to increase verbosity
 -c, --color             Colorize the ouput
 -q, --quiet             Quiet Mode, repeat to decrease verbosity
 -l, --log=xxxx          Tune log level
     --foreground        Get all in foreground mode
     --background        Get all in background mode
 -D, --daemon            Get all in background mode
 -n, --name=xxxx         Set the visible name
 -p, --port=xxxx         HTTP listening TCP port  [default 1234]
     --roothttp=xxxx     HTTP Root Directory [default no root http (files not served but apis still available)]
     --rootbase=xxxx     Angular Base Root URL [default /opa]
     --rootapi=xxxx      HTML Root API URL [default /api]
     --alias=xxxx        Multiple url map outside of rootdir [eg: --alias=/icons:/usr/share/icons]
     --apitimeout=xxxx   Binding API timeout in seconds [default 20]
     --cntxtimeout=xxxx  Client Session Context Timeout [default 32000000]
     --cache-eol=xxxx    Client cache end of live [default 100000]
 -w, --workdir=xxxx      Set the working directory [default: $PWD or current working directory]
 -u, --uploaddir=xxxx    Directory for uploading files [default: workdir] relative to workdir
     --rootdir=xxxx      Root Directory of the application [default: workdir] relative to workdir
     --ldpaths=xxxx      Load bindings from dir1:dir2:... [default: path of afb-dbus-binding.so]
 -b, --binding=xxxx      Load the binding of path
     --weak-ldpaths=xxxx Same as --ldpaths but ignore errors
     --no-ldpaths        Discard default ldpaths loading
 -t, --token=xxxx        Initial Secret [default=random, use --token= to allow any token]
 -r, --random-token      Enforce a random token
 -V, --version           Display version and copyright
 -h, --help              Display this help
     --ws-client=xxxx    Bind to an afb service through websocket
     --ws-server=xxxx    Provide an afb service through websockets
 -A, --auto-api=xxxx     Automatic load of api of the given directory
     --session-max=xxxx  Max count of session simultaneously [default 200]
     --tracereq=xxxx     Log the requests: no, common, extra, all
     --traceevt=xxxx     Log the events: no, common, extra, all
     --traceses=xxxx     Log the sessions: no, all
     --traceapi=xxxx     Log the apis: no, common, api, event, all
     --traceglob=xxxx    Log the globals: none, all
     --traceditf=xxxx    Log the daemons: no, common, all
     --tracesvc=xxxx     Log the services: no, all
     --call=xxxx         call at start, format of val: API/VERB:json-args
     --no-httpd          Forbid HTTP service
 -e, --exec              Execute the remaining arguments
 -M, --monitoring        Enable HTTP monitoring at <ROOT>/monitoring/
 -C, --config=xxxx       Load options from the given config file
 -Z, --dump-config       Dump the config to stdout and exit
 -s, --set=xxxx          Set parameters ([API]/[KEY]:JSON or {"API":{"KEY":JSON}}
 -o, --output=xxxx       Redirect stdout and stderr to output file (when --daemon)
     --trap-faults=xxxx  Trap faults: on, off, yes, no, true, false, 1, 0 (default: true)
```

## help

Prints help with available options

## version

Display version and copyright

## verbose

Increases the verbosity, can be repeated

## color

Add basic colorization to the ouput.

## quiet

Decreases the verbosity, can be repeated

## log=xxxx

Tune the log level mask. The levels are:

 - error
 - warning
 - notice
 - info
 - debug

The level can be set using + or -.

| Examples | descritpion
|-----------------|-------------------
| error,warning   | selects only the levels error and warning
| +debug          | adds level debug to the current verbosity
| -warning        | remove the level warning from the current verbosity
| +warning-debug,info | Adds error and remove errors and warnings

## port=xxxx

HTTP listening TCP port  [default 1234]

## workdir=xxxx

Directory where the daemon must run [default: $PWD if defined
or the current working directory]

## uploaddir=xxxx

Directory where uploaded files are temporarily stored [default: workdir]

## rootdir=xxxx

Root directory of the application to serve [default: workdir]

## roothttp=xxxx

Directory of HTTP served files. If not set, files are not served
but apis are still accessible.

## rootbase=xxxx

Angular Base Root URL [default /opa]

This is used for any application of kind OPA (one page application).
When set, any missing document whose url has the form /opa/zzz
is translated to /opa/#!zzz

## rootapi=xxxx

HTML Root API URL [default /api]

The bindings are available within that url.

## alias=xxxx

Maps a path located anywhere in the file system to the
a subdirectory. The syntax for mapping a PATH to the
subdirectory NAME is: --alias=/NAME:PATH.

Example: --alias=/icons:/usr/share/icons maps the
content of /usr/share/icons within the subpath /icons.

This option can be repeated.

## apitimeout=xxxx

binding API timeout in seconds [default 20]

Defines how many seconds maximum a method is allowed to run.
0 means no limit.

## cntxtimeout=xxxx

Client Session Timeout in seconds [default 32000000 that is 1 year]

## cache-eol=xxxx

Client cache end of live [default 100000 that is 27,7 hours]

## session-max=xxxx

Maximum count of simultaneous sessions [default 200]

## ldpaths=xxxx

Load bindings from given paths separated by colons
as for dir1:dir2:binding1.so:... [default = $libdir/afb]

You can mix path to directories and to bindings.
The sub-directories of the given directories are searched
recursively.

The bindings are the files terminated by '.so' (the extension
so denotes shared object) that contain the public entry symbol.

## weak-ldpaths=xxxx

Same as --ldpaths but instead of stopping on error, ignore errors and continue.

## binding=xxxx

Load the binding of given path.

## token=xxxx

Initial Secret token to authenticate.

If not set, no client can authenticate.

If set to the empty string, then any initial token is accepted.

## random-token

Generate a random starting token. See option --exec.

## ws-client=xxxx

Transparent binding to a binder afb-daemon service through a WebSocket.

The value of xxxx is either a unix naming socket, of the form "unix:path/api",
or an internet socket, of the form "host:port/api".

## ws-server=xxxx

Provides a binder afb-daemon service through WebSocket.

The value of xxxx is either a unix naming socket, of the form "unix:path/api",
or an internet socket, of the form "host:port/api".

## foreground

Get all in foreground mode (default)

## daemon

Get all in background mode

## no-httpd

Forbids HTTP serve

## exec

Must be the last option for afb-daemon. The remaining
arguments define a command that afb-daemon will launch.
The sequences @p, @t and @@ of the arguments are replaced
with the port, the token and @.

## tracereq=xxxx

Trace the processing of requests in the log file.

Valid values are 'no' (default), 'common', 'extra' or 'all'.

## traceapi=xxxx

Trace the accesses to functions of class api.

Valid values are 'no' (default), 'common', 'api', 'event' or 'all'.

## traceevt=xxxx

Trace the accesses to functions of class event.

Valid values are 'no' (default), 'common', 'extra' or 'all'.

## call=xxx

Call a binding at start (can be be repeated).
The values are given in the form API/VERB:json-args.

Example: --call 'monitor/set:{"verbosity":{"api":"debug"}}'

## monitoring

Enable HTTP monitoring at <ROOT>/monitoring/

## name=xxxx

Set the visible name

## auto-api=xxxx

Automatic activation of api of the given directory when the api is missing.

## config=xxxx

Load options from the given config file

This can be used instead of arguments on the command line.

Example:

	afb-daemon \
		--no-ldpaths \
		--binding /home/15646/bindings/binding45.so \
		--binding /home/15646/bindings/binding3.so \
		--tracereq common \
		--port 5555 \
		--token SPYER \
		--set api45/key:54027a5e3c6cb2ca5ddb97679ce32f185b067b0a557d16a8333758910bc25a72 \
		--exec /home/15646/bin/test654 @p @t

is equivalent to:

	afb-daemon --config /home/15646/config1

when the file **/home/15646/config1** is:

	{
	  "no-ldpaths": true,
	  "binding": [
	    "\/home\/15646\/bindings\/binding45.so",
	    "\/home\/15646\/bindings\/binding3.so"
	  ],
	  "tracereq": "common",
	  "port": 5555,
	  "token": "SPYER",
	  "set" : {
	    "api45": {
	      "key": "54027a5e3c6cb2ca5ddb97679ce32f185b067b0a557d16a8333758910bc25a72"
	    }
	  },
	  "exec": [
	    "\/home\/15646\/bin\/test654",
	    "@p",
	    "@t"
	  ]
	}

The options are the keys of the config object.

See option --dump-config

## dump-config

Output a JSON representation of the configuration resulting from
environment and options.

## output=xxxx

Redirect stdout and stderr to output file

## set=xxxx

Set values that can be retrieved by bindings.

The set value can have different formats.

The most generic format is **{"API1":{"KEY1":VALUE,"KEY2":VALUE2,...},"API2":...}**

This example set 2 keys for the api *chook*:

	afb-daemon -Z --set '{"chook":{"account":"urn:chook:b2ca5ddb97679","delay":500}}'
	{
	   "set": {
	     "chook": {
	       "account": "urn:chook:b2ca5ddb97679",
	       "delay": 500
	     }
	   }
	 }

An other format is: **[API]/[KEY]:VALUE**.
When API is omitted, it take the value " * ".
When KEY is ommitted, it take the value of " * ".

The settings for the API \* are globals and apply to all bindings.

The settings for the KEY \* are mixing the value for the API.

The following examples are all setting the same values:

	afb-daemon --set '{"chook":{"account":"urn:chook:b2ca5ddb97679","delay":500}}'
	afb-daemon --set 'chook/*:{"account":"urn:chook:b2ca5ddb97679","delay":500}'
	afb-daemon --set 'chook/:{"account":"urn:chook:b2ca5ddb97679","delay":500}'
	afb-daemon --set 'chook/account:"urn:chook:b2ca5ddb97679"' --set chook/delay:500