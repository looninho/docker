# Parrot Anafi 4K simulation in Docker container

stuck with `firmwared.service` failed to start

## build
```
docker build -t parrot-anafi_4k:simulation .
```

## run
### init systemd 
```
xhost +local:docker
```

```
docker run -it --rm --privileged --gpus all -u root \
-e DISPLAY=$DISPLAY \
-e QT_X11_NO_MITSHM=1 \
-e NVIDIA_VISIBLE_DEVICES=all \
-e NVIDIA_DRIVER_CAPABILITIES=all \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-v $(pwd):/home/docker_user/anafi4k \
-w /home/docker_user/anafi4k \
--name anafi_4k parrot-anafi_4k:simulation
```

in the init process I get an assert error and an failed error:
```
...
[  OK  ] Reached target Local File Systems.
apparmor.service: Starting requested but asserts failed.
[ASSERT] Assertion failed for Load AppArmor profiles.
         Starting Set Up Additional Binary Formats...
proc-sys-fs-binfmt_misc.automount: Got automount request for /proc/sys/fs/binfmt_misc, triggered by 34 (systemd-binfmt)
         Mounting Arbitrary Executable File Formats File System...
[  OK  ] Mounted Arbitrary Executable File Formats File System.
...
[  OK  ] Stopped Parrot products firmware loading daemon.
[FAILED] Failed to start Parrot products firmware loading daemon.
See 'systemctl status firmwared.service' for details.
[  OK  ] Started User Login Management.
```

### run Sphinx
open a new terminal and connect to the container as `docker_user`:
```
docker exec -it -u docker_user anafi_4k bash
```

then try the proposed command gives me:
```
systemctl status firmwared.service
× firmwared.service - Parrot products firmware loading daemon
     Loaded: loaded (/lib/systemd/system/firmwared.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Wed 2024-03-13 22:52:47 UTC; 19s ago
    Process: 83 ExecStart=/bin/firmwared (code=exited, status=1/FAILURE)
   Main PID: 83 (code=exited, status=1/FAILURE)
        CPU: 8ms
```

start the service as sudo:
```
sudo systemctl start firmwared.service
```
launch the simulation cannot connect to `firmwared`:
```
sphinx "/opt/parrot-sphinx/usr/share/sphinx/drones/anafi_ai.drone"::firmware="https://firmware.parrot.com/Versions/anafi2/pc/%23latest/images/anafi2-pc.ext2.zip"
Parrot-Sphinx simulator version 2.15.1

Connection with firmwared timed out

```
