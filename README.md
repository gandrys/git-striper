
# *git-stripper [Andrys Jiri, 2019.03.12, v0.2 ]*  

 Author        : Jiri Andrys  
 Maintainer    : Jiri Andrys  
 Contributors  : Jiri Andrys  
  
# 1. Contents:

* [2. Overview](#2-overview)
* [3. Getting Started](#3-getting-started) 
  * [3.1 Installation](#31-installation)
  * [3.2 Run](#32-run)
* [4. Dependency List](#4-dependency-list)
* [5. Examples ](#5-examples)
  * [5.1 In-house(access to private data), No Input Parameters ](#51-in-houseaccess-to-private-data-no-input-parameters)
  * [5.2 In-house(access to private data), Command Line Parameters ](#52-in-houseaccess-to-private-data-command-line-parameters)
  * [5.3 The 3rd party and stripped git in archive ](#53-the-3rd-party-and-stripped-git-in-archive)


# 2. Overview
The `git-stripper` strips all historical data from git with exception of one requested commit.

##  BSP can be divided to two parts :  
1. Publicly available, composed from OSS/GPL-ish data, available by repo init XXXX 
2. Private data, publicly unavailable, out-of-assembly(manifest.xml)

Primary purpose of this tool is sharing yocto-layers unavailable to 3rd party and in the same time keep assembly identification by SHA1/TAG across all involved party. The 3rd party and in-house company will have the same assembly and ids. Some layers simply can not be part of assembly(manifest.xml) due to speed of devel or internal workflows and together with bsp-probe we can identify assembly simpler way even when some layers are out-of-assembly.

Part of tool is verification if data(ignore changes in paths) in checkout directory are the same as in git objects(SHA1-SUM) -> stripping check. 

    - Tool asks for URL and TAG of git repository.  
    - Result is stripped git in tar.xz archive
    - Archive does not include checkout data, after copying layer to source user have to run **"git checkout -f"**
    - It supports copying yocto layers to BSP immediately after stripping, but only if stripper is located in  
      sources/<*bsp-release-layers>/tools/git-stripper.sh. Used in-house.
    - Only NXP/Freescale's based BSPs are supported.   
      NXP/Freescale has clear structure and divide   
      "meta-data"(in sources directory) from build itself(any directory "build").   


Yocto-tools has been tested on following systems:  

 * Ubuntu 16.04(64bit, docker image )   
 * Ubuntu 18.04(64bit, docker image )   
 * Fedora 24(64bit)   
 * Lubuntu 18.04(64bit, docker image )   


# 3. Getting Started 

Git stripper is usually part of BSP assembly located in **sources/meta-XXX-bsp-release/tools/** directory,   
so any in-house party can run it and distribute resulting ***.tar.xz** archive.

## 3.1 Installation

- **No installation is necessary.** 


## 3.2 Run

Tool asks for necessary information, no need to add any parameters, if necessary call commmand with  --help parameter.  

>`$ git-stripper`  

For more information.:
>`$ git-striper --help`  


# 4. Dependency List

1. Standard tools, included in almost all kind of distros and installed by default:  
   >` bash, mkdir, grep, egrep, awk, git, find, sort, pwd, printf, tar`  

   
# 5. Examples 

## 5.1. In-house(access to private data), No Input Parameters 
This can be case when we want to verify image and included functionality
inside company environment.  


### 5.1.1. BSP Setup: 
Here we just fetch and assembly **"publicly available"/in assembly** layers.
  
```bash
1. $ fslbsp$ repo init -u XXX.git -b <XXX> -m XXX.xml
2. $ fslbsp$ repo sync`  
3. $ fslbsp$ MACHINE=XXX DISTRO=XXX source fsl-setup-release.sh -b build   
4. $ fslbsp/build$ bitbake XXXX`
```   

### 5.1.2. git-stripper:   
Here we fetch **"publicly unavailable"/out-of-assembly** layers, copy it to BSP and create distribution archive for 3rd party.

```bash
5. $ fslbsp/build$ cd  ../sources/meta-XXX-bsp-release/tools     
6. $ fslbsp/sources/meta-XXX-bsp-release/tools$ ./git-stripper
 
    GIT-STRIPPER:
 
    DEPENDENCY-TEST:
    >OK

    Please enter repository's URL:
    >https://github.com/intel/luv-yocto

    Please enter TAG:
    >yocto-2.0.1

    GIT-CREATE:
    Initialised empty Git repository in /opt/XX/works/fslbsp/sources/meta-XXX-bsp-release/tools/SEND_TEMP/luv-yocto/.git/
    remote: Enumerating objects: 6365, done.
    .
    .
    .

    GIT-VERIFY:[luv-yocto]
    >COMMIT COUNT
    >SHA1-SUM
    >>SHA1-SUM->GIT-OBJECTS   : 42921609032
    >>SHA1-SUM->CHECKOUT-FILES: 42921609032
    >>Identical SHA1-SUM. Git does not include more data than given commit
    >>OK

    GIT-PACK:
    >CREATING ARCHIVE: -luv-yocto__yocto-2.0.1.tar.xz- 
    >>OK

    YOCTO-BSP FOUND:
        Do you want to copy luv-yocto yocto-layer to current BSP ? [y/n] 
        >y
    
       COPYING:
       >OK

       Please read README related to this external layer and set up BSP ! 

```   

Before we can build image we have to read notes related to out-of-assembly(luv-yocto) layer and setup yocto environment.   


### 5.1.3. Image Build:
Here we configure out-of-assembly's(luv-yocto) layers and rest of BSP and build final image.  
**We have to read notes related to out-of-assembly layer and configure BSP manually.**

```bash
7. $ fslbsp/sources/meta-XXX-bsp-release/tools$ cd ../../../build`
8. $ fslbsp/build$ bitbake XXX_IMAGE
```

### 5.1.4. bsp-probe:   
As last step, after image build, we have to generate bsp-probe.log in order to get information about image assembly.

```bash
9. $ fslbsp/build$ bsp-probe

  BSP-PROBE:
    DEPENDENCY-TEST:
    >OK
    LAYERS-GIT-INCLUDE-CHECK:
    >OK
    REPO-PROBE:
    >OK
    GIT-FILES-HASH:
      >/opt/xxx/works/fslbsp/sources/base
      >/opt/xxx/works/fslbsp/sources/luv-yocto
      >/opt/xxx/works/fslbsp/sources/meta-XXX-bsp-release
      >/opt/xxx/works/fslbsp/sources/meta-openembedded
      >/opt/xxx/works/fslbsp/sources/poky
    KERNEL-CONFIG:
    >OK
    LOCAL-CONFIG:
    >OK
```


## 5.2. In-house(access to private data), Command Line Parameters 

### 5.2.1. BSP Setup:   
Here we just fetch and assembly **"publicly available"/in assembly** layers.

```bash
1. $ fslbsp$ repo init -u XXX.git -b <XXX> -m XXX.xml
2. $ fslbsp$ repo sync`  
3. $ fslbsp$ MACHINE=XXX DISTRO=XXX source fsl-setup-release.sh -b build   
4. $ fslbsp/build$ bitbake XXXX`
```   

### 5.2.2. git-stripper:    
Here we fetch **"publicly unavailable"/out-of-assembly** layers, copy it to BSP and create distribution archive for 3rd party.

```bash
5. $ fslbsp/build$ cd  ../sources/meta-XXX-bsp-release/tools     
6. $ fslbsp/sources/meta-XXX-bsp-release/tools$ ./git-stripper -u="https://github.com/GENIVI/meta-ivi" -t="3.0.1"
 
    GIT-STRIPPER:
 
    DEPENDENCY-TEST:
    >OK

    GIT-CREATE:
    .
    .
    .

    GIT-VERIFY:[meta-ivi]
    >COMMIT COUNT
    >SHA1-SUM
    >>SHA1-SUM->GIT-OBJECTS   : 724021158
    >>SHA1-SUM->CHECKOUT-FILES: 724021158
    >>Identical SHA1-SUM. Git does not include more data than given commit
    >>OK

    GIT-PACK:
    >CREATING ARCHIVE: -meta-ivi__3.0.1.tar.xz- 
    >>OK

    YOCTO-BSP FOUND:
        Do you want to copy luv-yocto yocto-layer to current BSP ? [y/n] 
        >y
    
       COPYING:
       >OK

       Please read README related to this external layer and set up BSP ! 

```   

Before we can build image we have to read notes related to out-of-assembly(meta-ivi) layer and setup yocto environment.   


### 5.2.3. Image Build:   
Here we configure out-of-assembly's(meta-ivi) layers and rest of BSP and build final image.  
**We have to read notes related to out-of-assembly layer and configure BSP manually.** 

```bash
7. $ fslbsp/sources/meta-XXX-bsp-release/tools$ cd ../../../build`
8. $ fslbsp/build$ bitbake XXX_IMAGE
```



### 5.2.4. bsp-probe:   
As last step, after image build, we have to generate bsp-probe.log in order to get information about image assembly.

```bash
9. $ fslbsp/build$ bsp-probe

  BSP-PROBE:
    DEPENDENCY-TEST:
    >OK
    LAYERS-GIT-INCLUDE-CHECK:
    >OK
    REPO-PROBE:
    >OK
    GIT-FILES-HASH:
      >/opt/xxx/works/fslbsp/sources/base
      >/opt/xxx/works/fslbsp/sources/meta-ivi
      >/opt/xxx/works/fslbsp/sources/meta-XXX-bsp-release
      >/opt/xxx/works/fslbsp/sources/meta-openembedded
      >/opt/xxx/works/fslbsp/sources/poky
    KERNEL-CONFIG:
    >OK
    LOCAL-CONFIG:
    >OK
```



## 5.3 The 3rd party and stripped git in archive    

### 5.3.1 BSP Setup: 
Here we just fetch and assembly **"publicly available"/in assembly** layers.

```bash
1. $ fslbsp$ repo init -u XXX.git -b <XXX> -m XXX.xml
2. $ fslbsp$ repo sync`  
3. $ fslbsp$ MACHINE=XXX DISTRO=XXX source fsl-setup-release.sh -b build   
4. $ fslbsp/build$ bitbake XXXX`
```   

### 5.3.2 Out-of-assembly layer, Archive from git-stripper(meta-ivi__3.0.1.tar.xz):    
The 3rd party obtains archive from git-stripper and add layers to **"publicly available"/in assembly** part of BSP fetched in previous steps.
Here we extract **"publicly unavailable"/out-of-assembly** layers from git-stripper's archive and copy it to BSP.
Because of archive includes only .git, **git checkout -f** is necessary.  

```bash
5. $ fslbsp/build$ cd .. 
6. $ fslbsp $ ls -la 
    -rw-r--r--  1 alcz11702218 alcz11702218 154232 Mar 11 16:11 meta-ivi__3.0.1.tar.xz
    drwxr-xr-x  6 alcz11702218 alcz11702218   4096 Mar 11 15:48 build 
    drwxr-xr-x  3 alcz11702218 alcz11702218  36864 Feb 21 06:29 downloads
    drwxr-xr-x 19 alcz11702218 alcz11702218   4096 Mar 11 16:12 sources
    .
    .
    .
7. $ fslbsp $ tar meta-ivi__3.0.1.tar.xz -C sources/
8. $ fslbsp $ git -C sources/meta-ivi/ checkout -f
```



### 5.3.3. Image Build:
Here 3rd party configure out-of-assembly's(meta-ivi, meta-ivi__3.0.1.tar.xz) layers and rest of BSP and build final image.  
**The 3rd party have to read notes related to out-of-assembly layer and configure BSP manually.**   

```bash
9. $ fslbsp/build$ bitbake XXX_IMAGE
```



### 5.3.4. bsp-probe: 
**As last step, after final modifications in layers and image build,
3rd party have to generate bsp-probe.log in order to get information about image assembly and share it for purpose of reproducibility.**

```bash
10. $ fslbsp/build$ bsp-probe

  BSP-PROBE:
    DEPENDENCY-TEST:
    >OK
    LAYERS-GIT-INCLUDE-CHECK:
    >OK
    REPO-PROBE:
    >OK
    GIT-FILES-HASH:
      >/opt/xxx/works/fslbsp/sources/base
      >/opt/xxx/works/fslbsp/sources/meta-ivi
      >/opt/xxx/works/fslbsp/sources/meta-XXX-bsp-release
      >/opt/xxx/works/fslbsp/sources/meta-openembedded
      >/opt/xxx/works/fslbsp/sources/poky
    KERNEL-CONFIG:
    >OK
    LOCAL-CONFIG:
    >OK
```

