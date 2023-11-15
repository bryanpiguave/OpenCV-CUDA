# OpenCV-CUDA
How to copile OPENCV to use CUDA within a DOCKER image

In order to compile opencv to use cuda we need to understand first how do C++ build process works:

### Build Process (create an executable file)
- Preprocessing
  input: C++ source code
  output: file.ii
  The preprocessor does many things: macro replacements, check for conditional compilation, insert contet whenever a header file is being included. Only source files are passed to the preprocessor.
- Compiler (GCC: GNU COMPILER COLLECTION)
  input: file.ii (preprocessed file)
  output (low level language): assembly code/file --> file.s
  Transforms the high level programming language to a low level one.
- Assembler
  input: assembly code/file --> file.s
  output: object code --> file.obj / file.o
  Machine code representation of your source code.
- Linker
  input: object code --> file.obj / file.o
  output: file.exe 
  Link the Standard and External libraries and generate the executable file.

## Generate Docker image with compiled OpenCV to use CUDA
Now, we can proceed to compile the openCV source code in order for it to make some processes directly in a GPU.

1. You need to create a folder and there clone de opencv and opencv-contrib repositories (if their latest versions are too recent, I would suggest to try with the second to last)
2. Find out which nvidia docker image you have to use. In order to do that you have to check which GPU you have
3. Create a docker container based on the nvidia docker image you need
- Crear una carpeta y ahí clonar repo de opencv y opencv-contrib (intenta con la última versión, si es muy reciente intenta con la penúltima para evitar bugs).
- docker run --net=host --runtime=nvidia -it -v <directorio creado en el priemr paso>:/main_dir  <nombre de la imagen de docker> /bin/bash (Ejemplo docker: nvidia/cuda:12.0.0-cudnn8-devel-ubuntu20.04) (De preferencia usa una imagen con CUDNN ya instalado, sino encuentras usa una con CUDA e instalale manualmente CUDNN)
- INSTALAR PYTHON 
	- apt-get update && apt-get install -y software-properties-common && rm -rf /var/lib/apt/lists/ && add-apt repository ppa:deadsnakes/ppa
	- apt-get install -y python3.10 python3.10-dev libopencv-dev
	- alias python=python3.10 && alias python3=python3.10
	- update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
	- update-alternatives --config python3
- cd a la carpeta opencv, luego mkdir build y cd build
-  HACER EL CMAKE (Cambiar la arquietectura de la tarjeta, el path de python, y el path de cuda)
cmake -D CMAKE_BUILD_TYPE=Release -D BUILD_PNG=OFF -D BUILD_TIFF=OFF -D BUILD_TBB=OFF -D BUILD_JPEG=OFF -D BUILD_JASPER=OFF -D BUILD_ZLIB=OFF -D BUILD_EXAMPLES=OFF -D BUILD_opencv_java=OFF -D BUILD_opencv_python2=OFF -D BUILD_opencv_python3=ON -D ENABLE_NEON=ON -D WITH_OPENCL=OFF -D WITH_OPENMP=OFF -D WITH_FFMPEG=ON -D WITH_GSTREAMER=OFF -D WITH_GSTREAMER_0_10=OFF -D WITH_CUDA=ON -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda -D WITH_GTK=ON -D WITH_VTK=OFF -D WITH_TBB=ON -D WITH_1394=OFF -D WITH_OPENEXR=OFF -D CUDA_ARCH_BIN=7.5 -D CUDA_ARCH_PTX="" -D INSTALL_C_EXAMPLES=OFF -D INSTALL_TESTS=OFF -D WITH_CUDNN=ON -D OPENCV_DNN_CUDA=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D PYTHON_EXECUTABLE=/usr/bin/python3 -D WITH_CUBLAS=1 -D OPENCV_EXTRA_MODULES_PATH=/test_yolonas/opencv_contrib/modules ..
- make install
