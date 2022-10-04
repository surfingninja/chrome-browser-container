# chrome-browser-container #

**Chrome browser in container**

**Environment**  
***Platform to use***: Linux, MacOS  


**Set up and run**  
- docker build -t chrome .
- chmod +x chrome.sh
- bash ./chrome.sh

***Dockerfile***

1. Libs `dbus-x11` and `libexif12` are being installed to avoid Chrome's
warnings, it could work without ones though.
2. User `docker` with unique UID and GID parameters is set to work properly within
the application, the parameters is replaced with `HOST_UID` и `HOST_GID`
once `run_chrome` has started, for now browser cache and config is mapped with your
file system with the same access rights as your OS user.
3. In order to connect to X11 server the `tmp.X11-unix` port is required and environment variable 'DISPLAY' is set.
Remote connections are not allowed by default, so additionally read permissions granted for `$HOME/.Xauthority`,
do not virtualize a network (with parameter `–net=host` on start). Alternatively you would like to execute
`xhost +local` from your host machine.
4. We also need `/dev/snd` for sound.
5. Chrome creates sandboxes for its child process by default to isolate them and increase security stability. 
(Kind of docker in docker). There are no permissions by default for it. To resolve it run container with
`-privileged` flag. More proper way is to run with flags `–cap-add=SYS_ADMIN` and `--security-opt seccomp:unconfined`.
It is not recommended to run chrome container with `–no-sandbox` flag.
6. To run chrome with different cursors, fonts and timezones volumes `/usr/share/icons`, `/usr/share/fonts` and
`/etc/localtime` are added.
7. Directory `/dev/shm` is mapped due to unexpected chrome fails, e.g. when open Google Maps, because of insufficient
space set by default
(a [solution](https://github.com/elgalu/docker-selenium/issues/20#issuecomment-133011186) provided)
8. Socket `/var/run/dbus/system_bus_socket` is not required, however chrome would like it
9. Option `–log-driver=journald` is required in order to record the logs into journalctl. 
By default, they are recorded into separate file without rotation, as a result the size of it may 
increase significantly if you re-create container rarely. Take a look on logs with: 
``sudo journalctl CONTAINER_NAME=chrome``
10. Script `chrome.sh` creates container once during the first run, and it takes some time to execute.
The next runs could be performed with `docker start chrome` when container already exists, it would operate faster.

