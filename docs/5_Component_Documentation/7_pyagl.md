---
title: PyAGL
---

# 0. Intro
## Main purpose 
PyAGL was written to be used as a testing framework replacing the Lua afb-test one,
however the modules are written in a way that could be used as standalone utilities to query
and evaluate apis and verbs from the App Framework Binder services in AGL.

## High level overview
Python compatibility:
Initial development PyAGL was done with Python **3.6** in mind, heavily using f-strings and a few typings. As of writing this 
documentation(June 3rd 2021), current stable AGL version is Koi 11.0.2 which has Python 3.8, and further development is 
done using 3.8 and 3.9 runtimes although **no** version-specific features are used from later versions; 
features **used** are kept within features **offered** by Python version first used for PyAGL in AGL Jellyfish.

The test suite is written in a relatively standard way of extending **pytest** with a couple tweaks 
tailored to Jenkins CI and LAVA for AGL with regards to output and timings/timeouts, and these tweaks are enabled by running `pytest -L`
in order to enable LAVA logging behavior.

The way PyAGL works could be summarized in several bullets below:
  * `websockets` package is used to communicate to the services, `x-afb-ws-json1` is used as a subprotocol, 
  * base.py provides AGLBaseService to be extended for each service
  * AGLBaseService has a portfinder() routine which will use `asyncssh` if used remotely, 
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
There are few prerequisites to start using it. First, your AGL image **must** be bitbaked with **agl-devel** feature when sourcing aglsetup.sh;
if not - the running AGL instance won't have websocket services exposed to listening TCP ports and PyAGL will fail to connect.

```bash
git clone "https://gerrit.automotivelinux.org/gerrit/src/pyagl"
```
Preferably create a virtualenv and install the packages in the env
```bash
pip install -r requirements.txt
```
Hard requirements are asyncssh, websockets, pytest, pytest-dependency, pytest-async; the others in the file are dependencies of the mentioned packages.
```
cd pyagl/pyagl/services
python3 audiomixer.py 192.168.234.34 --list_controls
```
or if you have installed PyAGL as python package 
```
python3 -m pyagl.services.audiomixer 192.168.234.34 --list_controls
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

## Locally - On the board itself
There is the /usr/bin/pyagl script which invokes the tests residing in 
`/usr/lib/python3.8/site-packages/pyagl/tests`

```
qemux86-64:~# pyagl
=================== test session starts =============================
platform linux -- Python 3.8.2, pytest-5.3.5, py-1.8.1, pluggy-0.13.1
rootdir: /usr/lib/python3.8/site-packages/pyagl, inifile: pytest.ini
plugins: dependency-0.5.1, asyncio-0.10.0, reverse-1.0.1
collected 213 items

test_audiomixer.py .......                                                                                                                                                                                                                                                                                            [  3%]
test_bluetooth.py ............xxxsxx                                                                                                                                                                                                                                                                                  [ 11%]
test_bluetooth_map.py .x.xs. 
...
```

## Remotely
You must export `AGL_TGT_IP` environment variable first, containing a string with a reachable IP address 
configured(either DHCP or static) on one of the interfeces on the AGL instance(board or vm) on your network.
`AGL_TGT_PORT` is not required, however can be exported to skip over connecting to the board via ssh first 
in order to figure out the listening port of service. 

```
user@debian:~$ source ~/.virtualenvs/pyagl/bin/activate
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
Templates directory contains barebone _cookiecutter_ templates to create your project.
If you do not intend to use cookiecutter, you need a simple service file in which you inherit AGLBaseService
from base.py. 
You can take a look at pyagl/services/geoclue.py and pyagl/tests/test_geoclue.py which is probably the
simplest binding in PyAGL for a reference and example. All basic methods like 
send|receive|un/subscribe|portfinder are implemented in the base class. 
You would need to do minimal work to create new service binding from scratch and by example of the geoclue you need to do the following:
- do the basic imports 
```
from pyagl.services.base import AGLBaseService, AFBResponse
import asyncio
import os
```
- inherit AGLBaseService and type in the service class member the service name presuming you are following the AGL naming convention:
(if your new service does not follow the convention, the portfider routine wont work and you'll have to specify service port manually)
```
class GeoClueService(AGLBaseService):
    service = 'agl-service-geoclue'
```
- if you intend to run the new service binding as a standalone utility, you might want to add your new options to the argparser
```
    parser = AGLBaseService.getparser()
    parser.add_argument('--location', help='Get current location', action='store_true')
```
- override the __init__ method with the respective parameters as api(used in the binding) and systemd service slug
```
    def __init__(self, ip, port=None, api='geoclue'):
        super().__init__(ip=ip, port=port, api=api, service='agl-service-geoclue')
```
- define your methods and send requests with .request() which prepares the data in a JSON format, request returns message id
```
    async def location(self):
        return await self.request('location')
```
- get the raw response with data = await .response() or use .afbresponse() to get structured data

README.md in the root directory of the project also contains useful information.

