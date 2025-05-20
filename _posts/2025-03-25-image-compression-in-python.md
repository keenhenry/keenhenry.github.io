---
layout: post
title:  Image Compression in Python
date:   2025-03-25
categories: [technology, programming, python , notes]
tags: [technology, image processing, programming, python, notes]
description: How to compress images in Python?
---

Recently I had to write a script to compress batches of images for this personal blog; this is a part
of the efforts to improve [**SEO**][1].

Here I am keeping some notes about the general ideas and important details of the script:


## The Big Picture

### Why

Images displayed on a website are slowing down loading web pages in your brower. Depends on how many images
a site has and how big they are, this might hinder user-experience of the website.

In addition to that, compressed images in most cases are indiscernible in quality for most users, even
if with a staggering `40%` size-reduction lossy compression (see [does size matter?][size-matter])!

Therefore, it makes sense to compress images of my personal website to improve UX without sacrificing
the perceieved image quality[^lossy].

### What

A lot of images used by my site are JPEG images, they can be easily further compressed. The goal is to keep
the quality level at around `65%` of the original images, and keep only `15%` quality level for the [LQIP][lqip]
version of the original images.

The choice of `65%` and `15%` was not random; I did some experiments to find out the 'best' **maximum** compression
rate which does not produce visually perceptible quality degradation for my site.

I do not want to manually compress images because I have many pictures on the site; I'd like to compress them in one
go by a script. So, I need find software tools that support JPEG image compression and can control compression quality
level (how much data is kept after being compressed).


### How

After some simple research, I found popular tools like [`ImageMagick`][imagemagick] and Python library [`pillow`][pillow]
are viable solutions. Eventually, I chose to use Python because I enjoy writing Python program more :smirk:.


## The Code

:tada: Here you go:

```python
def compress_at(dir: Path) -> None:
    '''Function to compress photos in a directory `dir`'''

    for img in dir.iterdir():
        with (
            Image.open(img) as im,
            # We need to preserve EXIF information (like image orientation) before compression
            ImageOps.exif_transpose(im) as cim,
            ImageOps.exif_transpose(im) as lqip
        ):
            # Remember size before compression and report to the user
            im_size = img.stat().st_size / 1000

            print(
                f'Before compression, {img.name} was {im.size[0]}x{im.size[1]} pixels, '
                f'and {im_size} kB'
            )

            # Construct filenames
            compressed_file = dir / f'compressed-{img.name}'
            lqip_file       = dir / f'lqip-{img.name}'

            # Compress and save images
            cim.save(compressed_file, 'jpeg', quality=65)
            lqip.save(lqip_file, 'jpeg', quality=15)

            # Remember size after compression and report to the user
            cim_size  = compressed_file.stat().st_size / 1000
            lqip_size = lqip_file.stat().st_size       / 1000

            # Print results
            print(
                f'After compression, {compressed_file.name} is {cim.size[0]}x{cim.size[1]} pixels, '
                f'and {cim_size} kB. Compression rate = {cim_size / im_size}'
            )

            print(
                f'After compression, {lqip_file.name} is {lqip.size[0]}x{lqip.size[1]} pixels, '
                f'and {lqip_size} kB. Compression rate = {lqip_size / im_size}'
            )
```


## Important Details

Pay attention to the following code:

```python
            # We need to preserve EXIF information (like image orientation) before compression
            ImageOps.exif_transpose(im) as cim,
            ImageOps.exif_transpose(im) as lqip
```

This code is necessary otherwise the image after compression might get wrong *orientation*. This is because
`EXIF` contains orientation metatdata of an image, and when saving the new image during compression, this metadata
information is not saved along.

Check this post: https://alexwlchan.net/til/2024/photos-can-have-orientation-in-exif/

TODO


## Footnotes

[^lossy]: Pay attention! Image compression only makes sense when applying to *lossy compression format*, like [JPEG][2] (PNG is a lossless image format!) 


[1]: https://en.wikipedia.org/wiki/Search_engine_optimization
[2]: https://en.wikipedia.org/wiki/JPEG
[size-matter]: https://www.keptlight.com/does-size-matter/
[lqip]: https://www.guypo.com/introducing-lqip-low-quality-image-placeholders
[pillow]: https://pillow.readthedocs.io/en/stable/
[imagemagick]: https://en.wikipedia.org/wiki/ImageMagick
