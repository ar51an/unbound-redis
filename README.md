##  unbound - block ads and trackers  

<div align="justify">

### Summary
**Unbound** as validating, recursive, caching DNS resolver 🔹 **Redis** backend 🔹 Block **Ads and Trackers**

🔸 `Compile` latest Unbound on RaspPiOS with `Cache DB` and `TCP Fast Open` modules.  
🔸 `Recursive` resolving from the root. **No** forwarding to other resolvers.  
🔸 Redis backend database for `persistent` cache. Works as second level cache.  
🔸 Network wide `Ads and Trackers` block. **No** pi-hole/adguard. **No** extra hop to resolve DNS.  

#### Prerequisite:
* Unbound compilation and installation is validated on `RaspiOS/Debian`. `Post Install` startup service and scripts are reused from RaspiOS bullseye, they may require modification for other linux distributions.
* If unbound package is installed. Take a backup of current `unbound.conf`. Remove unbound package completely:  

  > `sudo apt --purge autoremove unbound`

#### Specs:
> |Unbound |OS                           |HW                      |
> |:-------|:----------------------------|:-----------------------|
> |`1.17.0`|`raspios-bullseye-arm64-lite`|`Raspberry Pi 4 Model B`|

#
### Steps
&nbsp;&nbsp;🔸 Redis ➜ Unbound ➜ Post Install ➜ Config ➜ Timers & Services ➜ Blocklist ➜ Start   
#### ❯ Redis
&nbsp;&nbsp;🔸 Install ➜ Config
* **Install:**  
  There are 2 options **either** install redis (6.0.16) from RaspiOS bullseye **or** install redis (7.0.5) from RaspiOS bullseye backports.
  * Install redis **(6.0.16)** from raspios bullseye:
    > `sudo apt install redis-server`  

  * Install redis **(7.0.5)** from raspios bullseye backports:
    > Enable backports. Edit sources list:  
    > `sudo nano /etc/apt/sources.list`  
    > Add backports source at the end:  
    > `deb http://deb.debian.org/debian bullseye-backports main`  

    > Install redis:  
    > `sudo apt install redis-server/bullseye-backports`  

* **Config:**  
  An optimized `redis.conf` for unbound is available in the release under `config` dir. Default _redis.conf_ from redis **7.0.5** is used as base config for the provided config. Some of the options may not be available or may be different if you are on an earlier version of redis. You can use _redis.conf_ **either** from the release **or** your preferred one.  

  If you installed redis **7.0.5** and going to use the provided _redis.conf_, below steps can be helpful:
  > Edit redis config:  
  > `sudo nano /etc/redis/redis.conf`  
  > Delete everything in default redis config:  
  > `Ctrl+6` `Alt+t` `Ctrl+6`  
  > Copy and paste the provided `redis.conf`. Save and exit nano  

  > `ℹ️` **Note:**  
  > Provided `redis.conf` is tweaked after some thorough testing in small network. Like 8mb maxmeory has pretty optimal performance with enough cache and evict least recently used keys. Similarly sanpshotting is used to save keys to database, current option will save after 2hrs if atleast 100 new keys were added or after 12hrs if atleast 1 new key is added. Reboot will save database as long as snapshotting is enabled. Feel free to change them as preferred.

* **Startup Warning:**  
  If you installed redis **7.0.5** from `backports`. Modify services to fix journal `⚠️` warning on redis startup.  
  > Edit: `sudo nano /usr/lib/systemd/system/redis-server.service`  
  > Edit: `sudo nano /usr/lib/systemd/system/redis-server@.service`  
  > Remove/Comment lines starting with `NoExecPaths` and `ExecPaths` from both above services  
  > Restart redis: `sudo systemctl restart redis-server`  

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### ❯ Unbound
&nbsp;&nbsp;🔸 Packages ➜ Extract ➜ CFLAGS ➜ Configure ➜ Compile ➜ Install
* **Packages:**  
  Install packages required for compiling unbound. Assuming gcc is already installed, below command will install 11 packages. Your environment may require additional packages. Check compilation error to find missing package (if any):
  > ```
  > sudo apt install bison flex libevent-dev libexpat1-dev libhiredis-dev libnghttp2-dev libprotobuf-c-dev protobuf-c-compiler python3-dev swig libssl-dev
  > ```

* **Extract:**  
  Download and extract unbound.  
  > Extract:  
  > `tar -xvzf unbound-release-1.17.0.tar.gz`

* **CFLAGS:**  
  Remove debugging information, otherwise unbound binary size will be much larger.  
  > Set CFLAG:  
  > `export CFLAGS="-O2"`

  > `ℹ️` **Note:**  
  > Unbound `1.17.0` binary size comparison:  
  > ![bookworm](https://user-images.githubusercontent.com/11185794/207215543-bf41ded3-0a9f-44e1-9f90-eb68600a8441.png) &nbsp;➟ _Debian Bookworm Prebuilt_ `Without Cachdb Module`  
  > ![debug-off](https://user-images.githubusercontent.com/11185794/207215583-244aa012-8f24-4848-a39d-8803ec771e0f.png) &nbsp;➟ _Compiled Without Debug Info_ `With Cachdb Module`  
  > ![debug-on](https://user-images.githubusercontent.com/11185794/207215554-6cc8d9be-4f07-47bc-ab0c-d7359ff68ee7.png) &nbsp;➟ _Compiled With Debug Info_ `With Cachdb Module`

* **Configure:**  
  Make sure you copy the full cmd and execute it inside the extracted unbound src dir.
  > ```
  > ./configure --build=aarch64-linux-gnu --prefix=/usr --includedir=\${prefix}/include --infodir=\${prefix}/share/info --libdir=\${prefix}/lib/aarch64-linux-gnu --mandir=\${prefix}/share/man --localstatedir=/var --runstatedir=/run --sysconfdir=/etc --with-chroot-dir= --with-dnstap-socket-path=/run/dnstap.sock --with-libevent --with-libhiredis --with-libnghttp2 --with-pidfile=/run/unbound.pid --with-pythonmodule --with-pyunbound --with-rootkey-file=/var/lib/unbound/root.key --disable-dependency-tracking --disable-flto --disable-maintainer-mode --disable-option-checking --disable-rpath --disable-silent-rules --enable-cachedb --enable-dnstap --enable-subnet --enable-systemd --enable-tfo-client --enable-tfo-server
  > ```

* **Compile:**  
  > `make`

* **Install:**  
  > `sudo make install`

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### ❯ Post Install
&nbsp;&nbsp;🔸 Add User ➜ Create Dirs ➜ Copy Files ➜ Create Crypto Keys
* Run script `post-install.sh` available in the release under `post-install` dir. It automates post install tasks.
  > Run: `sudo ./post-install.sh`  

  > `ℹ️` **Note:**  
  > Startup service and scripts are reused from unbound package in RaspiOS bullseye. `root.hints` is downloaded from `internic`, it will be automated through systemd timer.  
* Alternatively, create user manually and use your preferred startup service and scripts.

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### ❯ Config
&nbsp;&nbsp;🔸 Update unbound and sysctl configuration files
* A tweaked `unbound.conf` is available in the release under `config` dir. You can use _unbound.conf_ either from the release or your preferred one. If you use the provided _unbound.conf_, modify below attributes as per your environment:
  > Change interface to Raspberry Pi IP:  
  > `interface: <IP>`

  > Change access-control to allowed local subnet:  
  > `access-control: <subnet> allow`

  > Change private-address to your allowed local subnet:  
  > `private-address: <subnet>`

  > `cache-min-ttl` is set to 0. Zero keeps minimum ttl of cached record as the domain owner intended. After some thorough testing in small network with different values, zero gives optimal performance with `prefetch-key` and `serve-expired` enabled.

  > `serve-expired-reply-ttl` is set to 0. Default is 30. Unbound changed the default value from 0 to 30 a while back following the `RFC 8767` recommendation. Default nonzero value throws unbound startup `⚠️` warning when cachedb module is enabled in _unbound.conf_ under `module-config`.  
  >> `ℹ️` **Note:**  
  >> Unbound documentation and RFC suggested to use 30 for _serve-expired-reply-ttl_ if `serve-expired-client-timeout` (default is 0) is used. Unbound uses default 30 for _serve-expired-reply-ttl_ irrespective of _serve-expired-client-timeout_.  
  >> Unbound behavior with default _serve-expired-reply-ttl_:  
  >> &nbsp;• Record served from _cache_ ➟ `serve-expired-reply-ttl=30`  
  >> &nbsp;• Record served from _cachedb_ ➟ `serve-expired-reply-ttl=0`  
  >> Provided `unbound.conf` sets the _serve-expired-reply-ttl_ to 0. TTL of 0 indicates that the expired record is only meant for this DNS resolution and not to be cached by the client. It removes unbound startup warning, keeps cache and cachedb behavior consistent and _serve-expired-client-timeout_ is not used.

  > `num-threads` and various `cache-slabs` are optimized for Raspberry Pi 4 CPU.

* Modify `/etc/sysctl.conf`. Enable tcp fast open for unbound and change virtual memory commit mode for redis warning.
  > Edit: `sudo nano /etc/sysctl.conf`  

  > Add below lines at the end:
  > ```
  > ###################################################################
  > # Unbound & Redis
  > #
  > # Unbound: Enable TCP Fast Open - Reduces Network Latency
  > net.ipv4.tcp_fastopen=3
  > #
  > # Redis: Recommended To Use 1 - Removes Redis Log Warning
  > # Kernel Virtual Memory Overcommit Mode
  > # 0 Heuristic overcommit (Default)
  > # 1 Always overcommit, never check
  > # 2 Always check, never overcommit
  > vm.overcommit_memory=1
  > ```

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### ❯ Timers & Services
&nbsp;&nbsp;🔸 Automate blocklist and roothints update with systemd timers
* Run script `install-timers.sh` available in the release under `install-timers` dir. Installs systemd timers, services and scripts for `root servers` and `blocklist` update.
  > Run: `sudo ./install-timers.sh`  

  > `ℹ️` **Note:**  
  > Timers update:  
  > &nbsp;• `root servers` ➟ Monthly on the last Sunday at 3:55am  
  > &nbsp;• `blocklist` ➟ Monthly on the last Sunday at 4:00am  
  > You can change the time and frequency in `/etc/systemd/system/unbound-roothints.timer` and `/etc/systemd/system/unbound-blocklist.timer`
  > 

* Alternatively, use your preferred method for blocklist and roothints update.

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### ❯ Blocklist
* Create dir and empty blocklist:
  > `sudo mkdir -p /opt/unbound/blocklists`  
  > `sudo touch /opt/unbound/blocklists/unbound.block.conf`

  > `ℹ️` **Note:**  
  > `/opt/unbound/scripts/update-blocklists.sh` script uses StevenBlack's `unified hosts (adware + malware) + porn` as default list. It converts default list to unbound format, removes comments and sorts it.  
  > You can add more lists to the _update-blocklists.sh_ script. With some basic expertise in sed you can aggregate multiple lists into unbound blocklist `unbound.block.conf`  

<div align="center">
  <img src="https://user-images.githubusercontent.com/11185794/205388020-99c057ad-ee9d-440b-8df9-587f5c133f2e.png?raw=true" alt="divider"/>
</div>

#### ❯ Start
&nbsp;&nbsp;🔸 Initial run
* Let's do the initial run. Start `unbound`, manually update `root servers` and `blocklist`.
  > Enable unbound:  
  > `sudo systemctl enable unbound`  

  > Start unbound:  
  > `sudo systemctl start unbound`  

  > Update root servers:  
  > `sudo /opt/unbound/scripts/update-roothints.sh`  

  > Update blocklist:  
  > `sudo /opt/unbound/scripts/update-blocklists.sh`  

#
#### ❯ Validate
&nbsp;&nbsp;🔸 Verfiy unbound and redis
* Unbound resolving the DNS:
  > Run: `dig whoami.akamai.net`  
  > Returns your WAN IP.

  > Run: `dig +ttlunits github.com`  
  > Returns github IP in the _ANSWER SECTION_ and raspberry pi IP where unbound is running in the _SERVER_.

* Redis saving the cache:
  > Run: `redis-cli dbsize`  
  > Count starts increasing with each unbound DNS lookup request.

#
#### ❯ Cmds
&nbsp;&nbsp;🔸 Few handy cmds
* **Unbound:**  
  > Stats:  
  > `sudo unbound-control stats_noreset | grep total`  

  > Tail log:  
  > `sudo journalctl -u unbound -n 200 -f`  

  > Tail filtered log:  
  > `sudo journalctl -u unbound -n 200 -f | grep "10.1.5.30\|10.1.5.32"`  

  > Check config for errors:  
  > `unbound-checkconf /etc/unbound/unbound.conf`  

* **Redis CLI:**  
  > Db size:  
  > `redis-cli dbsize`  

  > Memory stats:  
  > `redis-cli info memory | grep human`  

  > Monitor live queries:  
  > `redis-cli monitor`

#
#### ❯ `ℹ️` Tips & Notes
* **Resolver Configuration:**  
  Make sure `/etc/resolv.conf` has only one name server.
  > `nameserver <RaspberryPi-IP>` **or** `nameserver 127.0.0.1`

* **Add LAN DNS:**  
  According to your router, change LAN DNS to Raspberry Pi IP. DNS setting under internet setup is WAN DNS, it is not same as LAN DNS. If router permits to change LAN DNS, it is usually under LAN setup.

* **Troubleshoot Blocked Domain:**  
  Below configuration logs only blocked domains, using that you can find domain causing the issue.
  > Edit: `sudo nano unbound.conf`  
  > Set: `verbosity: 1` and `log-local-actions: yes`  

* **Block Selective:**  
  Specific domains can be blocked for specific IPs with tag options. It works on top of existing ads and trackers block. Provided `unbound.conf` has selective block configuration commented out under `|Block|`. If interested uncomment it and replace the IPs and domains.
  > `ℹ️` **Note:**  
  > Provided specific block configuration handles simple use case pretty well. For complex scenarios utilize unbound RPZ.

* **DNS Lookup:**  
  To inspect if the record is served from redis cachedb.  
  > Increase log verbosity briefly:  
  > Run: `sudo unbound-control verbosity 4`

  > Tail log in separate terminal:  
  > Run: `sudo journalctl -u unbound -n 200 -f`

  > Run: `dig github.com`

  > _ANSWER SECTION_ in the log shows that DNS lookup is responded from redis cache:  
  > `info: redis ;; ->>HEADER<<- ...`

  > Revert log verbosity:  
  > Run: `sudo service unbound reload`

* **Uninstall Unbound:**  
  Unbound can be uninstalled by running below cmd in the build directory.
  > `sudo make uninstall`

  After uninstall all the `Post Install` and `Timers & Services` steps can be easily reverted by running `post-remove.sh` provided in the release.
  > `sudo ./post-remove.sh`
</div>
