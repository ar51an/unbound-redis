#### Release Notes:
* **1.2 Release:**  
  v1.2 release includes RaspberryPi OS Bookworm related scripts and service changes. If you are on Bullseye use v1.1 release.

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### Unbound Updates:
* **Unbound 1.18.0:**  
  Unbound added the option to connect to redis server over a unix socket. Unix sockets have better throughput. Check 
the `Tips & Notes` section in the README for enabling it.

* **Unbound 1.17.1:**  
  Unbound has a new option to keep cache intact between configuration reloads. It is integrated into blocklist & roothints update.

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### Upgrade:
To upgrade unbound from old-ver to latest-ver. Only below steps are required:  
* Unbound ➟ `Download, Extract, CFLAGS, Configure, Compile and Install`  
* Restart unbound

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>
