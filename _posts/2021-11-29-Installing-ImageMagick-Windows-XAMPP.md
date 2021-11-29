---
layout: post
title: Installing ImageMagick to Windows (XAMPP and PHP 8) 
---

Haven't developed with PHP for a while but now I got a need to do some updates to an older project. The project is based on PHP, MYSQL and HTML (+ jquery) and it also uses [TCPDF](https://tcpdf.org/) library for generating PDF files.

After installing [XAMPP](https://www.apachefriends.org/index.html) with PHP 8 and editing some settings, I managed to get stuff working. 
However, I noticed that the TCPDF won't work with default XAMPP installation as it rquires GD or ImageMagick library installed.

So I received an error when trying to create a PDF:

`TCPDF requires the Imagick or GD extension to handle PNG images with alpha channel.`

## Installing ImageMagick

It seems that I forgot how annoying installing PHP libraries *can* be.

I tried many guides, including https://ourcodeworld.com/articles/read/349/how-to-install-and-enable-the-imagick-extension-in-xampp-for-windows without success. 
There is good discussion here too: https://stackoverflow.com/questions/39609951/cannot-load-imagick-library

I always ended with the following error inside Apache `error.log` even though the file was there:

`PHP Warning:  PHP Startup: Unable to load dynamic library 'imagick' (tried: C:\\xampp\\php\\ext\\imagick (The specified module could not be found), C:\\xampp\\php\\ext\\php_imagick.dll (The specified module could not be found)) in Unknown on line 0`

Then I noticed that I had x86 version installed, but my phpinfo() told that I have x64 php installation. However, installing x64 versions didn't help, the same problem was still there.

### Final solution (for me)
0. Delete ImageMagick if installed previously 
1. Check using `phpinfo()` what version is installed
    - x64 or x86?
    - Thread safety enabled / disabled
    - PHP version

![image](https://user-images.githubusercontent.com/13457157/143831999-f8adf39b-158f-4240-8683-19c97252e2f3.png)

2. Download latest version from https://windows.php.net/downloads/pecl/releases/imagick/
    - I had x64 PHP, thread safety enabled and PHP 8
    - Latest imagick version was 3.6.0rc2
    - So my file was: `php_imagick-3.6.0rc2-8.0-ts-vs16-x64.zip` from https://windows.php.net/downloads/pecl/releases/imagick/3.6.0rc2/
3. Copy `php_imagick.dll` to `C:\xampp\php\ext`
4. Copy all other files to `C:\xampp\php`
5. Edit `php.ini` and add the following line under `Dynamic Extensions` 
  - `extension=imagick`
  - NOTE: In PHP8 the `php_` and `.dll` are not longer used in extension filename
6. Restart Apache
