# Arkadia

Arkadia turns an arbitrary file into a square grayscale PNG and back again. Every byte of the input becomes one pixel intensity (0–255), the original file extension is written into the first pixels, and the decoder reverses the process.

It is a small experiment, not an archival tool. Read [Limitations](#limitations) before using it on anything you cannot recreate.

## How it works

**Encoding** (`encoder.py`)

1. The input file is read and hex-encoded (`modules/ftb.py`).
2. The file extension is hex-encoded and prepended, followed by a `00` separator (`modules/bti.py`).
3. The byte stream is cut to the largest perfect square that fits, reshaped into an N×N `uint8` matrix with NumPy and saved as a grayscale PNG with Pillow.

**Decoding** (`decoder.py`)

1. The PNG is opened, converted to 8-bit grayscale and flattened (`modules/itb.py`).
2. The first four bytes are read back as the extension (a trailing `00` is stripped).
3. The remaining bytes are written to `<name>_output.<extension>` (`modules/btf.py`).

## Requirements

- Python 3
- `numpy` (1.x, see Limitations)
- `Pillow`

```
pip install "numpy<2" pillow
```

There is no `requirements.txt`; install the two packages manually.

## Usage

```
# file -> image
python encoder.py --file notes.txt        # writes notes.png

# image -> file
python decoder.py --file notes.png        # writes notes_output.txt
```

`--file` is the only option. Both scripts derive the output name from the input name.

## Limitations

All of these follow from the current implementation:

- **Trailing bytes are dropped.** The pixel count is forced to a perfect square by discarding bytes from the end of the stream (`modules/bti.py`), so the decoded file is byte-identical only when the padded payload length happens to be a perfect square. Formats that tolerate a truncated tail, such as plain text, look fine; formats with trailers or checksums may not open. This is most likely why the project description excludes JPEG and WEBP.
- **Extensions longer than three characters are not handled.** The decoder always reads four bytes as the extension, so a four-letter extension leaks the `00` separator into the data and longer extensions are cut.
- **File names with more than one dot** (for example `archive.tar.gz`) confuse the extension detection, which splits on every dot.
- **NumPy 2.x breaks the decoder.** `modules/itb.py` calls `ndarray.tostring()`, which was removed in NumPy 2.0. Use NumPy 1.x or change the call to `tobytes()`.
- No tests and no error handling for missing or malformed images.

## Project structure

```
encoder.py        CLI: file -> PNG
decoder.py        CLI: PNG -> file
modules/ftb.py    file to hex bytes
modules/bti.py    hex bytes to square grayscale image
modules/itb.py    image to hex bytes (+ extension)
modules/btf.py    hex bytes to file
```

## License

No license file is included yet. Until one is added, the code is all rights reserved by default.
