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

why
what
how
TODO


## The Code

only show the important function(s).
TODO

```python
#!/usr/bin/env python3

'''
This is a script to compress a batch of images.
'''

from pathlib import Path

from PIL import Image, ImageOps


# Configurable symbolic constants
IMG_DIR = Path(__file__).parent.parent / 'assets' / 'img' / '20241020'


if __name__ == '__main__':

    for img in IMG_DIR.iterdir():
        with (
            Image.open(img) as im,
            # We need to preserve EXIF information (like image orientation) before compression
            ImageOps.exif_transpose(im) as cim,
            ImageOps.exif_transpose(im) as lqip
        ):
            # Save iteration states
            im_size = img.stat().st_size / 1000

            print(
                f'Before compression, {img.name} size (pixels): {im.size}, '
                f'size (K bytes): {im_size}'
            )

            # Construct filenames
            compressed_file = IMG_DIR / f'compressed-{img.name}'
            lqip_file = IMG_DIR / f'lqip-{img.name}'

            # Compress and save images
            cim.save(compressed_file, 'jpeg', quality=65)
            lqip.save(lqip_file, 'jpeg', quality=15)

            # Save iteration states
            cim_size = compressed_file.stat().st_size / 1000
            lqip_size = lqip_file.stat().st_size / 1000

            # Print results
            print(
                f'After compression, {compressed_file.name} size (pixels): {cim.size}, '
                f'size (K bytes): {cim_size}, '
                f'compression rate = {cim_size / im_size}'
            )

            print(
                f'After compression, {lqip_file.name} size (pixels): {lqip.size}, '
                f'size (K bytes): {lqip_size}, '
                f'compression rate = {lqip_size / im_size}'
            )

```


## Important Details

important details in the code
TODO


[1]: https://en.wikipedia.org/wiki/Search_engine_optimization