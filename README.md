# carrierwave-video-thumbnailer

[![Build Status](https://travis-ci.org/evrone/carrierwave-video-thumbnailer.png)](https://travis-ci.org/evrone/carrierwave-video-thumbnailer) [![Code Climate](https://codeclimate.com/badge.png)](https://codeclimate.com/github/evrone/carrierwave-video-thumbnailer)

* [Homepage](https://github.com/evrone/carrierwave-video-thumbnailer#readme)
* [Issues](https://github.com/evrone/carrierwave-video-thumbnailer/issues)
* [Documentation](http://rubydoc.info/gems/carrierwave-video-thumbnailer/frames)
* [Email](mailto:argentoff at gmail.com)

## READ ME

This fork of `carrierwave-video-thumbnailer` adds the ability to post-process the resulting thumbnail easily using MiniMagick, in case the default resizing and quality options are not sufficient. To expose as much of MiniMagick's API as possible, I opted for the user to pass in a Proc.

To use it, simply pass in a Proc as `mini_magick_opts`:

```ruby
# app/uploaders/video_uploader.rb
require 'mini_exiftool' # you need to add this gem, here is the info https://github.com/janfri/mini_exiftool

mini_magick_proc = Proc.new { |image,source|
  #now the source is passed, you can use here tools as mini_exiftool to check the rotation of the video, this to rotate the thumb only if is a protrait video
  movie_exifi = MiniExiftool.new(source)
  if movie_exifi.rotation == 90 # if was take in portrait, you rate it the image
    image.rotate "90"
  end
  image
}

version :preview_image do
  process thumbnail: [{format: 'jpg', quality: 8, size: 360, logger: Rails.logger, mini_magick_opts: mini_magick_proc}]
  def full_filename for_file
    jpg_name for_file, version_name
  end
end

def jpg_name for_file, version_name
  %Q{#{version_name}_#{for_file.chomp(File.extname(for_file))}.jpg}
end
```

To see the full extent of image modifications possible, [visit the MiniMagick repository](https://github.com/minimagick/minimagick).

## Description

A thumbnailer plugin for Carrierwave. It mixes into your uploader setup and
makes easy thumbnailing of your uploaded videos. This software is quite an
alpha right now so any kind of OpenSource collaboration is welcome.

## Features

Runs `ffmpegthumbnailer` with CLI keys provided by your configuration or just
uses quite a reasonable ffmpegthumbnailer's defaults. See Examples section for
details.

## Examples

Here's a working example:

In your Rails `app/uploaders/reel_uploader.rb`:

```ruby
class ReelUploader < CarrierWave::Uploader::Base
  include CarrierWave::Video  # for your video processing
  include CarrierWave::Video::Thumbnailer

  version :thumb do
    process thumbnail: [{format: 'png', quality: 10, size: 192, strip: true, logger: Rails.logger}]
    def full_filename for_file
      png_name for_file, version_name
    end
  end

  def png_name for_file, version_name
    %Q{#{version_name}_#{for_file.chomp(File.extname(for_file))}.png}
  end
end
```

## Thumbnailer Options

The options are passed as a hash to the `thumbnail` processing callback as
shown in the example. The options may be, according to ffmpegthumbnailer's
manual:

  * format: 'jpg' or 'png' ('jpg' is the default).
  * quality:  compression quality (1 to 10, default is 8).
  * size: thumbnail length in pixels (defaults to 128).
  * strip: movie film strip decoration (defaults to `false`).
  * seek: where to take the snapshot. May be specified as HH:MM:SS or X%.
    Defaults to 10%.
  * square: if set to `true` makes a square thumbnail regardless of an initial
    aspect ratio.
  * workaround: if set to `true` runs ffmpegthumbnailer in some safe mode
    (read `man ffmpegthumbnailer` for further explanations).
  * logger: an object behaving like Rails.logger (may be omitted).

## Requirements

`ffmpegthumbnailer` binary should be present on the PATH.

## Install

    $ gem install carrierwave-video-thumbnailer

Or 
```ruby
gem 'carrierwave-video-thumbnailer'
```
in your Gemfile.

## Contributions

If you'like to contribute to the code please note that we are now using [A
Successful Git Branching Model](http://nvie.com/posts/a-successful-git-branching-model/).
So, please commit your changes to `feature/*` branches and make Pull Requests
to the `development` branch. BTW you can install and use `git flow` Git plugin, too.
Thanks in advance!

## Acknowledgements

Huge Thanks to **Rachel Heaton** (<https://github.com/rheaton>) whose
`carrierwave-video` gem has inspired me (and where I've borrowed some code as
well).

Thanks to [Evrone Web Laboratory](http://evrone.com) which feeds me well enough (with the
tasks of course) to do this job.

## Copyright

Copyright (c) 2012 Pavel Argentov
Copyright (c) 2012 Evrone.com

See [LICENSE.txt](LICENSE.txt) for details.
