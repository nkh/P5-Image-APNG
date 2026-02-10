# APNGGenerator - Animated PNG File Generator

## Overview

APNGGenerator is a Perl module for creating Animated PNG (APNG) files from a sequence of individual PNG images. APNG is a backward-compatible extension of PNG that supports animation, providing 24-bit images with 8-bit transparency as an alternative to animated GIF files.

## Features

- Generate APNG files from multiple PNG images with individual frame delays
- Support for images with different resolutions
- Automatic resolution normalization with centered frame placement
- Optional palette optimization (convert to 8-bit indexed PNG)
- Configurable background colors for frame padding
- Full control over animation parameters (loop count, disposal, blending)
- Robust error handling with detailed error messages
- Automatic handling of corrupt images with blank frame replacement

## Installation

### Dependencies

The module requires :

- **Image::Magick** - Required for image manipulation

Install Image::Magick:

```bash
# Using CPAN
cpan Image::Magick

# Using cpanm
cpanm Image::Magick

# On Debian/Ubuntu
apt-get install libimage-magick-perl

# On RedHat/CentOS
yum install perl-Image-Magick
```

### Module Installation

Place `APNGGenerator.pm` in your Perl library path or include it directly:

```perl
use lib '/path/to/module';
use Image::APNG;
```

## Basic Usage

### Simple Animation

```perl
use Image::APNG ;

# Define frames: [filename, delay_in_milliseconds]
my $frames = 
	[
		['frame1.png', 100],
		['frame2.png', 150],
		['frame3.png', 100],
		['frame4.png', 200]
	];

# Generate APNG with default options
my $result = Image::APNG::generate($frames);

# Save to file
if ($result->{status} == 0)
	{
	open my $fh, '>', 'animation.png';
	binmode $fh;
	print $fh $result->{data};
	close $fh;
	print "APNG created successfully!\n";
	}
else 
	{
	print "Errors occurred:\n";
	print "$_\n" for @{$result->{errors}};
	}
```

### Advanced Usage with Options

```perl
use APNGGenerator;

my $frames = 
	[
		['logo_large.png', 500],
		['logo_small.png', 500],
		['logo_wide.png', 500]
	];

my $options = {
	loop_count           => 3,                    # Loop 3 times (0 = infinite)
	normalize_resolution => 1,                    # Make all frames same size
	target_resolution    => [800, 600],           # Override automatic sizing
	background_color     => [255, 255, 255, 128], # Semi-transparent white
	optimize_palette     => 1,                    # Convert to 8-bit palette
	disposal_method      => 1,                    # Clear to background
	blend_operation      => 1                     # Alpha blend over previous
};

my $result = Image::APNG::generate($frames, $options);

if ($result->{status} == 0) 
	{
	# Success - save the file
	open my $fh, '>', 'optimized_animation.png';
	binmode $fh;
	print $fh $result->{data};
	close $fh;
	}
```

## Function Reference

### generate($frames, $options)

Main function for generating APNG files.

#### Parameters

**$frames** (required) - Array reference of frame definitions

Each frame is an array reference containing:
- `$filename` (string)  - Path to PNG file
- `$delay_ms` (integer) - Display duration in milliseconds

Example:
```perl
my $frames = 
	[
		['/path/to/image1.png', 100],
		['/path/to/image2.png', 150],
		['/path/to/image3.png', 100]
	];
```

**$options** (optional) - Hash reference of configuration options

See Options section below for details.

#### Return Value

Returns a hash reference with the following keys:

- **status** (integer)
  - `0` = Success
  - `1` = Error occurred (check errors array)

- **errors** (array reference)
  - List of error messages encountered during processing
  - Empty array if no errors occurred

- **data** (binary string or undef)
  - Complete APNG file data ready to write
  - `undef` if generation failed

Example:
```perl
my $result = Image::APNG::generate($frames, $options);

if ($result->{status} == 0)
	{
	# Success
	my $apng_data = $result->{data};
	# Write to file or process further
	}
else 
	{
	# Error
	foreach my $error (@{$result->{errors}})
		{
		warn "Error: $error\n";
		}
	}
```

## Options

All options are optional. Default values are used if not specified.

### Animation Control Options

#### loop_count

**Type:** Integer  
**Default:** `0`  
**Description:** Number of times the animation should loop

- `0` = Infinite looping (animation plays forever)
- `1` = Play once then stop on last frame
- `n` = Play n times then stop on last frame

Example:
```perl
loop_count => 0,  # Infinite loop (default)
loop_count => 1,  # Play once
loop_count => 5,  # Loop 5 times
```

### Resolution Handling Options

#### normalize_resolution

**Type:** Boolean (0 or 1)  
**Default:** `0`  
**Description:** Whether to make all frames the same resolution

When enabled:
- Uses the largest image dimensions by default
- Smaller images are centered on a background canvas
- Can be overridden with `target_resolution`

Example:
```perl
normalize_resolution => 1,  # Enable normalization
```

#### target_resolution

**Type:** Array reference `[width, height]`  
**Default:** `undef` (automatic)  
**Description:** Explicit resolution for all frames

Overrides automatic resolution detection. All frames will be resized/padded to match this resolution.

Example:
```perl
target_resolution => [1920, 1080],  # Full HD
target_resolution => [800, 600],    # SVGA
```

**Note:** Only used when `normalize_resolution` is enabled.

#### background_color

**Type:** Array reference `[R, G, B, A]`  
**Default:** `[0, 0, 0, 0]` (transparent black)  
**Range:** 0-255 for each component  
**Description:** Background color for padding smaller frames

Components:
- **R** - Red (0-255)
- **G** - Green (0-255)
- **B** - Blue (0-255)
- **A** - Alpha/opacity (0=transparent, 255=opaque)

Examples:
```perl
background_color => [0, 0, 0, 0],         # Transparent (default)
background_color => [255, 255, 255, 255], # Opaque white
background_color => [255, 0, 0, 128],     # Semi-transparent red
background_color => [240, 240, 240, 255], # Light gray
```

### Optimization Options

#### optimize_palette

**Type:** Boolean (0 or 1)  
**Default:** `0`  
**Description:** Convert frames to 8-bit indexed PNG with 256 colors or fewer

When enabled:
- Reduces file size significantly
- Uses Floyd-Steinberg dithering for better quality
- Preserves transparency
- May introduce color banding in gradients

Best for:
- Simple graphics, logos, icons
- Cartoon-style images
- Images with limited color palettes

Avoid for:
- Photographs
- Images with gradients
- High-fidelity color requirements

Example:
```perl
optimize_palette => 1,  # Enable 8-bit palette optimization
```

### Frame Behavior Options

#### disposal_method

**Type:** Integer (0, 1, or 2)  
**Default:** `1` (APNG_DISPOSE_OP_BACKGROUND)  
**Description:** How to handle the frame area after it's displayed

Values:

- **0 - APNG_DISPOSE_OP_NONE**
  - Frame remains, next frame drawn on top
  - Use for: Full-frame updates where each frame is complete
  
- **1 - APNG_DISPOSE_OP_BACKGROUND** (Recommended)
  - Frame area cleared to transparent before next frame
  - Use for: Standard animations, prevents ghosting
  
- **2 - APNG_DISPOSE_OP_PREVIOUS**
  - Restore to state before current frame
  - Use for: Temporary overlays or effects

Example:
```perl
disposal_method => 1,  # Clear to background (default, recommended)
```

#### blend_operation

**Type:** Integer (0 or 1)  
**Default:** `1` (APNG_BLEND_OP_OVER)  
**Description:** How to composite frame onto canvas

Values:

- **0 - APNG_BLEND_OP_SOURCE**
  - Frame replaces canvas content completely
  - Ignores alpha channel for blending
  - Use for: Opaque frames

- **1 - APNG_BLEND_OP_OVER** (Recommended)
  - Frame alpha-blended over existing canvas
  - Standard alpha compositing
  - Use for: Frames with transparency

Example:
```perl
blend_operation => 1,  # Alpha blend (default, recommended)
```

### Recommended Option Combinations

**Standard Animation (Most Common)**
```perl
my $options =
	{
	loop_count => 0,
	disposal_method => 1,
	blend_operation => 1
	} ;
```

**Size-Normalized Animation with Background**
```perl
my $options =
	{
	normalize_resolution => 1,
	background_color => [255, 255, 255, 255],  # White background
	loop_count => 0,
	disposal_method => 1,
	blend_operation => 1
	} ;
```

**Optimized for Web (Small File Size)**
```perl
my $options =
	{
	optimize_palette => 1,
	normalize_resolution => 1,
	target_resolution => [640, 480],
	loop_count => 0
	} ;
```

## Error Handling

### Error Types

The module handles several types of errors gracefully:

1. **Missing or Invalid Input**
   - No frames provided
   - Empty frame list

2. **File I/O Errors**
   - File not found
   - Permission denied
   - Invalid file format

3. **Image Processing Errors**
   - Corrupt PNG files
   - Unsupported image formats
   - Palette optimization failures

### Corrupt Image Handling

When a corrupt or invalid image is encountered:

1. An error message is added to the errors array
2. If a previous valid frame exists:
   - A blank frame with the same dimensions is created
   - The animation continues with the blank frame
3. If no previous valid frame exists:
   - Additional error logged
   - Frame is skipped

Example:
```perl
my $result = Image::APNG::generate($frames);

if ($result->{status} == 1)
	{
	print "Errors encountered:\n";
	foreach my $error (@{$result->{errors}})
		{
		print "  - $error\n";
		}
	
	# Data may still be available with some frames skipped
	if (defined $result->{data}) 
		{
		print "Partial APNG data generated\n";
		}
	}
```

### Best Practices

1. **Always check the status code**
```perl
die "APNG generation failed" if $result->{status} != 0;
```

2. **Log all errors for debugging**
```perl
if (@{$result->{errors}}) 
	{
	foreach my $error (@{$result->{errors}})
		{
		warn "WARNING: $error\n";
		}
	}
```

3. **Validate input files before processing**
```perl
foreach my $frame (@$frames)
	{
	die "File not found: $frame->[0]" unless -f $frame->[0];
	}
```

## Technical Details

### APNG Specification Compliance

The module implements the APNG 1.0 specification as defined at:
https://wiki.mozilla.org/APNG_Specification

Key implementation details:

- **PNG Signature:** Standard 8-byte PNG signature
- **IHDR Chunk:** Image header with dimensions from first frame
- **acTL Chunk:** Animation control (frame count, loop count)
- **fcTL Chunk:** Frame control for each frame (timing, placement, disposal, blending)
- **IDAT Chunk:** First frame data (standard PNG chunk)
- **fdAT Chunks:** Subsequent frame data (APNG-specific)
- **IEND Chunk:** End of PNG file marker

### Chunk Structure

All chunks follow PNG chunk format:
```
[4 bytes: Length] [4 bytes: Type] [Length bytes: Data] [4 bytes: CRC32]
```

### Frame Timing

Frame delays are specified in milliseconds and converted to APNG format:
- Numerator: delay in milliseconds
- Denominator: 1000 (seconds)

Example: 150ms delay = 150/1000 = 0.15 seconds

### Color Type Support

The module supports all PNG color types:
- **0** - Grayscale
- **2** - Truecolor (RGB)
- **3** - Indexed (Palette)
- **4** - Grayscale with Alpha
- **6** - Truecolor with Alpha (RGBA)

### Compression

Uses PNG compression level 9 (maximum compression) for optimal file size.

## Performance Considerations

### Memory Usage

Memory usage depends on:
- Number of frames
- Image resolution
- Color depth

Approximate memory calculation:
```
Memory ≈ (Width × Height × 4 bytes × Number of Frames) + Overhead
```

Example: 10 frames of 1920×1080 RGBA:
```
≈ (1920 × 1080 × 4 × 10) / 1024 / 1024 ≈ 79 MB
```

### Processing Time

Factors affecting processing time:
1. Number of frames
2. Image resolution
3. Palette optimization (adds significant time)
4. Resolution normalization (moderate impact)

Typical performance:
- 10 frames @ 1920×1080: ~2-5 seconds (no optimization)
- 10 frames @ 1920×1080: ~5-15 seconds (with palette optimization)
- 100 frames @ 640×480: ~10-20 seconds (no optimization)

### Optimization Tips

1. **Pre-process images**
   - Resize images before passing to module
   - Convert to appropriate color depth externally if needed

2. **Batch processing**
   - Process multiple APNGs in parallel if system resources allow

3. **Palette optimization**
   - Only use for appropriate image types
   - Consider pre-optimizing externally with tools like pngquant

4. **Resolution normalization**
   - Pre-normalize images if generating multiple APNGs with same frames

## Examples

### Example 1: Simple Loading Animation

```perl
use APNGGenerator;

# Create spinner frames
my $frames = 
	[
		['spinner_01.png', 100],
		['spinner_02.png', 100],
		['spinner_03.png', 100],
		['spinner_04.png', 100],
		['spinner_05.png', 100],
		['spinner_06.png', 100],
		['spinner_07.png', 100],
		['spinner_08.png', 100]
	];

my $result = Image::APNG::generate($frames, {loop_count => 0});

if ($result->{status} == 0)
	{
	open my $fh, '>', 'spinner.png';
	binmode $fh;
	print $fh $result->{data};
	close $fh;
	}
```

### Example 2: Product Showcase

```perl
use APNGGenerator;

# Show product from different angles
my $frames = 
	[
		['product_front.png', 1000],
		['product_side.png', 1000],
		['product_back.png', 1000],
		['product_top.png', 1000]
	];

my $options = 
	{
	loop_count => 3,              # Show 3 times then stop
	normalize_resolution => 1,
	background_color => [255, 255, 255, 255],  # White background
	optimize_palette => 1          # Reduce file size
	};

my $result = Image::APNG::generate($frames, $options);

if ($result->{status} == 0)
	{
	open my $fh, '>', 'product_showcase.png';
	binmode $fh;
	print $fh $result->{data};
	close $fh;
	}
```

### Example 3: Banner Ad with Different Sizes

```perl
use APNGGenerator;

# Frames of different sizes
my $frames =
	[
		['banner_wide.png', 2000],    # 800x200
		['banner_square.png', 2000],  # 400x400
		['banner_tall.png', 2000]     # 200x600
	];

my $options = 
	{
	normalize_resolution => 1,
	target_resolution    => [800, 600],  # Force specific size
	background_color     => [240, 240, 240, 255],  # Light gray
	loop_count           => 0
	};

my $result = Image::APNG::generate($frames, $options);

if ($result->{status} == 0)
	{
	open my $fh, '>', 'banner_ad.png';
	binmode $fh;
	print $fh $result->{data};
	close $fh;
	}
```

### Example 4: Error Handling Example

```perl
use APNGGenerator;

my $frames =
	[
		['frame1.png', 100],
		['missing_frame.png', 100],  # This file doesn't exist
		['frame3.png', 100],
		['corrupt_frame.png', 100],  # This file is corrupt
		['frame5.png', 100]
	];

my $result = Image::APNG::generate($frames);

if ($result->{status} == 0)
	{
	print "APNG created successfully!\n";
	
	if (@{$result->{errors}}) 
		{
		print "Warnings:\n";
		print "  $_\n" for @{$result->{errors}};
		}
	
	open my $fh, '>', 'output.png';
	binmode $fh;
	print $fh $result->{data};
	close $fh;
	}
else 
	{
	print "APNG creation failed!\n";
	print "Errors:\n";
	print "  $_\n" for @{$result->{errors}};
	}
```

## Troubleshooting

### Common Issues

**Issue: "Failed to load image"**
- **Cause:** File not found, incorrect permissions, or corrupt file
- **Solution:** Verify file path and permissions; check file integrity

**Issue: "No frames provided"**
- **Cause:** Empty or undefined frames array
- **Solution:** Ensure frames array contains at least one frame

**Issue: Large file sizes**
- **Cause:** High resolution images, many frames, or 24-bit color
- **Solution:** Enable `optimize_palette` option or reduce image resolution

**Issue: Color banding after optimization**
- **Cause:** Palette optimization on photographs or gradients
- **Solution:** Disable `optimize_palette` for high-quality images

**Issue: Animation plays too fast/slow**
- **Cause:** Incorrect delay values
- **Solution:** Adjust frame delays (values are in milliseconds)

**Issue: Frames appear offset or misaligned**
- **Cause:** Mixed resolutions without normalization
- **Solution:** Enable `normalize_resolution` option

### Debug Mode

Add verbose error checking:

```perl
my $result = Image::APNG::generate($frames, $options);

print "Status: " . ($result->{status} == 0 ? "SUCCESS" : "FAILURE") . "\n";
print "Errors count: " . scalar(@{$result->{errors}}) . "\n";
print "Data size: " . (defined $result->{data} ? length($result->{data}) : 0) . " bytes\n";

if (@{$result->{errors}}) 
	{
	print "\nDetailed errors:\n";
	my $i = 1;
	
	foreach my $error (@{$result->{errors}})
		{
		print "$i. $error\n";
		$i++;
		}
	}
```

## Browser Support

APNG is supported in:

- **Firefox:** Full support (all versions)
- **Chrome/Edge:** Version 59+
- **Safari:** Version 8+
- **Opera:** Version 46+
- **iOS Safari:** Version 8+
- **Android Chrome:** Version 59+

For unsupported browsers, the first frame will display as a static PNG image (backward compatibility).

## License

Same license as Perl or GPL v3

## See Also

- APNG Specification: https://wiki.mozilla.org/APNG_Specification
- PNG Specification: http://www.w3.org/TR/PNG/
- Image::Magick Documentation: https://imagemagick.org/script/perl-magick.php

## Support

For issues, questions, or contributions, please refer to the module documentation and the APNG specification.
