# photo-tools

A humble collection of command line tools for managing photos. Most require that you have [ExifTool](https://exiftool.org) and/or
[ImageMagick](https://imagemagick.org) installed.

Caveat Emptor: Some of these modify image metadata in place. Always use backups until you're sure the behavior is as you would expect.

So far only tested on macOS.

## Tools

### `perceptual-hash`

Usage: `perceptual-hash filename`

Prints a [perceptual hash](https://en.wikipedia.org/wiki/Perceptual_hashing) of an image, as a binary string.
Useful for finding near-duplicate images, like processed versions of an original or exposure bracket sets.

The algorithm is resistent to even fairly large color changes (like saturation, contrast, brightness, etc.),
resizing, and minor rotation and cropping (such as straightening by a few degress), but is defeated by any major
cropping or rotation.

The hash is 144 bits, and near-duplicates will typically differ by no more than a dozen bits or so, though some
tuning is required to find the right balance between false positives and false negatives.

Based on the [dHash algorithm](http://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html).
Requires ImageMagick and Python 3.

### `index-images` (experimental)

Usage: `image-index [--phash] [--md5] db dir`

Creates a simple SQLite database of image metadata, from images found in `dir`. The database can be initialized as follows:

```
$ sqlite3 test.db < schema/image-index.schema
```

The `--phash` flags adds a perceptual hash to each row. Needed by `perceptual-neighbors`. The `--md5` flag adds a MD5 hash,
which is useful for finding exact duplicates. (Yes, MD5 is cryptographically broken. But it's reasonably fast and this is
as-of-yet toy project.)

Note that this is EXPERIMENTAL. In particular it does very little validation or massaging of values, and will explode on
wonky EXIF data.

Requires ExifTool, ImageMagick (if `--phash` is used), Python 3, and SQLite 3.

### `perceptual-neighbours` (experimental)

Usage: `perceptual-neighbours [--preview] db`

Given a database populated by `index-images`, prints lists of images whose perceptual hash differ by no more than a given
limit, and are possibly visually similar.

The appropriate limit (currently hardcoded to 20) will need tuning depending on the use-case. Too high a value will give a
lot of false positives with large databases, and too low will miss near-duplicates.

The optional `--preview` flag will open each set of matches in Preview on macOS, and halt execution until Preview is closed.

Requires Python 3 and SQLite 3.

### `import-sidecar-gps`

Usage: `import-sidecar-gps directory`

Applies GPS tags from sidecar files (e.g image.jpg.xmp) to the matching base file (e.g. image.jpg). This comes in handy
if you want to use e.g. Darktable for geotagging, and want geotags applied to the originals.

Requires ExifTool.

### `missing-geotags`

Usage: `missing-geotags directory`

Prints paths of files which do not have GPS tags.

Requires ExifTool.

### `strip-exif`

Usage: `strip-exif target(s)`

Removes EXIF data from target(s). Note that ExifTool, and therefore `strip-exif`, does not guarantee complete removal
of all metadata. This applies in particular to RAW formats because some proprietary tags may be necessary for rendering
the image.

### `strip-exif-gps`

Usage: `strip-exif-gps target(s)`

Removes location data, in the form of standard GPS EXIF tags, from target(s).

Note that vendors may have additional proprietary location tags, which will not
be removed.

### `exif-summary`

Usage: `exif-summary target(s)`

Prints summary EXIF info, rather than the firehose ExifTool gives you by default.

Requires ExifTool.

## Installation

The bad news is there's no fancy installation method. The good news is you only need to add the files in `./bin` to a location in your `$PATH`.
