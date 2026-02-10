# generate-apng Quick Start

## Installation

### Make Executable

	chmod +x generate-apng

### Install to System

	sudo cp generate-apng /usr/local/bin/

### Install Manpage

	pod2man generate-apng.pod > generate-apng.1
	gzip generate-apng.1
	sudo cp generate-apng.1.gz /usr/share/man/man1/

### View Documentation

	man generate-apng

Or view POD directly:

	perldoc generate-apng.pod

## Basic Usage

### Simple Animation

All frames with same delay:

	generate-apng -o animation.png -d 100 frame1.png frame2.png frame3.png

### Individual Delays

Each frame with specific delay:

	generate-apng -o animation.png frame1.png:100 frame2.png:150 frame3.png:100

### Using Frame List File

Create frames.txt:

	frame1.png 100
	frame2.png 150
	frame3.png 100

Generate:

	generate-apng -o animation.png --list frames.txt

### From Standard Input

	find . -name "frame*.png" | generate-apng -o anim.png --list - --delay 100

## Common Options

### Normalize Resolution

	generate-apng -o anim.png --normalize --size 800x600 frame*.png

### Background Color

	generate-apng -o anim.png --background white --normalize frame*.png

### Optimize for Web

	generate-apng -o anim.png --optimize --size 640x480 frame*.png

### Loop Control

	generate-apng -o anim.png --loop 3 frame*.png
	generate-apng -o anim.png --no-loop frame*.png

### Dry Run

	generate-apng -o anim.png --dry-run --verbose frame*.png

## Exit Codes

	0 - Success
	1 - General error
	2 - File not found
	3 - Invalid options
	4 - Output file exists (use --force)

## Dependencies

	Image::APNG module
	Image::Magick module

Install:

	cpan Image::APNG
	# or
	cpanm Image::APNG

