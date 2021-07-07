---
title: PyAGL
---

# 0. Intro
## Main purpose 
PyAGL was written to be used as a testing framework replacing the Lua afb-test one,
however the modules are written in a way that could be used as standalone utilities to query
and evaluate apis and verbs from the App Framework Binder services in AGL.

## High level overview
Python compatibility: **Python 3.8 preferred**, **should** be compatible with **3.6**

Initial development PyAGL was done with Python **3.6** in mind, heavily using f-strings and a few typings. As of writing this 
documentation(June 3rd 2021), current stable AGL version is Koi 11.0.2 which has Python 3.8, and further development is 
done using 3.8 and 3.9 runtimes although **no** version-specific features are used from later versions; 
features **used** are kept within features offered by Python **3.8**.

The test suite is written in a relatively standard way of extending **pytest** with a couple tweaks 
tailored to Jenkins CI and LAVA for AGL with regards to output and timings/timeouts, and these tweaks 
are enabled by running `pytest -L` in order to enable LAVA logging behavior.

The way PyAGL works could be summarized in several bullets below:

* `websockets` package is used to communicate to the services, `x-afb-ws-json1` is used as a subprotocol, 
* base.py provides AGLBaseService to be extended for each service
* AGLBaseService has a `portfinder()` routine which will use `asyncssh` if used remotely, 
to figure out the port of the service's websocket that is listening on. When this was implemented services had a hardcoded listening port,
and was often changed when a new service was introduced. If you specify port, pyagl will connect to it directly. If no port is specified and
portfinder() cannot find the process or listening port should throw an exception and exit.
* main() implementations in most PyAGL services' bindings are intended to be used as a convenient standalone utility to query verbs, although
not necessarily available.
* PyAGL bindings are organized in classes, method names and respective parameters mostly adhere to service verbs/apis described 
per service in https://git.automotivelinux.org/apps/agl-service-*/about
For example, in https://git.automotivelinux.org/apps/agl-service-audiomixer/about/ the docs for the service describe 5 verbs -
subscribe, unsubscribe, list_controls, volume, mute - and their respective methods in audiomixer.py.
* as mentioned above `pytest` package is required for unit tests. 
* `pytest-async` is needed by pytest to cooperate with asyncio
* `pytest-dependency` is used in cases where specific testing order is needed and used via decorators

# 1. Using PyAGL
Command line execution is intended for debugging and quick evaluation.
There are few prerequisites to start using it. First, your AGL image **must** be bitbaked with **agl-devel** feature when sourcing __aglsetup.sh__;
if not - the running AGL instance won't have websocket services exposed to listening TCP ports and PyAGL will fail to connect.

```bash
git clone "https://gerrit.automotivelinux.org/gerrit/src/pyagl"
```
Preferably create a virtualenv and install the packages in the env
```bash
user@debian:~$ python3 -mvenv ~/.virtualenvs/aglenv
user@debian:~$ source ~/.virtualenvs/aglenv/bin/activate
(aglenv) user@debian:~$ pip install -r requirements.txt
```
Hard requirements are asyncssh, websockets, pytest, pytest-dependency, pytest-async; the others in the file are dependencies of the mentioned packages.

If you have installed PyAGL as python package or current working directory is in the project root:
```
(aglenv) user@debian:~/pyagl$ python3 -m pyagl.services.audiomixer 192.168.234.34 --list_controls
```
should produce the following or similar result depending on how many controls are exposed and which AGL version you are running:
```
matching services: ['afm-service-agl-service-audiomixer--0.1--main@1001.service']
Requesting list_controls with id 359450446
[RESPONSE][Status: success][359450446][Info: None][Data: [{'control': 'Master Playback', 'volume': 1.0, 'mute': 0}, 
{'control': 'Playback: Speech-Low', 'volume': 1.0, 'mute': 0}, {'control': 'Playback: Emergency', 'volume': 1.0, 'mute': 0}, 
{'control': 'Playback: Speech-High', 'volume': 1.0, 'mute': 0}, {'control': 'Playback: Navigation', 'volume': 1.0, 'mute': 0}, 
{'control': 'Playback: Multimedia', 'volume': 1.0, 'mute': 0}, {'control': 'Playback: Custom-Low', 'volume': 1.0, 'mute': 0}, 
{'control': 'Playback: Communication', 'volume': 1.0, 'mute': 0}, {'control': 'Playback: Custom-High', 'volume': 1.0, 'mute': 0}]]
```

# 2. Running the tests

## Markers
Before running the tests it must be noted that __Markers__ are metavariables applied to the tests which play important part 
in the behavior of pyagl and add great flexibility when deciding which tests to run.  
Each test suite usually have unambiguous marker and description in `pytest.ini` for the reason above.  
The default markers are applied in the list variable `pytestmark` in the beginning of each test file.  
Each test suite has at least 2 default markers.

`pytest.mark.asyncio` is a special marker needed because we are running pytest in async mode,
and the other marker is the name of the testsuite - for example `pytest.mark.geoclue` is for `geoclue` tests.

`pytest.mark.dependency` is another special marker used by the pytest-dependency library helping for 
running tests in a particular order when a more complicated setup other than a fixture is needed for 
a test or the setup itself is a test or sequence of tests.

`pytest.mark.hwrequired` is a marker signifying that additional hardware is required for tests to be successful - e.g. bluetooth phonebook tests need a phone with a pbap profile connected

`pytest.mark.internet` is a marker for tests which require internet connection

As mentioned above, behavior of pytest is altered by the markers because pytest does expression matching by marker and test names for which suite to be run.  
In the example below we select all internet requiring and subscription tests but skip those that require additional hardware.  
If we have a match with `internet or subscribe` the test will be selected, but if the test has `hwrequired` will be skipped, otherwise deselected overall.
```
h3ulcb:~# pyagl -vk "internet or subscribe and not hwrequired"
============================ test session starts ============================
platform linux -- Python 3.8.2, pytest-5.3.5, py-1.8.1, pluggy-0.13.1 -- /usr/bin/python3
cachedir: .pytest_cache
rootdir: /usr/lib/python3.8/site-packages/pyagl, inifile: pytest.ini
plugins: asyncio-0.10.0, dependency-0.5.1, reverse-1.0.1
collected 218 items / 163 deselected / 55 selected                          

test_bluetooth.py::test_subscribe_device_changes PASSED               [  1%]
...
test_radio.py::test_unsubscribe_all PASSED                            [ 90%]
test_signal_composer.py::test_subscribe SKIPPED                       [ 92%]
test_signal_composer.py::test_unsubscribe SKIPPED                     [ 94%]
test_weather.py::test_current_weather FAILED                          [ 96%]
test_weather.py::test_subscribe_weather PASSED                        [ 98%]
test_weather.py::test_unsubscribe_weather PASSED                      [100%]
==== 1 failed, 37 passed, 17 skipped, 163 deselected, 1 warning in 3.23s ====

```

## Locally - On the board itself
There is the /usr/bin/pyagl convenience wrapper script which invokes pytest  
with the tests residing in `/usr/lib/python3.8/site-packages/pyagl/tests` which  
also will pass all commandline arguments down to pytest

```console
qemux86-64:~# pyagl
=================== test session starts =============================
platform linux -- Python 3.8.2, pytest-5.3.5, py-1.8.1, pluggy-0.13.1
rootdir: /usr/lib/python3.8/site-packages/pyagl, inifile: pytest.ini
plugins: dependency-0.5.1, asyncio-0.10.0, reverse-1.0.1
collected 213 items

test_audiomixer.py .......                                    [  3%]
test_bluetooth.py ............xxxsxx                          [ 11%]
test_bluetooth_map.py .x.xs.                                  [ 12%]
...
```

## Remotely
You must export `AGL_TGT_IP` environment variable first, containing a string with a reachable IP address 
configured(either DHCP or static) on one of the interfeces on the AGL instance(board or vm) on your network.
`AGL_TGT_PORT` is not required, however can be exported to skip over connecting to the board via ssh first 
in order to figure out the listening port of service. 

```console
user@debian:~$ source ~/.virtualenvs/aglenv/bin/activate
(pyagl) user@debian:~$ export AGL_TGT_IP=192.168.234.34
(pyagl) user@debian:~$ cd pydev/pyagl/pyagl/tests
(pyagl) user@debian:~/pydev/pyagl/pyagl/tests$ pytest test_geoclue.py
========================= test session starts =========================
platform linux -- Python 3.9.2, pytest-5.4.1, py-1.8.1, pluggy-0.13.1
rootdir: /home/user/pydev/pyagl/pyagl, inifile: pytest.ini
plugins: dependency-0.5.1, asyncio-0.11.0
collected 3 items

test_geoclue.py ...                                              [100%]
```
# 3. Writing bindings and/or tests for new services
This assumes you have already went through ["3. Developer Guides > Creating a new service"](../3_Developer_Guides/2_Creating_a_New_Service.md).

Templates directory contains barebone _cookiecutter_ templates to create your project.
If you do not intend to use cookiecutter, you need a simple service file in which you inherit AGLBaseService
from base.py. 
You can take a look at pyagl/services/geoclue.py and pyagl/tests/test_geoclue.py which is probably the
simplest binding in PyAGL for a reference and example. All basic methods like 
[send|receive|un/subscribe|portfinder] are implemented in the base class. 
You would need to do minimal work to create new service binding from scratch and by example of the geoclue service you need to do the following:  

* do the basic imports 
```python3
from pyagl.services.base import AGLBaseService, AFBResponse
import asyncio
```

* inherit AGLBaseService and type in the service class member the service name presuming you are following the AGL naming convention:
(if your new service does not follow the convention, the portfider routine wont work and you'll have to specify service port manually)
```python3
class GeoClueService(AGLBaseService):
    service = 'agl-service-geoclue'
```

* if you intend to run the new service binding as a standalone utility, you might want to add your new options to the argparser
```python3
    parser = AGLBaseService.getparser()
    parser.add_argument('--location', help='Get current location', action='store_true')
```

* override the __init__ method with the respective parameters as api(used in the binding) and systemd service slug
```
    def __init__(self, ip, port=None, api='geoclue'):
        super().__init__(ip=ip, port=port, api=api, service='agl-service-geoclue')
```

* define your methods and send requests with .request() which prepares the data in a JSON format, request returns message id
```
    async def location(self):
        return await self.request('location')
```

* get the raw response with data = await .response() or use .afbresponse() to get structured data

PyAGL is written with asyncio so you'd need an event loop to get your code running.  
Most modules have standalone CLI functionality, which is done via ArgumentParser and each module that  
is CLI usable inherits static base parser and extends its arguments as shown in the example below:

```python3
async def main(loop):
    args = GeoClueService.parser.parse_args()
    gcs = await GeoClueService(args.ipaddr)

    if args.location:
        msgid = await gcs.location()
        print(f'Sent location request with messageid {msgid}')
        print(await gcs.afbresponse())

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
```
`GeoClueService` will try to return an instance with connection to the service. It will throw an exception if it fails to do so.  
`AGLBaseService.afbresponse()` method is a wrapper on top of .response() which will prepare, validate and format data for a convenient usage intended for CLI usage.  
`AFBResponse.__str__`  is also a convenience method to be able to print all relevant data in regarding state of the response.  

```console
(aglenv) user@debian:~$ python3 -m pyagl.services.geoclue 192.168.234.251 --location
matching services: ['afm-service-agl-service-geoclue--1.0--main.service']
Sent location request with messageid 29188435
[RESPONSE][Status: success][29188435][Info: GeoClue location data][Data: {'latitude': 42.6898421, 'longitude': 23.3099069, 'accuracy': 107.4702045}]
```


# 4. Environment variables
The following environment variables are used in the PyAGL project:

Variable | Description
--- | ---
**AGL_TGT_IP** | **required**
**AGL_TGT_PORT** | **optional**, when this is used portfinder() routine is skipped, attempts direct websocket connection to this port.
**AGL_TEST_TIMEOUT** | **optional**, over-ride the default 5 second timeout value for binding responses. This is useful in many cases where a long-standing request has taken place - e.g. importing few thousand entries from a phonebook
**AGL_AVAILABLE_INTERFACES** | **optional**, specify which of ethernet, wifi, and bluetooth interfaces are available.  The value is a comma separated list, with a default value of "ethernet,wifi,bluetooth".
**AGL_BTMAP_RECIPIENT** | **optional**, when running Bluetooth MAP tests, this would be used as phone number to write text messages to.
**AGL_BTMAP_TEXT** | **optional**, when running Bluetooth MAP tests, messages will be composed with this text.
**AGL_CAN_INTERFACE** | **optional**, specify the CAN interface to use for CAN testing, default value is "can0".
**AGL_PBAP_PHONENUM** | **optional**, when running Bluetooth PBAP tests, this phone number will be used to .search().
**AGL_PBAP_VCF** | **optional**, for the Bluetooh PBAP tests query a contact entry out of the phonebook.
**AGL_BT_TEST_ADDR** | **optional**, for the Bluetooth tests pair/connect/disconnect with an actual Bluetooth device(phone)'s address .  The address should have the DBus address style as "dev_00_01_02_03_04_05" instead of a regular colon-separated MAC address.

# 5. Debugging
PyAGL uses pythons logger library so you can rise the logging level and see the output from most of the modules, 
but was disabled by default because of the massive amount of output produced. When the default logger is enabled 
all other libraries will show their output with pyagl - e.g. websockets, asyncssh etc.

README.md in the root directory of the project also contains useful information.

