# Embexia Air File Transfer

[![Try now](https://img.shields.io/badge/Try%20now-air--transfer.embexia.com-19a88b?style=for-the-badge&logo=googlechrome&logoColor=white)](https://air-transfer.embexia.com)
[![Static HTML](https://img.shields.io/badge/Static-HTML5-E34F26?style=flat&logo=html5&logoColor=white)](#running-locally)
[![Browser-based processing](https://img.shields.io/badge/Processing-in%20the%20browser-3670A0?style=flat&logo=firefoxbrowser&logoColor=white)](#privacy)
[![Experimental](https://img.shields.io/badge/Status-experimental-orange?style=flat)](#compatibility-and-practical-notes)

**Embexia Air File Transfer** is an experimental browser-based prototype that turns files into sequences of colored visual symbols. It combines optional Re-Pair compression, a custom four-color transfer format (Q4R1), and RaptorQ forward error correction. A second device can capture the displayed sequence with its camera and reconstruct the file.

> **Experimental:** Q4R1 is a custom visual format, not a standard QR code. Transfer reliability and speed depend on the display, camera, lighting, focus, and browser. Do not rely on it as the only copy or transfer method for important data.

## Try It

Open the hosted application at **[air-transfer.embexia.com](https://air-transfer.embexia.com)**. Use a current browser. Camera reception requires HTTPS and permission to use the camera.

## Features

- Select files of any format, or load a repetitive sample to try the workflow.
- Optionally compress data with a greedy Re-Pair grammar builder and export the result as `.rpair`.
- Verify the compressed stream by decompressing it and comparing every byte with the input.
- Send either the compressed `.rpair` stream or the original bytes.
- Render a sequence of custom four-color Q4R1 symbols, navigate or play the frames, and save individual frames as PNG.
- Configure symbol density (81, 121, 161, or 201 cells per side), playback rate (1-60 frames per second), and RaptorQ overhead (0-100%).
- Receive symbols continuously through a camera or decode saved PNG frames; duplicate frames are ignored.
- Reconstruct from RaptorQ symbols, check CRC-32 values, and download the received file.
- Inspect transfer statistics, rendering benchmarks, and optional optical-capture diagnostics.

## How to Transfer

1. Open the app on the sending device and choose a file.
2. Choose **Compress with Re-Pair** if you want to try compression. Compression is optional; the compressed output can be larger than the input.
3. Choose **Send compressed (.rpair)** or **Send original (uncompressed)** to generate the visual sequence.
4. On the receiving device, open the same app, choose the same matrix density as the sender, and select **Start camera**. Alternatively, save PNG frames on the sender and decode them on the receiver.
5. Display the sequence on the sending device and point the receiving camera at it. Keep the whole symbol visible, upright, in focus, and well lit. The default playback rate is 3 frames per second.
6. Once enough valid RaptorQ symbols have been captured and the stream passes verification, download the reconstructed file.

For camera transfers, both devices need to run a compatible version of the app. The receiver expects Q4R1 protocol version 4 and the same matrix density as the transmitter. Reset the receiver before switching to a different transfer session.

## Running Locally

The app is a self-contained static page; there is no build step or package installation. Clone or download this repository and serve its root directory using any static web server. For example, with Python installed:

```bash
python -m http.server 8000
```

Then open [http://localhost:8000](http://localhost:8000). A local server is recommended over opening the file directly, and camera access is available in secure contexts such as `localhost` and HTTPS.

The RaptorQ WebAssembly module is loaded on demand from [`@raptorqr/raptorq-wasm`](https://www.npmjs.com/package/@raptorqr/raptorq-wasm) through `esm.sh`. An Internet connection is required the first time a transfer is generated or camera reception is started. The dependency is not bundled with this repository.

## Privacy

File selection, compression, rendering, camera decoding, and reconstruction are performed in the browser. **File contents are not uploaded to an Embexia application server by this page.** The browser does request the RaptorQ module from `esm.sh` when needed, so using the app is not fully offline and that module request reveals ordinary connection metadata to the module host. Camera video is processed in the browser and is not sent to an application server.

## Formats and Protocol

### RPR1 Compressed Files

`.rpair` is the app's custom binary container for Re-Pair output. It starts with the `RPR1` signature and stores the original length, grammar rules, and final symbol sequence using variable-length integers. Rules are assigned symbols beginning at 256. The app validates rule references, decompresses the result, and verifies the round trip before marking compression complete.

Re-Pair repeatedly selects a frequent adjacent symbol pair and replaces non-overlapping occurrences. This is a simple greedy strategy: it is not guaranteed to find a globally minimal grammar or make every file smaller. The configured maximum steps can limit processing time; stopping early keeps the smallest serialized representation generated so far.

### Q4R1 Visual Transfer

Q4R1 maps two bits to each of four colors and adds monochrome corner markers for camera detection. Each frame contains a session identifier, sequence information, payload length, CRC-32 checksums, source metadata, and RaptorQ data. RaptorQ can recover the source stream from sufficient valid symbols, so the receiver does not necessarily need every displayed frame.

Q4R1 is a project-specific format, not an ordinary QR code, and cannot be read by typical QR-scanner applications. The current page uses protocol version 4; it is not backward-compatible with the noted Q4R1 v2 frames or the previous 49 x 49 grid.

## Compatibility and Practical Notes

- Use a modern browser with JavaScript modules, WebAssembly, and the Canvas API.
- Camera input requires browser camera permission and a secure context (HTTPS or `localhost`).
- An Internet connection is needed to fetch the RaptorQ WebAssembly module unless the implementation is changed to host it locally.
- Start with 81 x 81 or 121 x 121 for camera reception. Higher densities carry more data per frame but make cells smaller and scanning more sensitive to image quality.
- Set the same matrix density on the transmitter and receiver. Larger grids need to occupy more pixels in the camera image.
- A higher RaptorQ overhead adds redundancy and may improve tolerance to missed frames, at the cost of a longer sequence.
- Actual playback and capture rates can be lower than the selected FPS due to device and browser performance.
- Large files can require substantial memory and processing time; practical limits vary by device and browser.

## Repository Layout

```text
.
|-- index.html   # Application markup, styles, and browser-side implementation
|-- LICENSE      # Apache-2.0 license for original project code
`-- README.md    # Project documentation
```

## Deployment on GitHub Pages

This repository needs no build command. To publish it with GitHub Pages:

1. Push the repository to GitHub.
2. In the repository, open **Settings > Pages**.
3. Under **Build and deployment**, select **Deploy from a branch**.
4. Choose the branch and `/ (root)` folder, then save.
5. Use the HTTPS Pages URL to open the application and enable camera access.

The published root must contain `index.html`. If you deploy under a path prefix, ensure the hosting setup serves that file at the selected Pages URL.

## Contributing

Bug reports, reproducible transfer failures, and improvements are welcome. For a useful issue report, include the browser and device models, selected matrix density and FPS, whether the input was compressed, and the relevant receiver diagnostics. Do not attach private files; a small synthetic sample or reproduction steps are preferable.

## License

Original code in this repository is licensed under the Apache License 2.0. See [LICENSE](LICENSE). This license does not cover third-party software or grant rights to third-party patents or trademarks.

### Third-Party RaptorQ WASM

The app loads `@raptorqr/raptorq-wasm` from `esm.sh` at runtime; the WASM binary is not included in this repository. Version 0.1.1, reviewed for this project, declares MIT in its npm metadata and includes an MIT license file. Its README says its WASM artifacts are generated from `cberner/raptorq`; that upstream crate declares Apache-2.0, and the package manifest identifies crate version 2.0.1. The reviewed npm archive did not include a separate copy of the upstream Apache-2.0 license. The package's MIT declaration should therefore not be treated as proof that all terms for the embedded upstream code are satisfied. Check the exact dependency version and its notices before bundling or redistributing the WASM. This page currently imports `@latest`, so the resolved package can change over time.

RFC 6330 requires a Simplified BSD notice for code components extracted from the RFC text. This project uses the separately published WASM package; whether that requirement applies to its generated binary has not been established here. Qualcomm has also filed [IPR disclosures 1958](https://datatracker.ietf.org/ipr/1958/) and [2554](https://datatracker.ietf.org/ipr/2554/) related to RFC 6330. Open-source copyright licenses do not by themselves establish patent clearance.