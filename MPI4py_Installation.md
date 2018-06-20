# OpenMPI and MPI4py installation instructions for Ubuntu 16.04.

The instructions are to install MPI4Py by building from source. MPI4Py requires OpenMPI or similar <b>mpicc</b> compiler wrapper. So let us first install OpenMPI. 
For that, download the installation files first.

    wget https://download.open-mpi.org/release/open-mpi/v3.1/openmpi-3.1.0.tar.gz

Extract the file and open it.

    tar -zxf openmpi-3.1.0.tar.gz
    cd openmpi-3.1.0/

Configure the installation files by running the following code:

    ./configure --prefix="/home/$USER/.openmpi"

Now, we can make and install the files.

    sudo make -j4
    sudo make install

If there are no errors, OpenMPI is successfully installed. 
To get the env paths linked correctly, include the following in the `.bashrc` file.

    export PATH="$PATH:/home/$USER/.openmpi/bin"
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/$USER/.openmpi/lib/"
    
Run `source ~/.bashrc` to make the changes available in current session.

Confirm that OpenMPI is successfully installed run:
    
    mpiexec -V

This will print you the version of MPI installed.
Good job if you got that right !

## Install MPI4Py.

### Requirements
    
    sudo apt-get install libopenmpi-dev python-dev python3-dev
    
Download the installation files.

    wget https://bitbucket.org/mpi4py/mpi4py/downloads/mpi4py-3.0.0.tar.gz

Extract the file and open it.

    tar -zxf mpi4py-3.0.0.tar.gz
    cd mpi4py-3.0.0/

Now we need to build and install the files. For this run these:

    sudo python setup.py build
    sudo python setup.py install

or use `python3` if required.

    sudo python3 setup.py build
    sudo python3 setup.py install


If the build and install process went well, you have MPI4Py successfully installed on your system.
Now let us confirm that. Run (in the mpi4py-3.0.0 folder):

    mpiexec -n 5 python demo/helloworld.py

If the installation went well, you should see the <i>Hello, World!</i> output from 5 different nodes.

    
