---
title: Contribution Checklist
---

**Open Source Code Contribution Checklist**

## General
- [ ] Does the component have a name ? (Pick one fitting the purpose and the project.)
- [ ] Is a separate git repo required for the component ?
- [ ] Does the component have a README.md containing all basic information about it ?
    - [ ] Description.
    - [ ] Dependencies.
    - [ ] Build instructions.
    - [ ] Installation instructions.
    - [ ] Usage instructions.
    - [ ] Example invocations.
- [ ] Does the component have a CONTRIBUTIONS.md file? (Containing all necessary information on how to contribute, e.g. pointing to project website? )

## License
 - [ ] Is the license an OSI approved open source software license?
 - [ ] Are all files under an OSI approved open source license?
 - [ ] Does the component have a LICENSE (or COPYING) file detailing the license of the code?
 - [ ] Do the source code files have the license mentioned in the header?
 - [ ] Do the source code files have an SPDX tag? (Note: An SPDX tag can be used in a file header instead of the license note)
 - [ ] Are there files with other licenses in their header?
     - [ ] If so, LICENSE should be the for the majority of the files and LICENSE.xyz for the exceptions.

## docs/
 - [ ] Are there docs/ folder for the component ?
     - [ ] e.g. Are all APIs described inclusive description, usage and example invocations ?
     - [ ] e.g. Are all cmdline tools or options described in the documentation ?
     - [ ] e.g. Is the program flow described ?
 - [ ] Contain Changelog.md ? (Keep track of major changes in the changelog.)

## tests/
 - [ ] Must have tests available.
 - [ ] Must have simple invocation scripts available.
 - [ ] Must have instructions for CI available.
 - [ ] Must contribute CI test definitions.

## Git repository
 - [ ] Must have: a .gitreview file.
     - [ ] Option: Can have a .gitignore file.
     - [ ] Option: Can have a .editorconfig file.
 - [ ] All code needs to build against master.
 - [ ] Is a backport to a release branch required ?
 - [ ] Code contributions submitted need to have a Sign-off-by! (Follow [**DCO**](https://developercertificate.org/).)

## Yocto/OE
 - [ ] Recipes need to follow the guidelines of : [**new-recipe-writing-a-new-recipe**](https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#new-recipe-writing-a-new-recipe).
 - [ ] Recipes follow the [**bitbake style guide**](https://www.openembedded.org/wiki/Styleguide).
 - [ ] Your 'meta-*' layer needs to pass the yocto-check-layer tool.

## Gerrit Reviews
**All gerrit reviews need to be addressed. All issues are to be discussed with the experts.**

 - [ ] Issues are to be discussed in the EG first.
 - [ ] Consent needs to be reached.
 - [ ] Gerrit commits need two upvotes (not from authors!) to be merged.
 - [ ] Uploads should be 'ready for review' or marked 'WIP'.