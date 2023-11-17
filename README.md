# OpenCV-CUDA
How to copile OPENCV to use CUDA within a DOCKER image

In order to compile opencv to use cuda we need to understand first how do C++ build process works:

### Build Process (create an executable file)
- **Preprocessing**: The preprocessor does many things: macro replacements, check for conditional compilation, insert contet whenever a header file is being included. Only source files are passed to the preprocessor.
	- *input*: C++ source code
  	- *output*: file.ii
- **Compiler (GCC: GNU COMPILER COLLECTION)**: Transforms the high level programming language to a low level one.
	- *input*: file.ii (preprocessed file)
	- *output (low level language)*: assembly code/file --> file.s
- **Assembler**: Machine code representation of your source code.
  	- *input*: assembly code/file --> file.s
  	- *output*: object code --> file.obj / file.o
- Linker
  input: object code --> file.obj / file.o
  output: file.exe 
  Link the Standard and External libraries and generate the executable file.

## Generate Docker image with compiled OpenCV to use CUDA
Now, we can proceed to compile the openCV source code in order for it to make some processes directly in a GPU.

1. You need to create a folder to clone de opencv and opencv-contrib repositories there (if their latest versions are too recent, I would suggest to try with the second to last)
2. Find out which nvidia docker image you have to use. In order to do that you have to check which GPU you have.
   - Here [CUDA-GPUS](https://developer.nvidia.com/cuda-gpus) you can find the compute capability of the card.
   - Here [CUDA TOOLKIT, CUDA DRIVER AND CUDNN](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html#:~:text=For%20best%20performance%2C%20the%20recommended,was%20used%20for%20tuning%20heuristics.) you can find the supported versions regarding the GPU card.
3. Go to [Docker hub](https://hub.docker.com/r/nvidia/cuda/tags) and choose a docker image matching your GPU requirements.
   - Be aware that there are [types of images](https://hub.docker.com/r/nvidia/cuda), choose one that satisfies your project requirements.
5. Create a docker container based on the nvidia docker image you selected
   - docker run --net=host --runtime=nvidia -it -v <path to folder in step 1>:/main_dir  <docker image> /bin/bash (Exr: nvidia/cuda:12.0.0-cudnn8-devel-ubuntu20.04) (Preferably use a docker image with CUDNN, otherwise you shall install it manually)
6. Once the container is running you have to install:
   - Python (Keep in mind to choose the python version you need):
	- apt-get update && apt-get install -y software-properties-common && rm -rf /var/lib/apt/lists/ && add-apt repository ppa:deadsnakes/ppa
	- apt-get install -y python3.10 python3.10-dev libopencv-dev
	- alias python=python3.10 && alias python3=python3.10
	- update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
	- update-alternatives --config python3
   - Build Opencv:
        - Go to the opencv folder (cd path/to/opencv/folder)
        - Run *mkdir build*, then *cd build*
        - Run *cmake*
        	- Change the card arquitecture CUDA_ARCH_BIN (compute capability from step 2)
         	- Change the python folder path PYTHON_EXECUTABLE
          	- Change the cuda folder path CUDA_TOOLKIT_ROOT_DIR
          	- Change the opencv contrib modules OPENCV_EXTRA_MODULES_PATH
          	- Finally there are some parameters that can be modified to your convenience. Please check the [docs](https://docs.opencv.org/4.x/db/d05/tutorial_config_reference.html) to analyze what is best for your project.
          Example of a cmake command:
cmake -D CMAKE_BUILD_TYPE=Release -D BUILD_PNG=OFF -D BUILD_TIFF=OFF -D BUILD_TBB=OFF -D BUILD_JPEG=OFF -D BUILD_JASPER=OFF -D BUILD_ZLIB=OFF -D BUILD_EXAMPLES=OFF -D BUILD_opencv_java=OFF -D BUILD_opencv_python2=OFF -D BUILD_opencv_python3=ON -D ENABLE_NEON=ON -D WITH_OPENCL=OFF -D WITH_OPENMP=OFF -D WITH_FFMPEG=ON -D WITH_GSTREAMER=OFF -D WITH_GSTREAMER_0_10=OFF -D WITH_CUDA=ON -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda -D WITH_GTK=ON -D WITH_VTK=OFF -D WITH_TBB=ON -D WITH_1394=OFF -D WITH_OPENEXR=OFF -D CUDA_ARCH_BIN=7.5 -D CUDA_ARCH_PTX="" -D INSTALL_C_EXAMPLES=OFF -D INSTALL_TESTS=OFF -D WITH_CUDNN=ON -D OPENCV_DNN_CUDA=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D PYTHON_EXECUTABLE=/usr/bin/python3 -D WITH_CUBLAS=1 -D OPENCV_EXTRA_MODULES_PATH=/main/opencv_contrib/modules ..
	- Run *make install*
7. If everything went well now you should have a docker image with cuda, cudnn and opencv ready to use cuda! Now, you can exit the container without killing it (Ctrl+p, Ctrl+q) and save a docker image based on that container *docker commit  -m "cudaxx-cudnxx-opencvx.x" <container id>  cudaxx-cudnnxx-opencvx.x:1*
8. Finally you can kill the running container and start using the docker image for your projects. (Keep in mind that this process can also be summarized in a Dockerfile, but it is always good to understand step by step what is happening during the building process)
