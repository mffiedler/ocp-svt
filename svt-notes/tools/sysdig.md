# sysdig
* [Sysdig documentation](https://github.com/draios/sysdig/wiki)

## Install
Not available in RHEL or EPEL repos.   Need to install it from the private sysdig yum repo (and need to trust it!).

```sh
# curl -s https://s3.amazonaws.com/download.draios.com/stable/install-sysdig | sudo bash
* Detecting operating system
* Installing sysdig public key
* Installing sysdig repository
* Installing kernel headers
* Installing sysdig

Creating symlink /var/lib/dkms/sysdig/0.24.1/source ->
                 /usr/src/sysdig-0.24.1

DKMS: add completed.

Kernel preparation unnecessary for this kernel.  Skipping...

Building module:
cleaning build area....
make -j4 KERNELRELEASE=3.10.0-954.el7.x86_64 -C /lib/modules/3.10.0-954.el7.x86_64/build M=/var/lib/dkms/sysdig/0.24.1/build...
cleaning build area...

DKMS: build completed.

sysdig-probe.ko.xz:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/3.10.0-954.el7.x86_64/extra/
Adding any weak-modules

depmod....
```

## Chisels
Add-on modules for particular data.
```sh
# sysdig -cl                                                                                                                                                                                                                                                      
                                                                                                                                                                                                                                                                                          
Category: Application                                                                                                                                                                                                                                                                     
---------------------                                                                                                                                                                                                                                                                     
httplog         HTTP requests log                                                                                                                                                                                                                                                         
httptop         Top HTTP requests                                                                                                                                                                                                                                                         
memcachelog     memcached requests log                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                                          
Category: CPU Usage                                                                                                                                                                                                                                                                       
-------------------                                                                                                                                                                                                                                                                       
spectrogram     Visualize OS latency in real time.                                                                                                                                                                                                                                        
subsecoffset    Visualize subsecond offset execution time.                                                                                                                                                                                                                                
topcontainers_cpu                                                                                                                                                                                                                                                                         
                Top containers by CPU usage                                                                                                                                                                                                                                               
topprocs_cpu    Top processes by CPU usage                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                                          
Category: Errors                                                                                                                                                                                                                                                                          
----------------                                                                                                                                                                                                                                                                          
topcontainers_error                                                                                                                                                                                                                                                                       
                Top containers by number of errors                                                                                                                                                                                                                                        
topfiles_errors Top files by number of errors                                                                                                                                                                                                                                             
topprocs_errors top processes by number of errors                                                                                                                                                                                                                                         
                                                                                                                                                                                                                                                                                          
Category: I/O                                                                                                                                                                                                                                                                             
-------------                                                                                                                                                                                                                                                                             
echo_fds        Print the data read and written by processes.                                                                                                                                                                                                                             
fdbytes_by      I/O bytes, aggregated by an arbitrary filter field                                                                                                                                                                                                                        
fdcount_by      FD count, aggregated by an arbitrary filter field                                                                                                                                                                                                                         
fdtime_by       FD time group by
iobytes         Sum of I/O bytes on any type of FD
iobytes_file    Sum of file I/O bytes
spy_file        Echo any read/write made by any process to all files. Optionall
                y, you can provide the name of one file to only intercept reads
                /writes to that file.
stderr          Print stderr of processes
stdin           Print stdin of processes
stdout          Print stdout of processes
topcontainers_file
                Top containers by R+W disk bytes
topfiles_bytes  Top files by R+W bytes
topfiles_time   Top files by time
topprocs_file   Top processes by R+W disk bytes

Category: Logs
--------------
spy_logs        Echo any write made by any process to a log file. Optionally, e
                xport the events around each log message to file.
spy_syslog      Print every message written to syslog. Optionally, export the e
                vents around each syslog message to file.

Category: Misc
--------------
around          Export to file the events around the time range where the given
                 filter matches.

Category: Net
-------------
iobytes_net     Show total network I/O bytes
spy_ip          Show the data exchanged with the given IP address
spy_port        Show the data exchanged using the given IP port number
topconns        Top network connections by total bytes
topcontainers_net
                Top containers by network I/O
topports_server Top TCP/UDP server ports by R+W bytes
topprocs_net    Top processes by network I/O

Category: Performance
---------------------
bottlenecks     Slowest system calls
fileslower      Trace slow file I/O
netlower        Trace slow network I/0
proc_exec_time  Show process execution time
scallslower     Trace slow syscalls
topscalls       Top system calls by number of calls
topscalls_time  Top system calls by time

Category: Security
------------------
list_login_shells
                List the login shell IDs
shellshock_detect
                print shellshock attacks
spy_users       Display interactive user activity

Category: System State
----------------------
lscontainers    List the running containers
lsof            List (and optionally filter) the open file descriptors.
netstat         List (and optionally filter) network connections.
ps              List (and optionally filter) the machine processes.

Category: Tracers
-----------------
tracers_2_statsd
                Export spans duration as statds metrics.

Use the -i flag to get detailed information about a specific chisel
```

## csysdig
top-like interface for sysdig beginners.  Provides access to popular chisels.