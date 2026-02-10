# generate-apng

Create Animated PNG files from individual PNG images

## Synopsis

	generate-apng [options] [file[:delay]] ...

## Description

**generate-apng** creates Animated PNG (APNG) files from a sequence of PNG images. Each frame can have an individual display delay specified in milliseconds.

APNG is a backward-compatible extension of PNG that supports animation, providing 24-bit images with 8-bit transparency as an alternative to animated GIF files.

## Options

### Required Options

**-o file, --output=file**

Output APNG filename. This option is required.

### Frame Input Options

**-d ms, --delay=ms**

Default delay in milliseconds for frames that don't specify their own delay. Default: 100 ms.

**-l file, --list=file**

Read frame list from a file. Use `-` to read from standard input. The file should contain one frame per line with format: `filename delay`

**file[:delay]**

Positional arguments specifying frame files. Optionally append `:delay` to specify frame-specific delay in milliseconds. Files without delay specification use the default delay.

### Animation Control Options

**-L count, --loop=count**

Number of times to loop the animation. Use 0 for infinite looping. Default: 0 (infinite).

**--no-loop**

Play animation once. Equivalent to `--loop 1`.

### Resolution Options

**-n, --normalize**

Normalize all frames to the same resolution. Smaller frames are centered on a background canvas.

**--width=pixels**

Target width in pixels. Must be used with `--height`. Automatically enables normalization.

**--height=pixels**

Target height in pixels. Must be used with `--width`. Automatically enables normalization.

**--size=WIDTHxHEIGHT**

Target resolution in format WIDTHxHEIGHT (e.g., 800x600). Automatically enables normalization.

### Background Options

**-b color, --background=color**

Background color for padding smaller frames when normalizing.

Color formats:

	R,G,B,A
		RGBA values (0-255). Example: 255,255,255,128

	R,G,B
		RGB values (0-255), alpha defaults to 255. Example: 255,255,255

	N
		Grayscale value (0-255), defaults to opaque. Example: 128

	transparent
		Fully transparent black (0,0,0,0)

	white
		Opaque white (255,255,255,255)

	black
		Opaque black (0,0,0,255)

Default: transparent (0,0,0,0)

### Optimization Options

**-O, --optimize**

Enable palette optimization. Converts frames to 8-bit indexed PNG with up to 256 colors per frame. Reduces file size but may introduce color banding in photographs or gradients.

### Frame Behavior Options

**--disposal=mode**

Frame disposal method. Specifies how the frame area is handled after display.

Modes:

	none or 0
		Frame remains, next frame drawn on top.

	background or 1
		Frame area cleared to transparent before next frame (default).

	previous or 2
		Restore to state before current frame.

Default: background (1)

**--blend=mode**

Frame blending operation. Specifies how frame is composited onto canvas.

Modes:

	source or 0
		Frame replaces canvas content completely.

	over or 1
		Frame alpha-blended over existing canvas (default).

Default: over (1)

### Output Control Options

**-v, --verbose**

Enable verbose output. Shows warnings and detailed processing information.

**-q, --quiet**

Suppress non-error output. Only errors are displayed.

**--force**

Overwrite existing output file without prompting.

**--dry-run**

Show what would be done without creating the output file.

### Help Options

**-h, --help**

Display brief usage information and exit.

**--version**

Display version information and exit.

## Frame List File Format

Frame list files can be specified with `--list` option. The file should contain one frame per line with the following format:

### Space-separated format

	filename delay

Example:

	frame1.png 100
	frame2.png 150
	frame3.png 100

### Colon-separated format

	filename:delay

Example:

	frame1.png:100
	frame2.png:150
	frame3.png:100

### Comments

Lines starting with `#` are treated as comments and ignored. Inline comments (`#` after content) are also supported.

Example:

	# Animation frames
	frame1.png 100    # Intro
	frame2.png 150    # Main
	frame3.png 100    # Outro

Empty lines and whitespace are ignored.

## Exit Status

The program returns the following exit codes:

**0**

Success. APNG file created successfully.

**1**

General error. APNG generation failed.

**2**

File error. Input file not found or not readable.

**3**

Invalid options. Command-line options are incorrect or incomplete.

**4**

Output file exists. Use `--force` to overwrite.

## Examples

### Simple Animation

Create animation from all PNG files with default 100ms delay:

	generate-apng -o animation.png frame*.png

### Individual Frame Delays

Specify different delay for each frame using colon notation:

	generate-apng -o animation.png \
		intro.png:500 \
		action1.png:100 \
		action2.png:100 \
		outro.png:1000

### Using Frame List File

Create frames.txt:

	frame1.png 100
	frame2.png 150
	frame3.png 100

Generate animation:

	generate-apng -o animation.png --list frames.txt

### Normalized with Background

Create animation with all frames normalized to 800x600 with white background:

	generate-apng -o banner.png \
		--size 800x600 \
		--background white \
		--delay 2000 \
		banner_*.png

### Optimized for Web

Create optimized animation with palette reduction:

	generate-apng -o web_anim.png \
		--optimize \
		--normalize \
		--size 640x480 \
		--list frames.txt

### Reading from Standard Input

Generate animation from find command output:

	find . -name "frame*.png" -type f | \
		generate-apng -o anim.png --list - --delay 100

### Combining List and Arguments

Use frame list and add additional frames:

	generate-apng -o anim.png \
		--list frames.txt \
		extra1.png:200 \
		extra2.png:300

### Loop Three Times

Create animation that plays three times then stops:

	generate-apng -o product.png \
		--loop 3 \
		front.png:1000 \
		side.png:1000 \
		back.png:1000

### Dry Run

Preview what would be created without generating file:

	generate-apng -o test.png --dry-run --verbose \
		frame1.png:100 \
		frame2.png:150

## Warnings

### Long Delays

When using `--verbose`, warnings are displayed for frame delays exceeding 5000 milliseconds (5 seconds).

### File Validation

All input files are validated for existence and readability before processing. If any file is missing or inaccessible, the program exits with error code 2.

### Overwrite Protection

By default, the program will not overwrite existing files. Use `--force` to allow overwriting.

## Dependencies

**Image::APNG**

Perl module for APNG generation.

**Image::Magick**

Required by Image::APNG for image manipulation.

## Browser Support

APNG is supported in modern browsers:

- Firefox (all versions)
- Chrome/Edge 59+
- Safari 8+
- Opera 46+

For unsupported browsers, the first frame displays as a static PNG image.

## Installation

### Generate Manpage from POD

	pod2man generate-apng.pod > generate-apng.1
	gzip generate-apng.1
	sudo cp generate-apng.1.gz /usr/share/man/man1/

### Install Script

	chmod +x generate-apng
	sudo cp generate-apng /usr/local/bin/

### View Manpage

	man generate-apng

Or view POD directly:

	perldoc generate-apng.pod

## See Also

- Image::APNG
- APNG Specification: https://wiki.mozilla.org/APNG_Specification
- PNG Specification: http://www.w3.org/TR/PNG/

## Author

APNG Generator Author

## Copyright

This software is copyright (c) 2025 by the author.

This is free software; you can redistribute it and/or modify it under the same terms as the Perl 5 programming language system itself.
