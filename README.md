# pyopenvdb-build-guide
A brief description of how to build the pyopenvdb module


Build module pyopenvdb (version 3.2.0) on linux fedora 21 for use with python 3.5:

Maybe some of this steps are useless but i wasnt able to reproduce some of the errors.
They can of course be handeled in a much more sophisticated manner.
(I'm new to linux ;-))

General Problems:
python.boost library has to be built against python 3!
That caused a lot of trouble for me.

1. Clone the openvdb-files from github using the console command:
git clone https://github.com/dreamworksanimation/openvdb.git

2. install python 3.5 coming with anaconda from this link
https://www.continuum.io/downloads

3. The important point with this is that boost.python has to be built 
against python 3.5 and not 2.7 or something else to work.
For this reason i downloaded the boost libraries from this link:
http://www.boost.org/users/history/version_1_61_0.html

At first set the CPLUS_INCLUDE_PATH like the one described here:
http://stackoverflow.com/questions/19810940/ubuntu-linking-boost-python-fatal-error-pyconfig-cannot-be-found

export CPLUS_INCLUDE_PATH="$CPLUS_INCLUDE_PATH:~/anaconda3/include/python3.5m/"
(this path contains the files pyconfig.h and Python.h, which are both possible reasons for the
boost.python lib to fail building.)

with help of this build boost like this
http://stackoverflow.com/questions/25188861/libboost-python3-so-1-56-0-undefined-symbol-pyclass-type 

which tells you how to build boost against python 3 (maybe configure the project-config.jam data on your own:
using python : 3.5 : /~/anaconda3 ;

$ ./bootstrap.sh
here you have to configure the project-config.jam file and then run

$ ./b2 --with-python --clean
$ ./b2 --with-python --buildid=3

the libboost_python-3.so should be directly in the boost/stage/lib folder

make sure to tell the makefile the path to your libboost_python which was built against python3!

4. Configure the paths in the openvdb/openvdb/makefile depending on your system configuration
i left the log4cplus stuff blank as it threw a lot of errors
check whether you have installed the asked dependency or not
check out the INSTALL FILE for details

important: provide all the python paths like this
# The version of Python for which to build the OpenVDB module
# (leave blank if Python is unavailable)
PYTHON_VERSION := 3.5m
# The directory containing Python.h
#PYTHON_INCL_DIR := $(HFS)/include/python$(PYTHON_VERSION)
PYTHON_INCL_DIR := /home/lgartmair/anaconda3/include
# The directory containing pyconfig.h
PYCONFIG_INCL_DIR := /home/lgartmair/anaconda3/include/python3.5m
# The directory containing libpython
PYTHON_LIB_DIR := /home/lgartmair/anaconda3/lib
PYTHON_LIB := -lpython3.5m
# The directory containing libboost_python
BOOST_PYTHON_LIB_DIR := /home/lgartmair/Downloads/boost_1_61_0/stage/lib
BOOST_PYTHON_LIB := -lboost_python-3
# The directory containing arrayobject.h
# (leave blank if NumPy is unavailable)
#NUMPY_INCL_DIR := /home/lgartmair/anaconda3/lib/python3.5/site-packages/numpy/core/#include/numpy
NUMPY_INCL_DIR := /usr/include/numpy

5. Then in order to prevent a bunch of errors coming from the openvdb/python/pyOpenVDBModule.cc file some lines have to be changed in this file.
5.1 add the following function after the include steps and before namespace py = boost::python; :
based on this link: http://numpy-discussion.10968.n7.nabble.com/How-to-call-import-array-properly-td1383.html

(ca. line 50)

int init_numpy()
{
	import_array();
}

and replace this code (ca. line 600)

#ifdef PY_OPENVDB_USE_NUMPY
    // Initialize NumPy.
    import_array();
#endif

with the new one:

#ifdef PY_OPENVDB_USE_NUMPY
    // Initialize NumPy.
    init_numpy;
#endif

now the error regarding a return type void/NULL instead of int which is required should be solved!

i don't know if this was neccessary, but 
i also replaced this section here  (ca. line 580/590)

//#ifdef DWA_OPENVDB
//#define PY_OPENVDB_MODULE_NAME  _openvdb
//#else
//#define PY_OPENVDB_MODULE_NAME  pyopenvdb
//#endif

simply by this one

#define PY_OPENVDB_MODULE_NAME  pyopenvdb

in order for the following line BOOST_PYTHON_MODULE(PY_OPENVDB_MODULE_NAME)
to call the correct module name in any case.

6. The actual build step is as follwos:
Navigate to the folder containing the openvdb make file and type in the command line:
$ -make clean
$ -make python

and hopefully there is a pyopenvdb.so (together with a libopenvdb.so.3.2.0)  library in your folder.
To prevent the error libopenvdb.so - no such file or directory! when trying to import pyopenvdb
type the following commands to create symlinks to the libopenvdb file (while you are still in the openvdb/openvdb folder)

$ ln -s libopenvdb.so.3.2.0 libopenvdb.so.3.2
$ ln -s libopenvdb.so.3.2.0 libopenvdb.so

as i don't know where to put these files to make it visible for python i just added the openvdb folder to my path

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/openvdb/openvdb

now you can hopefully call pyopenvdb with import pyopenvdb as vdb in your python ide/console.

Regards, Lukas
