# MPI4Py installation instructions for Ubuntu Trusty.

The instructions are to install MPI4Py by building from source. MPI4Py requires OpenMPI or similar <b>mpicc</b> compiler wrapper. So let us first install OpenMPI. 
For that, download the installation files first.

    wget https://www.open-mpi.org/software/ompi/v1.10/downloads/openmpi-1.10.2.tar.bz2

Extract the file and open it.

    tar -zxf openmpi-1.10.2.tar.gz
    cd openmpi-1.10.2/

Configure the installation files by running the following code:

    ./configure --prefix="/home/$USER/.openmpi"

Now, we can make and install the files.

    sudo make -j4
    sudo make install

If there are no errors, OpenMPI is successfully installed. 
To get the env paths linked correctly, include the following in the .bashrc file.

    export PATH="$PATH:/home/$USER/.openmpi/bin"
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/$USER/.openmpi/lib/"

Confirm that OpenMPI is successfully installed run:
    mpiexec -V

This will print you the version of MPI installed.
Good job if you got that right !

Now let us install MPI4Py.

Download the installation files.

    wget https://bitbucket.org/mpi4py/mpi4py/downloads/mpi4py-2.0.0.tar.gz

Extract the file and open it.

    tar -zxf mpi4py-2.0.0.tar.gz
    cd mpi4py-2.0.0/

Now we need to build and install the files. For this run these:

    sudo python setup.py build
    sudo python setup.py install

If the build and install process went well, you have MPI4Py successfully installed on your system.
Now let us confirm that. Run (in the mpi4py-2.0.0 folder):

    mpiexec -n 5 python demo/helloworld.py

If the installation went well, you should see the <i>Hello, World!</i> output from 5 different nodes.

    
