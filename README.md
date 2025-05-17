# Access Control System

A web-based access control system that uses a Shelly device with Tasmota firmware to control a magnetic lock. The system provides a simple PIN code interface for locking and unlocking doors.

## Features

- Web-based interface for controlling magnetic locks
- PIN code authentication system
- Real-time lock status monitoring
- Mobile-responsive design
- Secure access control
- Easy setup and configuration
- Custom webpage embedded in Tasmota firmware

## Prerequisites

- Shelly device (compatible with Tasmota firmware)
- Magnetic lock
- Basic knowledge of networking and web development

## Installation

1. Flash your Shelly device with Tasmota firmware
2. Configure your Shelly device's network settings
3. Upload the custom webpage to the device using Tasmota's web interface:
   - Go to Configuration -> Configure Module
   - Scroll down to "Custom Page"
   - Paste the HTML code for the access control interface
   - Save the configuration
4. Configure the access control settings

## Configuration

1. Set up your Shelly device's IP address in the configuration
2. Configure your PIN codes in the custom webpage
3. Set up any additional security measures
4. Configure the relay settings for the magnetic lock

## Usage

1. Access the device's IP address through your browser
2. The custom webpage will load automatically
3. Enter the PIN code to unlock the door
4. The system will automatically lock after a configured time period

## Security Considerations

- Change default PIN codes
- Use HTTPS for secure communication
- Regularly update firmware and software
- Monitor access logs
- Implement additional security measures as needed
- Keep your device's firmware up to date

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the GNU General Public License v3.0 (GPL-3.0) - see the LICENSE file for details.

## Support

For support, please open an issue in the GitHub repository or contact the maintainers.

## Acknowledgments

- Tasmota project for firmware support and custom webpage feature
- Shelly for hardware
- Contributors and users of the system 