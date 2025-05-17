# Access Control System

A web-based access control system that uses a Shelly device flashed with Tasmota to control a magnetic lock. The system provides a PIN code interface for locking and unlocking doors, with the web interface embedded directly in the Tasmota firmware.

## Features

- Custom web interface embedded in Tasmota firmware
- PIN code authentication system with master code for administration
- Real-time lock status monitoring
- Mobile-responsive design
- Secure access control
- Support for multiple PIN codes
- Automatic re-locking capability

## Prerequisites

- Shelly device (compatible with Tasmota firmware)
- Magnetic lock
- Development environment for Tasmota firmware modification
- Basic knowledge of C++ and web development

## Installation

1. Clone the Tasmota repository:
   ```sh
   git clone https://github.com/arendst/Tasmota.git
   ```

2. Modify the Tasmota firmware:
   - Locate the web server handler in `tasmota/xdrv_01_webserver.ino`
   - Add the custom endpoint handler for the lock control page
   - Implement PIN management logic
   - Add lock status monitoring

3. Compile and flash the modified firmware to your Shelly device

4. Configure the device:
   - Set up network settings
   - Configure the master code
   - Set up initial PIN codes

## Configuration

1. Access the device's web interface at `http://<device-ip>/lock`
2. Use the master code to manage PIN codes
3. Configure the relay settings for the magnetic lock
4. Set up any additional security measures

## Usage

1. Access the lock control page at `http://<device-ip>/lock`
2. Enter a valid PIN code to unlock the door
3. The system will automatically lock after a configured time period
4. Use the master code to manage PIN codes through the web interface

## Security Considerations

- Change the default master code before deployment
- Enable Tasmota's web authentication
- Use HTTPS for secure communication
- Regularly update firmware
- Monitor access logs
- Do not display PINs in production
- Keep your device's firmware up to date

## Limitations

- Web interface must be embedded in the firmware
- No support for uploading arbitrary HTML files
- Access limited to local network unless ports are opened (not recommended)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the GNU General Public License v3.0 (GPL-3.0) - see the LICENSE file for details.

## Support

For support, please open an issue in the GitHub repository or contact the maintainers.

## Acknowledgments

- Tasmota project for firmware support
- Shelly for hardware
- Contributors and users of the system 