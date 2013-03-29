# php-auto-rotate - Automatic rotation of JPEGs

This is pseudo code for performing auto-rotation of JPEGs (like those from an iPhone, taken sideways) 
when using ImageMagick. Documented here to avoid me having to look it up again.

To start, find out image info:

    $ident_out = shell_exec("/usr/bin/identify -format \"%w|%h|%m\" {$filename}[0] 2>&1");
    list($w, $h, $format) = explode('|', trim($ident_out));

Next we can try and extract the orientation from the IFD0 block. Orientations 1 through 4 keep the same 
aspect ratio (id=1 means the identity), while 5 through 8 flip the width/height. Possible values are 
explained here:

http://sylvana.net/jpegcrop/exif_orientation.html

    $rotate = '';
    $rotate_aspect = false;

    if ($format == 'JPEG' || $format == 'TIFF'){

      $meta = exif_read_data($filename, null, true);
      if (is_array($meta) && is_array($meta['IFD0'])){

    	$or = $meta['IFD0']['Orientation'];

    	if ($or == 2) $rotate = '-flop';
    	if ($or == 3) $rotate = '-rotate 180';
    	if ($or == 4) $rotate = '-flip';
    	if ($or == 5) $rotate = '-flop -rotate 270';
    	if ($or == 6) $rotate = '-rotate 90';
    	if ($or == 7) $rotate = '-flop -rotate 90';
    	if ($or == 8) $rotate = '-rotate 270';

    	if ($or >= 5 && $or <=8) $rotate_aspect = true;
      }
    }

If we're going to be doing calculations based on source image size, swap them now for 
images rotated on their sides.

    if ($rotate_aspect){
      $t = $w;
      $w = $h;
      $h = $t;
    }

Now we have everything we need to e.g. produce thumbnails:

    exec("/usr/bin/convert {$filename}[0] {$rotate} (OTHER STUFF HERE) 2>&1", $ret, $exit);

