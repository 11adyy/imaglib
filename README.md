# imaglib

A C command-line image-processing library and utility for uncompressed 24-bit BMP files using the `BITMAPINFOHEADER` layout. It reads a bitmap, applies the requested filters in order, and writes the processed image to a new file. The implementation includes BMP input/output, pixel and matrix helpers, and reusable filters without relying on an external image framework.

## Supported input

The program is designed for 24-bit, uncompressed BMP images. Other bit depths, compression modes, or DIB header variants may not be supported. Use a compatible source image and retain the `.bmp` extension for output files.

## Build and run

Build the command-line program with the repository's C toolchain and run it with an input path, output path, and optional filter sequence:

```bash
make
./image_processor input.bmp output.bmp -crop 800 600 -gs -blur 0.5
```

The example loads `input.bmp`, keeps the upper-left 800 by 600 rectangle, converts it to grayscale, applies Gaussian blur with sigma 0.5, and writes the result to `output.bmp`. Running the program without enough arguments prints usage information. The filter list may be empty; in that case the input is copied to the output through the image reader and writer.

## Filter pipeline

Filters are applied from left to right in the order supplied on the command line. Each filter takes its parameters immediately after its name:

| Option | Parameters | Effect |
| --- | --- | --- |
| `-crop` | `width height` | Keeps the requested rectangle starting at the upper-left corner. |
| `-gs` | none | Converts RGB values to grayscale using `0.299R + 0.587G + 0.114B`. |
| `-neg` | none | Inverts each color component with `1 - value`. |
| `-sharp` | none | Applies a 3 by 3 sharpening kernel. |
| `-edge` | `threshold` | Applies an edge kernel to grayscale pixels and thresholds the result to black or white. |
| `-blur` | `sigma` | Applies Gaussian blur with the given sigma. |

Matrix operations use neighboring pixels and extend edge pixels when the kernel reaches outside the image. Crop dimensions larger than the available source area are limited to the pixels that exist.

## Project structure

- `main.c` parses the command line and connects the filters into a processing pipeline.
- `include/bitmap.h` describes the bitmap data and file interface.
- `src/bitmap.c` reads and writes the BMP representation.
- `include/filters.h` and `src/sfilters.c` contain simple per-pixel filters.
- `include/matrix.h` and `src/matrix.c` provide matrix operations used by spatial filters.
- `src/hfilters.c` contains convolution-based filters such as blur, sharpen, and edge detection.
- `test_script/` contains image-processing test support.

## Testing and limitations

The Python helper in `test_script/` can exercise the executable against prepared BMP data. Use an uncompressed 24-bit BMP with a `BITMAPINFOHEADER`. This utility is intended for small command-line workflows and learning about image filters; it does not aim to decode arbitrary image formats or preserve metadata from unrelated BMP variants.
