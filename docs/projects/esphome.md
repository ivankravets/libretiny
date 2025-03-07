# ESPHome

!!! tip
	See the [Cloudcutter video guide](https://www.youtube.com/watch?v=sSj8f-HCHQ0) for a complete tutorial on flashing with [Cloudcutter](https://github.com/tuya-cloudcutter/tuya-cloudcutter) and installing LibreTiny-ESPHome. **Includes Home Assistant Add-On setup.**

Because ESPHome does not natively support running on non-ESP chips, you need to use a fork of the project.

There are three basic ways to install and use LibreTiny-ESPHome. You can choose the option that best suits you:

- ESPHome Dashboard (GUI) - for new users, might be an easy way to go; config management & compilation using web-based dashboard
- command line (CLI) - for more experienced users; compilation using CLI commands, somewhat easier to troubleshoot
- [Home Assistant Add-On](https://github.com/libretiny-eu/esphome-hass-addon/pkgs/container/libretiny-esphome-hassio) - using LibreTiny-ESPHome in Home Assistant, alongside your normal ESPHome installation - click the link for more info

!!! tip
	You can use LibreTiny-ESPHome for ESP32/ESP8266 compilation as well - the forked version *extends* the base, and doesn't remove any existing features. Keep in mind that you might not have latest ESPHome updates until the fork gets updated (which usually happens at most every few weeks).

## Find your device's board

Go to [Boards & CPU list](../status/supported.md), find your board (chip model), click on it and remember the **`Board code`**. This will be used later, during config creation.

If your board isn't listed, use one of the **Generic** boards, depending on the chip type of your device.

## Download ESPHome

=== "GUI"

	For this, you need Docker, Docker Compose and Python installed. After running the commands, you'll have a running ESPHome Dashboard interface that you can connect to.

	1. Open a terminal/cmd.exe.
	2. Create a `docker-compose.yml` file in a directory of choice:

		```yaml title="docker-compose.yml"
		version: "3"
		services:
		  esphome:
		    container_name: esphome-libretiny
		    image: docker pull ghcr.io/libretiny-eu/libretiny-esphome-docker:latest
		    volumes:
		      - ./configs:/config:rw # (1)!
		      - /etc/localtime:/etc/localtime:ro
		    restart: always
		    privileged: false
		    network_mode: host
		```

		1. You can change `./configs` to another path, in which your ESPHome configs will be stored.
	3. Start the container using `docker-compose up`. You should be able to open the GUI on [http://localhost:6052/](http://localhost:6052/).

=== "CLI"

	!!! important
		Read [Getting started](../getting-started/README.md) first - most importantly, the first part about installation.

		**It is very important that you have the latest version of LibreTiny installed** (not `libretiny-esphome` - this is a different thing!) **so that you don't face issues that are already resolved**.

	Assuming you have PlatformIO, git and Python installed:

	1. Open a terminal/cmd.exe, create `esphome` directory and `cd` into it.
	2. `git clone https://github.com/kuba2k2/libretiny-esphome`
	3. `cd` into the newly created `libretiny-esphome` directory.
	4. Check if it works by typing `python -m esphome`

	!!! tip
		For Linux users (or if `python -m esphome` doesn't work for you):

		- uninstall ESPHome first: `pip uninstall esphome`
		- install the forked version: `pip install -e .`

## Create your device config

=== "GUI"

	1. Open the GUI on [http://localhost:6052/](http://localhost:6052/) (or a different IP address if you're running on a Pi).
	2. Go through the wizard steps:
		- `New Device`
		- `Continue`
		- enter name and WiFi details
		- choose `LibreTiny`
		- choose the board that you found before
		- select `Skip`
	3. A new config file will be added. Press `Edit` and proceed to the next section.

=== "CLI"

	1. Create a YAML config file for your device. You can either:
		- use `python -m esphome wizard yourdevice.yml` - type answers to the six questions the wizard asks, OR:
		- write a config file manually:
			```yaml title="yourdevice.yml"
			esphome:
			  name: yourdevice

			bk72xx:  # adjust accordingly: bk72xx or rtl87xx
			  board: cb2s  # THIS IS YOUR BOARD CODE
			  framework:
			    version: latest

			logger:
			api:
			  password: ""
			ota:
			  password: ""

			wifi:
			  ssid: "YourWiFiSSID"
			  password: "SecretPa$$w0rd"
			  ap:
			    ssid: "Yourdevice Fallback Hotspot"
			    password: "Dv2hZMGZRUvy"
			```

## Automatically generate config

Instead of adding components manually and writing everything from scratch, you can use [UPK2ESPHome](https://upk.libretiny.eu/) to generate a working config (for supported BK7231 devices only). If your device has a Cloudcutter profile, there's a high chance it can have a generated config.

## Add components

Now, just like with standard ESPHome on ESP32/ESP8266, you need to add components for your device. Visit [ESPHome homepage](https://esphome.io/) to learn about YAML configuration. If you want, you can upload an "empty" config first, and add actual components later.

!!! danger "Important"
	It's highly recommended to **always** include the [`web_server`](https://esphome.io/components/web_server.html) and [`captive_portal`](https://esphome.io/components/captive_portal.html) components - **even in your first "empty" upload**.

	Adding these two components will safeguard you against accidentally soft-bricking the device, by e.g. entering invalid Wi-Fi credentials. The Web Server provides an easy way to flash a new image over-the-air, and the Captive Portal allows to easily open the Web Server on a fallback AP.

## Build & upload

=== "GUI"

	Close the config editor. Press the three dots icon and select `Install`. Choose `Manual download` and `Modern format`. The firmware will be compiled and a UF2 file will be downloaded automatically.

=== "CLI"

	The command `python -m esphome compile yourdevice.yml` will compile ESPHome.

Now, refer to the [flashing guide](../flashing/esphome.md) to learn how to upload ESPHome to your device. There's also info on using `tuya-cloudcutter` in that guide.

## Advanced: LT configuration

!!! note
	This part is for advanced users. You'll probably be fine with the default options.

All options from [Options & config](../dev/config.md) can be customized in the LibreTiny block:

```yaml title="yourdevice.yml"
bk72xx:
  framework:
    version: latest
  lt_config:
    LT_LOG_HEAP: 1
    LT_UART_DEFAULT_PORT: 2
    LT_UART_SILENT_ALL: 0
```
(this is only an example)

Additionally, few options have their dedicated keys:

```yaml title="yourdevice.yml"
bk72xx:
  framework:
    version: latest
  # verbose/trace/debug/info/warn/error/fatal
  loglevel: warn
  # suppress chip's SDK log messages
  # (same as LT_UART_SILENT_ALL above)
  sdk_silent: true
  # disable SWD/JTAG so that all GPIOs can be used
  # set to false if you want to attach a debugger
  gpio_recover: true
```
(these values are defaults)
