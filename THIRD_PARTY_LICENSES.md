# Third-Party Licenses

This software uses the following third-party libraries:

## Qt6

- **License:** GNU Lesser General Public License v3 (LGPL v3)
- **Website:** https://www.qt.io
- **Full License Text:** https://www.gnu.org/licenses/lgpl-3.0.txt

### LGPL v3 Compliance

This application dynamically links against Qt6. In accordance with the LGPL v3:

1. **Source Code Availability:** Qt6 source code is available at https://code.qt.io
2. **Relinking:** Users may relink this application against a modified version of Qt6
3. **License Notice:** The Qt libraries are provided under LGPL v3

To relink against a different Qt version:
1. Obtain or build the desired Qt6 libraries
2. Set `LD_LIBRARY_PATH` to point to your Qt6 installation
3. Run the application

For the complete LGPL v3 license text, see: https://www.gnu.org/licenses/lgpl-3.0.txt
