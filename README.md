# ros2_v4l2_camera

A ROS 2 camera driver using Video4Linux2 (V4L2).

## Dependencies

* `image_transport` - to allow for compressed image transport.

    The ROS 2 port of `image_transport` in the `image_common`
    repository is needed:

        git clone --branch ros2 https://github.com/ros-perception/image_common.git src/image_common

    Note that `image_transport` only supports raw transport by default
    and needs additional plugins to actually provide compression; see
    below how to do this.
  
## Nodes

### ros2_v4l2_camera_node

The `ros2_v4l2_camera_node` interfaces with standard V4L2 devices and
publishes images as `sensor_msgs/Image` messages.

Uses the `image_transport` to allow compressed image transport. For
this, the ROS 2 port of `image_transport` in the `image_common`
repository is needed:

    cd src && git clone --branch ros2  https://github.com/ros-perception/image_common.git


#### Published Topics

* `/raw_image` - `sensor_msgs/Image`

    The image.

#### Parameters

* `video_device` - `string`, default: `"/dev/video0"`

    The device the camera is on.

* `output_encoding` - `string`, default: `"rgb8"`

    The encoding to use for the output image. Currently supported: "rgb8" or "yuv422".

* `image_size` - `integer_array`, default: `[640, 480]`

    Width and height of the image.

* Camera Control Parameters

    Camera controls, such as brightness, contrast, white balance, etc,
    are automatically made available as parameters. The driver node
    enumerates all controls, and creates a parameter for each, with
    the corresponding value type. The parameter name is derived from
    the control name reported by the camera driver, made lower case,
    commas removed, and spaces replaced by underscores. So
    `Brightness` becomes `brightness`, and `White Balance, Automatic`
    becomes `white_balance_automatic`.

## Compressed Transport

By default `image_transport` only supports raw transfer, plugins are
required to enable compression. Standard ones are available in the
[`image_transport_plugins`](https://github.com/ros-perception/image_transport_plugins)
repository. These depend on the OpenCV facilities provided by the
`vision_opencv` repository. You can clone these into your workspace to
get these:

    cd path/to/workspace
    git clone https://github.com/ros-perception/vision_opencv.git --branch ros2 src/vision_opencv
    git clone https://github.com/ros-perception/image_transport_plugins.git --branch ros2 src/image_transport_plugins

### Building: Ubuntu

The following packages are required:

    sudo apt install libtheora-dev libogg-dev

### Building: Arch

To get the plugins compiled on Arch Linux, a few special steps are
needed:

* Arch provides OpenCV 4.x, but OpenCV 3.x is required
* Arch provides VTK 8.2, but VTK 8.1 is required
* `boost-python` is used, which [needs to be linked to python libs
  explicitly](https://bugs.archlinux.org/task/55798):

        colcon build --symlink-install --packages-select cv_bridge --cmake-args "-DCMAKE_CXX_STANDARD_LIBRARIES=-lpython3.7m"

### Usage

If the compression plugins are compiled and installed in the current
workspace, they will be automatically used by the driver and an
additional `/image_raw/compressed` topic will be available.

Neither Rviz2 or `showimage` use `image_transport` (yet). Therefore, to
be able to view the compressed topic, it needs to be republished
uncompressed. `image_transport` comes with the `republish` node to do
this:

    ros2 run image_transport republish compressed in/compressed:=image_raw/compressed raw out:=image_raw/uncompressed

The parameters mean:

* `compressed` - the transport to use for input, in this case
  'compressed'. Alternative: `raw`, to republish the raw `/image_raw`
  topic
* `in/compressed:=image_raw/compressed` - by default, `republish` uses
  the topics `in` and `out`, or `in/compressed` for example if the
  input transport is 'compressed'. This parameter is a ROS remapping
  rule to map those names to the actual topic to use.
* `raw` - the transport to use for output. If omitted, all available
  transports are provided.
* `out:=image_raw/uncompressed` - remapping of the output topic.
