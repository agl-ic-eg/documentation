---
title: Using CMake Templates from BitBake Recipes
---

If you have developed an application and you want to include it in an AGL image,
you must add a BitBake recipe in one of the following layers:

* [meta-agl](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/AGL/meta-agl):
 meta-agl layer (core AGL)
* [meta-agl-cluster-demo](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/AGL/meta-agl-cluster-demo):
 cluster demo specific recipes and configuration
* [meta-agl-demo](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/AGL/meta-agl-demo):
 meta-agl-demo layer (demo/staging/"one-shot")
* [meta-agl-devel](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/AGL/meta-agl-devel):
 meta-agl-devel (Development and Community BSPs)
* [meta-agl-extra](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/AGL/meta-agl-extra):
 meta-agl-extra (additional/optional components for AGL)

Once you have the recipe in place, edit it to include the following
line to cause the `aglwgt` class to be inherited:

```bb
inherit aglwgt
```

Following is an example that uses the HVAC application recipe (i.e. `hvac.bb`), which
builds the HVAC application:

```bb
SUMMARY     = "HVAC Service Binding"
DESCRIPTION = "AGL HVAC Service Binding"
HOMEPAGE    = "https://gerrit.automotivelinux.org/gerrit/#/admin/projects/apps/agl-service-hvac"
SECTION     = "apps"

LICENSE     = "Apache-2.0"
LIC_FILES_CHKSUM = "file://LICENSE;md5=ae6497158920d9524cf208c09cc4c984"

SRC_URI = "gitsm://gerrit.automotivelinux.org/gerrit/apps/agl-service-hvac;protocol=https;branch=${AGL_BRANCH}"
SRCREV  = "${AGL_APP_REVISION}"

PV = "1.0+git${SRCPV}"
S  = "${WORKDIR}/git"

DEPENDS = "json-c"
RDEPENDS_${PN} += "agl-service-identity-agent"

inherit cmake aglwgt pkgconfig
```

The following links provide more examples of recipes that use the
CMake templates:

* [agl-service-helloworld](https://gerrit.automotivelinux.org/gerrit/admin/repos/apps/agl-service-helloworld)
* [agl-service-audio-4a](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/apps/agl-service-audio-4a)
* [agl-service-unicens](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/apps/agl-service-unicens)
* [4a-hal-unicens](https://gerrit.automotivelinux.org/gerrit/#/admin/projects/src/4a-hal-unicens)