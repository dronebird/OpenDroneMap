![](https://opendronemap.github.io/OpenDroneMap/img/odm_image.png)

What is it?
===========

OpenDroneMap is a toolchain for processing raw civilian UAS imagery to other useful products. What kind of products?

1. Point Clouds
2. Digital Surface Models
3. Textured Digital Surface Models
4. Orthorectified Imagery
5. Classified Point Clouds
6. Digital Elevation Models
7. etc.

So far, it does Point Clouds, Digital Surface Models, Textured Digital Surface Models, and Orthorectified Imagery.

Users' mailing list: http://lists.osgeo.org/cgi-bin/mailman/listinfo/opendronemap-users

Developer's mailing list: http://lists.osgeo.org/cgi-bin/mailman/listinfo/opendronemap-dev

Overview video: https://www.youtube.com/watch?v=0UctfoeNB_Y

Developers
=================

Help improve our software!

1. Join our [Gitter](https://gitter.im/OpenDroneMap)
2. Try to keep commits clean and simple
3. Submit a pull request with detailed changes and test results

Steps to get OpenDroneMap running:
==================================

(Requires Ubuntu 12.04 or later, see https://github.com/OpenDroneMap/odm_vagrant for running on Windows in a VM)

To get ODM installed in 12.04, follow these steps:
```
# Cmake
wget https://cmake.org/files/v3.4/cmake-3.4.0.tar.gz
tar xvf cmake-3.4.0.tar.gz
cd cmake-3.4.0
./bootstrap && make && make install

# gflags:
wget https://github.com/schuhschuh/gflags/archive/master.zip
unzip master.zip
cd gflags-master
mkdir build && cd build
export CXXFLAGS="-fPIC"
cmake .. -DGFLAGS_NAMESPACE=google
make 
sudo make install

# google glog:

https://github.com/google/glog/archive/v0.3.4.tar.gz
tar xvf v0.3.4.tar.gz
cd glog-0.3.4
configure
make
sudo make install

# suitesparse:

wget http://faculty.cse.tamu.edu/davis/SuiteSparse/SuiteSparse-4.4.5.tar.gz
tar xvf SuiteSparse-4.4.5.tar.gz
cd SuiteSparse/
make library
sudo make install

# Eigen 3:
wget http://bitbucket.org/eigen/eigen/get/3.2.7.tar.gz
tar xvf 3.2.7.tar.gz
cd eigen-eigen-b30b87236a1b/
mkdir build && cd build
cmake ..
sudo make install

# Other dependencies
sudo apt-get install libboost1.48-all-dev
```

Run install.sh to build.

    ./install.sh

From a directory full of your images, run

    ./run.pl

An overview of installing and running OpenDroneMap on Ubuntu can be found here: https://www.youtube.com/watch?v=e2qp3o8caPs

Here are some other videos:

- https://www.youtube.com/watch?v=7ZTufQkODLs (2015-01-30)
- https://www.youtube.com/watch?v=m0i4GQdfl8A (2015-03-15)

Now that texturing is in the code base, you can access the full textured meshes using MeshLab. Open MeshLab, choose `File:Import Mesh` and choose your textured mesh from a location similar to the following: `reconstruction-with-image-size-1200-results\odm_texturing\odm_textured_model.obj`

---

Alternatively, you can also run OpenDroneMap in a Docker container:

    export IMAGES=/absolute/path/to/your/images
    docker build -t opendronemap:latest .
    docker run -v $IMAGES:/images opendronemap:latest

To pass in custom parameters to the `run.pl` script, simply pass it as arguments to the `docker run` command.

---

Example data can be found at https://github.com/OpenDroneMap/odm_data

---

Long term, the aim is for the toolchain to also be able to optionally push to a variety of online data repositories, pushing hi-resolution aerials to [OpenAerialMap](http://opentopography.org/), point clouds to [OpenTopography](http://opentopography.org/), and pushing digital elevation models to an emerging global repository (yet to be named...). That leaves only digital surface model meshes and UV textured meshes with no global repository home.

---


Documentation:
==============

For documentation, please take a look at our [wiki](https://github.com/OpenDroneMap/OpenDroneMap/wiki).


Troubleshooting:
================

Make sure you have enough RAM and CPU. Only lowercase file extension supported now.

If you run ODM with your own camera, it is possible you will see something like this:

```
  - configuration:
    --cmvs-maxImages: 500
    --end-with: pmvs
    --match-size: 200
    --matcher-ratio: 0.6
    --matcher-threshold: 2
    --pmvs-csize: 2
    --pmvs-level: 1
    --pmvs-minImageNum: 3
    --pmvs-threshold: 0.7
    --pmvs-wsize: 7
    --resize-to: 1200
    --start-with: resize


  - source files - Fri Sep 19 13:47:42 UTC 2014


    no CCD width or focal length found for DSC05391.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05392.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05393.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05394.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05395.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05396.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05397.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05398.JPG - camera: "SONY DSC-HX5V"
    no CCD width or focal length found for DSC05399.JPG - camera: "SONY DSC-HX5V"

    found no usable images - quitting
Died at ../../OpenDroneMap/./run.pl line 364.


```

This means that your camera is not in the database, https://github.com/OpenDroneMap/OpenDroneMap/blob/gh-pages/ccd_defs.json

This problem is easily remedied. We need to know CCD size in the camera. We'll get these for our Sony Cyber-shot DSC-HX5 from dpreview: http://www.dpreview.com/products/sony/compacts/sony_dschx5/specifications

So, we'll add the following line to our ccd_defs.json:

     "SONY DSC-HX5V": 6.104,

To check that ccd_defs.json compiles, run ccd_defs_check.pl
If it prints the message 'CCD_DEFS compiles OK', then you can commit your changes.

And so others can use it, we'll do a pull request to add it to our array for everyone else.

---

Maintainers can run the ccd_defs.json compilation test automatically by creating a
symbolic link in .git/hooks to hooks/pre-commit

     cd .git/hooks
     ln -s ../../hooks/pre-commit

If ccd_defs.json does not compile, then the pre-commit hook will abort the commit.
