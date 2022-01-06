# Ambient Light Sensor Daemon For Linux
It's a user mode daemon for changing brightness based on light sensor value, designed for Asus Zenbooks, but usable for other vendors after some tuning.

## How to test befor install
Run `sudo watch cat /sys/bus/acpi/devices/ACPI0008\:00/iio\:device0/in_illuminance_raw` from a terminal and check that the number changes (try covering the sensor or exposing it to more light). If the number remains the same, it means the sensor driver doesn't work. If you see a file not found error, try to find the correct path to `in_illuminance_raw` in subfolders of `/sys/bus/acpi/devices/`.

For the ZBook 15 G6, and possibly others, the sensor path is `/sys/bus/iio/devices/iio:device0/in_illuminance_raw`. So try `sudo watch cat /sys/bus/iio/devices/iio:device0/in_illuminance_raw` and change the config if it works for you.

## Supported laptops

Works on ASUS Zenbooks with the built-in driver acpi-als:
* UX303UB
* UX305LA
* UX305FA
* UX310UQ
* UX330UA

On the Dell Inspiron 13 7353, the driver path and brightness levels need to be changed.

sometimes work (base on feedback):
* UX303UA
* UX305CA with [als driver](https://github.com/danieleds/als)
* UX430UQ Ubuntu with build in driver acpi-als, an extra ACPI call to enable the sensor `(_SB.PCI1.LPCB.EC0.ALSC)`
* UX410UQ

doesn't work on the following Zenbooks because of driver issues:
* UX303LN
* UX305UA
* UX31A
* UX32LN

An issue with Arch Linux may be related to the syslog. A pull request would be appreciated.

Please fill in [this feedback form](https://drive.google.com/open?id=1mjr_R3nXBFAeObI7zB7BPD_EpSvTTpOf_H67x-HE2qo). It may help other users.

The keyboard backlight is not adjusted, because my laptop doesn't have one. Want to help? Create an issue.

## Install package (experimental)

I finally found time to create a [deb package](https://drive.google.com/file/d/1bGBXRmiMMWeg6JIsV2SuZQ2RPYvezSbE/view)

To install and start, run the following commands after downloading:
```
sudo dpkg -i ~/Download/illuminanced_1.0-0.deb
sudo systemctl enable illuminanced.service
sudo systemctl start illuminanced.service
```

You can check the status by running `systemctl status illuminanced.service`.


Please open an issue if it doesn't work.

## How to build & install
* install rust: `curl https://sh.rustup.rs -sSf | sh`
* clone : `git clone https://github.com/mikhail-m1/illuminanced.git`
* build: `cd illuminanced; cargo build --release`
* install `sudo ./install.sh`

## How to adjust
* open the config file `/usr/local/etc/illuminanced.toml` (Default)
* choose how many light values do you need with `[general].light_steps`
* set the defined points count with `[light].points_count`
* set each point with `illuminance_<n>` and `light_<n> where` illuminance from `in_illuminance_raw` (see below) and light in range `[0..light_steps)`

## How it works
Illuminance is read from `/sys/bus/acpi/devices/ACPI0008:00/iio:device0/in_illuminance_raw`, a Kalman-like filter is applied, and the backlight value is set based on defined points.
Unfortunately I cannot find a way to get events from [iio buffers](https://www.kernel.org/doc/htmldocs/iio/iiobuffer.html) for the acpi-als driver, so for now it polls.

## `<Fn> + A`
switches between three modes:
- auto-adjust
- disabled
- max brightness (useful for movies, can be disabled in the config file `/usr/local/etc/illuminanced.toml`)

## Contributing
Any feedback is welcome!
