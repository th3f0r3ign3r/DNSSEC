# DNSSEC
Implementation of DNS Security Extension

This document is the final transcription of the efforts made in the realization of my thesis in audit, security of systems and computer networks on the theme "Implementation of a public key infrastructure: case of DNSSEC".

## OS
For the realisation of this topic we use 
* Ubuntu 16.04 LTS, [Download Here](https://releases.ubuntu.com/16.04/)

## Tools
In this topic we use :
```bash
sudo apt install bind9 bind9-utils haveged
```

## Architecture
For the implementation here, we use 4 Servers like :
* Root Server "."
* TLD Server (.com)
* Recursif Server
* Reference Server (example.com)
And finally we also use a client machine.

The Architecture of our implementation look like this

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Architecture")
