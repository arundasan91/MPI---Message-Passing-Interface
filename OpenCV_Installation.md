#OpenCV Installation Instructions for Ubuntu Trusty

Extracted from [OpenCV - Community Help Wiki](https://help.ubuntu.com/community/OpenCV)

First let us install the dependencies. Building OpenCV can be challenging at first, but if you have all the dependencies correct it will be done in no time.

Go ahead and run the following lines:

    sudo apt-get install build-essential

The 'build-essential' ensures that we have the compilers ready. 
Now we will install some required packages. Run:

    sudo apt-get -qq remove ffmpeg x264 libx264-dev
    sudo apt-get -qq install libopencv-dev checkinstall cmake pkg-config yasm libjpeg-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libdc1394-22-dev libxine-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libv4l-dev python-dev python-numpy libtbb-dev libqt4-dev libgtk2.0-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev x264 v4l-utils ffmpeg cmake qt5-default checkinstall

Download the files:

    wget http://sourceforge.net/projects/opencvlibrary/files/opencv-unix/2.4.9/opencv-2.4.9.zip

Unzip the file by:

    unzip opencv-2.4.9.zip

Go to the opencv folder and create a directory called build:

    cd opencv-2.4.9
    mkdir build

Go inside the build directory and build the files using cmake.

    cd build
    cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D WITH_QT=ON -D WITH_OPENGL=ON -D WITH_VTK=ON ..

In the summary seen in the terminal, make sure that FFMPEG is installed, also check whether the Python, Numpy, Java and OpenCL are properly installed and recognized.

Now we will run the make process as 4 jobs by specifying it like <b>-j4</b>. More on it [here](http://www.tutorialspoint.com/unix_commands/make.htm " make command")

    sudo make -j4

Go ahead and continue installation.

    sudo make install

Once the installation is complete, do these steps to get OpenCV configured.

    sudo sh -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'

    sudo ldconfig

    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/opencv/lib
