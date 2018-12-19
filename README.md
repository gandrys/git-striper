
# *git-striper [Andrys Jiri, 2018.12.18, v0.1 ]*  

 Author        : Jiri Andrys  
 Maintainer    : Jiri Andrys  
 Contributors  : Jiri Andrys  
  
# 1. Contents:

* [2. Overview](#2-overview)
* [3. Getting Started](#3-getting-started) 
  * [3.1 Installation](#31-installation)
  * [3.2 Run](#32-run)
* [4. Dependency List](#4-dependency-list)


# 2. Overview
The `git-striper` strips all history data from git with exception of one requested commit.   
Part of tool is verification if data(ignore changes in paths) in checkout directory are the same as in git objects(SHA1-SUM). 
Tool asks for URL and TAG of git repository.  Result is tar.xz archive .


Yocto-tools has been tested on following systems:  

 * Ubuntu 14.04(64bit)
 * Ubuntu 16.04(64bit with docker) 
 * Fedora 24(64bit)  


# 3. Getting Started 

## 3.1 Installation

- **No installation is necessary.** 


## 3.2 Run

Tool asks for necessary information, no need to add any parameters.
>`$ git-striper`  

For more information.:
>`$ git-striper --help`  


# 4. Dependency List

1. Standard tools, included in almost all kind of distros and installed by default:  
   >` bash, mkdir, grep, egrep, awk, git, find, sort, pwd, printf, tar`  
