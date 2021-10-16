# Network UPS Tools server

Docker image for Network UPS Tools server.

## Usage

This image provides a complete UPS monitoring service (USB driver only).

Start the container (docker-compose):

```console
  nut:
    image: orenhecht/nut-upsd:latest
    container_name: nut
    restart: unless-stopped
    devices:
      - /dev/bus/usb:/dev/bus/usb
    ports:
    - 3493:3493
    environment:
      - UPS_SERIAL=000000000
      - UPS_VENDOR_ID=ffff
      - UPS_PORT=/dev/ups0
      - UPS_API_PASSWORD=API_PASS
      - UPS_ADMIN_PASSWORD=ADMIN_PASS
```

## Auto configuration via environment variables

This image supports customization via environment variables.

### Mandatory Variables

#### UPS_SERIAL

Serial number of the UPS

#### UPS_VENDOR_ID

Vendor ID of the UPS

### Optional Variables

#### UPS_NAME

*Default value*: `ups`

The name of the UPS.

#### UPS_DESC

*Default value*: `Eaton 5SC`

This allows you to set a brief description that upsd will provide to clients that ask for a list of connected equipment.

#### UPS_DRIVER

*Default value*: `usbhid-ups`

This specifies which program will be monitoring this UPS.

#### UPS_PORT

*Default value*: `auto`

This is the serial port where the UPS is connected.

#### UPS_POLL_INTERVAL

*Default value*: `30`

This is the serial port where the UPS is connected.

#### API_PASSWORD

*Default value*: Randomized (see container logs for the generated value)

This is the password for the monitor user.

#### ADMIN_PASSWORD

*Default value*: Randomized (see container logs for the generated value)

This is the password for the admin user.

#### SHUTDOWN_CMD

*Default value*: `echo 'System shutdown not configured!'`

This is the command upsmon will run when the system needs to be brought down. The command will be run from inside the container.

#### ENABLE_WATCHDOG

*Default value*: true

Enables the watchdog that kills the container when connection to the UPS is lost. Together with the `restart: unless-stopped`
policy the container will restart upon losing connection.

When the UPS is disconnected the driver isn't able to reconnect, so this is a poor man's solution for it.

## USB Udev Rules

If the UPS gets disconnected it can change its usb mapping. To prevent this a udev rule can be added to create a symlink
to `/dev/ups0`:

1. Create the file `/etc/udev/rules.d/99-usb-serial.rules`
2. Add the rule: `ACTION=="add|bind", SUBSYSTEM=="usb", ATTRS{idVendor}=="ffff", ATTRS{idProduct}=="ffff", MODE="0666", SYMLINK+="ups0"`
(Update idVendor and idProduct from lsusb -v or lsusb -D /dev/<DEVICE> (device can be learned from dmesg)).
3. Reload rules: `sudo udevadm control --reload-rules`
