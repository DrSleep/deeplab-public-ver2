## Standalone Dockerfiles

The `standalone` subfolder contains a docker file to correctly install GPU executable image for [deeplab-public-ver2](https://bitbucket.org/aquariusjay/deeplab-public-ver2).
The image can be built by running:

```
nvidia-docker build -t caffe_dl:gpu standalone/dl_gpu
```

The image requires a CUDA 7.5 capable driver to be installed on the system and [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) for running the Docker containers.

For more information on how to run Caffe using the docker image please refer to the [Caffe guide](https://github.com/BVLC/caffe/blob/master/docker/README.md).

# Running Caffe using the docker image

In order to test the Caffe image, run:
```
nvidia-docker run -ti caffe_dl:gpu caffe --version
```
which should show a message like:
```
libdc1394 error: Failed to initialize libdc1394
caffe version 1.0.0-rc3
```

One can also build and run the Caffe tests in the image using:
```
nvidia-docker run -ti caffe_dl:gpu bash -c "cd /opt/caffe/build; make runtest"
```

In order to get the most out of the caffe image, some more advanced `docker run` options could be used. For example, running:
```
nvidia-docker run -ti --volume=$(pwd):/workspace caffe_dl:gpu caffe train --solver=example_solver.prototxt
```
will train a network defined in the `example_solver.prototxt` file in the current directory (`$(pwd)` is maped to the container volume `/workspace` using the `--volume=` Docker flag).

Note that docker runs all commands as root by default, and thus any output files (e.g. snapshots) generated will be owned by the root user. In order to ensure that the current user is used instead, the following command can be used:
```
nvidia-docker run -ti --volume=$(pwd):/workspace -u $(id -u):$(id -g) caffe_dl:gpu caffe train --solver=example_solver.prototxt
```
where the `-u` Docker command line option runs the commands in the container as the specified user, and the shell command `id` is used to determine the user and group ID of the current user. Note that the Caffe docker images have `/workspace` defined as the default working directory. This can be overridden using the `--workdir=` Docker command line option.

# Other use-cases

Although running the `caffe` command in the docker containers as described above serves many purposes, the container can also be used for more interactive use cases. For example, specifying `bash` as the command instead of `caffe` yields a shell that can be used for interactive tasks. (Since the caffe build requirements are included in the container, this can also be used to build and run local versions of caffe).

Another use case is to run python scripts that depend on `caffe`'s Python modules. Using the `python` command instead of `bash` or `caffe` will allow this, and an interactive interpreter can be started by running:
```
nvidia-docker run -ti caffe_dl:gpu python
```
(`ipython` is also available in the container).

Since the `caffe/python` folder is also added to the path, the utility executable scripts defined there can also be used as executables. This includes `draw_net.py`, `classify.py`, and `detect.py`
