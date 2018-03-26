OpenDetection
=============

OpenDetection is a standalone open source project for object detection and recognition in images and 3D point clouds.

Website - http://opendetection.com or http://krips89.github.io/opendetection_docs

Git - https://github.com/krips89/opendetection

Getting started - http://opendetection.com/getting_started.html

User guide - http://opendetection.com/introduction_general.html

API Documentation - http://opendetection.com/annotated.html

Short demo video - https://www.youtube.com/watch?v=sp8W0NspY54


# Opendetection Installation Ubuntu 16.04

**Authored by Joseph Polden.**

Opendetection is a great library, however installation is not so straight forward. What follows is a short summary of the steps taken to install the opendection package on my laptop:
- Dell Precision 5520
- Ubuntu 16.04
- Nvidia Quadro M1200 Graphics card
- Nvidia Binary Driver version 384.111

# Step 1: Cuda. 

Cuda is required. I installed Cuda 9, based off the method [outlined here]. There are a few changes made since the original guide is a couple of years old.

Download the relevant CUDA .run file from the [NVIDIA download site]. I used cuda version 9.1.85_387.26_linux, which can be found at <https://developer.nvidia.com/compute/cuda/9.1/Prod/local_installers/cuda_9.1.85_387.26_linux>

Once downloaded, use the terminal to navigate to the downloaded file and run it: 

    cd path/to/cuda_9.1.85_387.26_linux.run/
    chmod 755 cuda_9.1.85_387.26_linux.run 
    sudo ./cuda_9.1.85_387.26_linux.run --override

I made use of the "--override" suffix to bypass some initial errors I encountered (it may not be neccesary on other systems). Before installation, a script will prompt you with a few basic installation questions. Everything is fairly standard, however I did **NOT** elect to install the provided NVIDIA driver since the one currently installed on my computer is a more recent version. Below are the settings I used in my installation. After these prompts, the installation ran smoothly.
 
    Do you accept the previously read EULA? (accept/decline/quit): accept
    You are attempting to install on an unsupported configuration. Do you wish to continue? ((y)es/(n)o) [ default is no ]: y
    Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 387.26? (y)es/(n)o/(q)uit: n
    Install the CUDA 9.1 Toolkit? (y)es/(n)o/(q)uit: y
    Enter Toolkit Location [ default is /usr/local/cuda-9.1 ]: 
    Do you want to install a symbolic link at /usr/local/cuda? (y)es/(n)o/(q)uit: y
    Install the CUDA 9.1 Samples? (y)es/(n)o/(q)uit: y
    Enter CUDA Samples Location [ default is /home/rosi ]: /usr/local/cuda-9.1
    Installing the CUDA Toolkit in /usr/local/cuda-9.1 ...
    Installing the CUDA Samples in /usr/local/cuda-9.1 ...
    Copying samples to /usr/local/cuda-9.1/NVIDIA_CUDA-9.1_Samples now...
    Finished copying samples. 

# 2. Step 2: Openni
Openni is a required to properly configure the required PCL installation (step 3, below). I used steps 1 and 2 of [the following guide] (i did not install the kinect modules etc, for now)  
  
First, I installed some dependencies

    sudo apt-get update
    sudo apt-get install openjdk-9-jdk  
    sudo apt-get install doxygen graphviz mono-complete 
 
Then we cloned the openNI package, and checked out the Unstable branch (it is newer than the master). 

    git clone https://github.com/OpenNI/OpenNI.git
    cd OpenNI/
    git checkout Unstable-1.5.4.0 

Navigate to the RedistMaker file and run it. It will compile the library for you:

    cd Platform/Linux/CreateRedist/
    chmod +x RedistMaker
    ./RedistMaker

After this, navigate to the Redist directory and run the install script to install the software on your system.
  
    cd Redist/OpenNI-Bin-Dev-Linux-x64-v1.5.4.0/Bin/x64-Release/
    sudo ./install.sh 

# Step 3: PCL 1.8.0

Opendetection requires PCL 1.8 with '3d_rec_framework' enabled. To build with this setting you need to install OpenNI as well, which is the mandatory dependency for this app (refer to step 2 above).

Download the PCL 1.8.1 zip file [from here], and unpack it in your home directory. I used cmake-gui to configure the library.

    cd /path/to/unzipped/pcl-pcl-1.8.1
    mkdir build
    cd build
    cmake-gui

Once you have specified the source and build directories in the GUI interface, click the configure button and you should be able to select the following additional items:
- BUILD_apps
- BUILD_apps_3d_rec_framework
- BUILD_apps_cloud_composer
- BUILD_apps_in_hand_scanner
- BUILD_apps_modeler
- BUILD_apps_optronic_viewer
- BUILD_apps_point_cloud_editor

Not all of these additions are required dependencies, however I installed them all regardless. Click generate to build the makefile and once finished, exit the GUI. We can now make and install the library:

    make - j8
    sudo make install

#  Step 4. OpenCV 3.4.0

I used [this guide] as the basis to install openCV. First, some initial packages were installed (your system will most likely already have them)

    sudo apt-get install build-essential
    sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
    sudo apt install checkinstall auto-apt ccache

Now we can get the sourced. Download the openCV-3.4.0 and opencv_contrib tarballs (or you optionally clone them from source).
- [opencv-3.4.0.tar.gz]
- [opencv_contrib-3.4.0.tar.gz]

Unzip them in your home diirectory (or wherever you wish to keep them). Navigate to the created opencv-3.4.0 directory and create a build folder

    cd /path/to/opencv-3.4.0/
    mkdir build
    cd build
from here we use cmake-gui to configure the makefile

    cmake-gui
    
click the configure button in the GUI, the following settings were used:
- CMAKE_BUILD_TYPE=RELEASE 
- CMAKE_INSTALL_PREFIX=/usr/local
- OPENCV_EXTRA_MODULES_PATH=/path/to/opencv_contrib-3.4.0/modules  
- WITH_V4L=ON 
- WITH_VTK=ON 
- INSTALL_C_EXAMPLES=ON 
- INSTALL_PYTHON_EXAMPLES=ON 
- BUILD_EXAMPLES=ON 
- ENABLE_FAST_MATH=ON  
- WITH_CUBLAS=ON 
- WITH_OPENGL=ON 
- WITH_CUDA=ON
- CUDA_ARCH_PTX="6.1"

Click generate to build the makefile and once finished, exit the GUI. The final step is to make and install the library:

    make - j8
    sudo make install

Upon sucsessful installation, an openCV config file was then created:

    sudo /bin/bash -c 'echo "/usr/local/lib"> /etc/ld.so.conf.d/opencv.conf'
    sudo ldconfig


# Step 5. Opendetection

Now that the dependancies have been installed, we can move on to opendection. I used the same method provided in the [official documentation], with some small alterations made.

First, clone the opendetection source into your home directory and create a build folder.

    cd <path_to_desired_download_location>
    git clone https://github.com/krips89/opendetection.git
    mkdir build; cd build
    
from here, we use cmake-gui to update the OpenCV_DIR variable. I have ROS installed on my computer, which has its own copy of openCV packaged with it. I need to update the OpenCV_DIR varialble to point to the openCV installation done in step 4 (initially it was pointing to the openCV copy packaged with ROS). To do this, launch the cmake-gui from the build directory, click the configure button and adjust the following variable to:
- OpenCV_DIR=/path/to/opencv-3.4.0/build
    
Click generate to build the makefile and once finished, exit the GUI. Before we compile and install, we need to adjust some code in the ODFaceRecognizer.cpp file, which does not compile. Open the file in a text editor and make change the following lines:  

Line 44:   

    from: cvrecognizer_ = cv::face::createFisherFaceRecognizer(num_components_, threshold_); 
    to:   cvrecognizer_ = cv::face::FisherFaceRecognizer::create(num_components_, threshold_); 
 
Line 48:  
    
    from: cvrecognizer_ = cv::face::createEigenFaceRecognizer(num_components_, threshold_);
    to:   cvrecognizer_ = cv::face::EigenFaceRecognizer::create(num_components_, threshold_);

Line 77:  
    
    from: cvrecognizer_->load(files[0]);
    to:   cvrecognizer_->read(files[0]);

Once the updates were made, opendetection library was able to compile. Navigate to the opendetection build folder created earlier and run:

    make -j8
    sudo make install




   [outlined here]: <https://www.pugetsystems.com/labs/hpc/NVIDIA-CUDA-with-Ubuntu-16-04-beta-on-a-laptop-if-you-just-cannot-wait-775/>
   [NVIDIA download site]: <https://developer.nvidia.com/cuda-downloads>
   [the following guide]: <https://www.20papercups.net/programming/kinect-on-ubuntu-with-openni/> 
   [from here]: <https://github.com/PointCloudLibrary/pcl/archive/pcl-1.8.1.zip>
   [this guide]: <http://demura.net/misc/14118.html>
   [opencv-3.4.0.tar.gz]: <https://github.com/opencv/opencv/archive/3.4.0.tar.gz>
   [opencv_contrib-3.4.0.tar.gz]: <https://github.com/opencv/opencv_contrib/archive/3.4.0.tar.gz>
   [official documentation]: <http://opendetection.com/installation_instruction.html>
   
   