# spng-c3
A c3 binding for libspng/spng.c, a simple PNG file reader and writer. It has a lighter footprint and simpler interface than libpng but is still secure.

https://libspng.org/

The majority of the interface is methods of a spng::Context object. The library does its own memory management so this object should be allocated with `spng::new` or `spng::new2`, and freed with `spng::Context.free` rather than using c3 memory allocation.

## Usage

The binding is one file, compile it along with any other source files. The module name is 'spng'. There are two ways to link.

Firstly, you can add "spng" to project.json as a linked library. Most linux distros include libspng by default.

Secondly, given that libspng is designed to be small and portable, it is written as only one C file. You can simply add that one file to the project as a c-source file.

spng optionally uses zlib. This is a very common library and almost every linux distro will already have this installed. Add it to project.json as a linked library called simply "z".

## Example code
The example reads an image from an open file into a buffer. The decoded image would then be loaded into a graphics API, OpenGL, Vulkan, SDL or something similar. In this case it is simply removed from RAM again. Note that the library uses a libc file handle, not a c3 file object.
```
fn bool load_png (String path) {
	spng::Context *spng = spng::new (0);

	if (spng) {
		defer spng.free ();
		CFile fpPNG = libc::fopen (path.zstr_tcopy (), "rb");

		if (fpPNG) {
			defer libc::fclose (fpPNG);
			spng::Ihdr pngHdr;
			spng.set_png_file (fpPNG);
			spng.get_ihdr (&pngHdr);

			spng::Format spngFmt;
			int width = pngHdr.width;
			int height = pngHdr.height;

			switch (pngHdr.color_type) {
				case ColorType.GRAYSCALE:
					spngFmt = Format.G8;
					break;

				case ColorType.TRUECOLOR:
				case ColorType.INDEXED:
					spngFmt = Format.RGB8;
					break;

				case ColorType.TRUECOLOR_ALPHA:
					spngFmt = Format.RGBA8;
					break;
			}

      		usz size;
			spng.decoded_image_size (spngFmt, &size);
			char* pixels = libc::malloc (size);

			if (pixels) {
				spng.decode_image (pixels, size, spngFmt, 0);
        		libc::free (pixels);
				return true;
			}
		}
	}

	return false;
}
```
The library can also parse a PNG image that is already in RAM, if it is loaded from a resource for example, or over a network.
The process of saving a PNG file is equally simplified. Full documentation is available [here](https://libspng.org/docs/).
