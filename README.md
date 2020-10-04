# ImageDiff
ImageDiff

you need install CMake and OpenCV.

Clone with submodule.

```sh
git clone --recurse-submodules --remote-submodules https://github.com/Bensuperpc/ImageDiff.git
```

After:

```sh
./make.sh -DCMAKE_BUILD_TYPE=Release
```

After build, go to build and bin.

```sh
./diff img.png img.jpg
```

Program will output image file diff.png.
