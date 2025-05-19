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

important details in the code
TODO


[1]: https://en.wikipedia.org/wiki/Search_engine_optimization