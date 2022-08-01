# Jetson Nano DevKit Blazepose GPU

Amaan Rahman 



## Setup

1. Install latest versions of `gcc` and `g++`, this will ensure all C++ scripts are compiled appropriately in Mediapipe during the build process
   
   ```bash
   sudo add-apt-repository ppa:ubuntu-toolchain-r/test
   sudo apt-get update && upgrade -y
   sudo apt-get install gcc-9 g++-9
   sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-9
   
   ```
   
   <u><em>Note</em></u>: If you already executed `bazel` to build or run Mediapipe examples prior to upgrading `gcc` and `g++`, then make sure to clear the `bazel` cache like so: 
   ```
   rm -rf ~/.cache/bazel
   ```
2.  Install OpenGL ES packages
   
   ```bash
   sudo apt-get install mesa-common-dev libegl1-mesa-dev libgles2-mesa-dev mesa-utils
   ```

3. Check OpenGL ES version, make sure it is >=3.2
   
   ```bash
   glxinfo | grep -i opengl
   ```

4. Download Bazelisk via `npm` and Mediapipe via `git` 
   
   ```bash
   cd ~                            # go to home directory
   sudo apt-get install npm git    # install if you don't have
   npm install -g @bazel/bazelisk  # if this doesn't work, try with sudo
   git clone https://github.com/google/mediapipe.git
   cd mediapipe
   ```

   <u><em>Note</em></u>: Make sure to be in the root directory of `mediapipe` from here on out, unless                    specified otherwise.

4. Install OpenCV and FFmpeg 
   
   ```bash
   sudo apt-get install -y \
       libopencv-core-dev \
       libopencv-highgui-dev \
       libopencv-calib3d-dev \
       libopencv-features2d-dev \
       libopencv-imgproc-dev \
       libopencv-video-dev
   ```

   <u>*Note*</u>: Don't attempt to build OpenCV from source, there is a high possibility that                   the Jetson Nano may crash

5. Check in the `mediapipe` root directory that there is a soft link directory called `bazel-bin`, if not execute the following commands to check and instantiate the soft link: 
   
   ```bash
   bazel info bazel-bin    # check if bazel-bin exists, the path will be printed
   bazel build             # do this step, if it doesn't
   ```

6. Build Blazepose with GPU configuration
   
   ```bash
   bazel build --copt -DMESA_EGL_NO_X11_HEADERS --copt -DEGL_NOX11 mediapipe/examples/desktop/pose_tracking:pose_tracking_gpu
   ```

7. Run Blazepose on GPU
   
   ```bash
   GLOG_logtostderr=1 bazel-bin/mediapipe/examples/desktop/pose_tracking/pose_tracking_gpu --calculator_graph_config_file=mediapipe/graphs/pose_tracking/pose_tracking_gpus.pbtxt
   ```

## Miscellaneous Steps

Perform these steps if you have encountered CUDA resolution errors. One way to perform a test is executing the following command: 

```bash
nvcc --version
```

To fix such errors, add these lines to your `~/.bashrc`:

```bash
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64/${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

Check if `/usr/local/cuda/extras/CUPTI/lib64` exists. If not, then skip this step (check **Appendix** if you are interested in the why). 

If it does exist, then `LD_LIBRARY_PATH` will be exported in the `~/.bashrc` file like so:

```bash
export LD_LIBRARY_PATH=/usr/local/cuda/extras/CUPTI/lib64,/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

After `~/.bashrc` file has been appropriately modified, execute this command to make sure the environment variables `PATH` and `LD_LIBRARY_PATH` are configured in your system:

```bash
exec $SHELL
```

## Appendix

It may be possible that `/usr/local/cuda/extras/CUPTI/lib64` may not exist. This was certainly the case for me in my Jetson Nano DevKit. `CUPTI` package is normally installed as a part of the CUDA Toolkit. To check if CUDA Toolkit is configured in your system:

```bash
nvcc --version
```

If command is not found, then CUDA Toolkit may not have been installed properly or CUDA path has not been configured in your system properly (check **Miscellaneous Steps**). Usually, it is installed in the Jetson Nano via the Jetpack module. 

The expected files in `/usr/local/cuda/extras/CUPTI/lib64` are `libcupti` and `libnvrtc` shared object (`.so`) and archive (`.a`) type files. Do not fret if this directory doesn't exist because that doesn't necessarily mean that the aforementioned files are missing. You can locate these files in `/usr/local/cuda/lib64`. Thus, don't worry about the path `/usr/local/cuda/extras/CUPTI/lib64`, and no need to add that to your environment variable `LD_LIBRARY_PATH`. 

## References

[Installation - mediapipe (google.github.io)](https://google.github.io/mediapipe/getting_started/install.html)

[GPU Support - mediapipe (google.github.io)](https://google.github.io/mediapipe/getting_started/gpu_support.html)

[MediaPipe in C++ - mediapipe (google.github.io)](https://google.github.io/mediapipe/getting_started/cpp.html)

[Pose - mediapipe (google.github.io)](https://google.github.io/mediapipe/solutions/pose.html#desktop)


