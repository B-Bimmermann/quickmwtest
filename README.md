Multi-screen test app for Qt 5.7 and up using the eglfs platform
plugin. Targeted mainly at the NVIDIA DRIVE CX and Renesas R-Car H2
and H3, and is the test bed for QTBUG-55188, QTBUG-55161 and
QTBUG-54151, QTBUG-57762, etc.

There will be one QQuickView created for each screen. With the default
threaded render loop both will get its own render thread which is good
since they can then be throttled independently.


Logging is enabled automatically, a typical output should be something
like the following, assuming a mouse and two screens connected via
HDMI and DisplayPort.

Pay special attention to lines like:

```
qt.qpa.eglfs.kms: Physical size is QSizeF(0, 0) mm for output "HDMI1"
```

This shows that the DRM subsystem did not report a physical width and
height for the output in question.  If this happens, you will want to
manually provide the correct size via the JSON configuration file, see
http://doc-snapshots.qt.io/qt5-5.8/embedded-linux.html#eglfs-with-eglfs-kms-backend

```
qt.qpa.eglfs.kms: Sorted screen list: QVector(OrderedScreen(0x44a3d0 : 2147483647), OrderedScreen(0x44b700 : 2147483647))
qt.qpa.eglfs.kms: Adding screen 0x44a3d0 to QPA with geometry QRect(0,0 1920x1200)
qt.qpa.eglfs.kms: Adding screen 0x44b700 to QPA with geometry QRect(1920,0 1920x1080)
...
Application sees 2 screens
(QScreen(0x449d80, name="DP1"), QScreen(0x449d40, name="HDMI1"))
```

This shows what gets registered to Qt. In the example above two
QScreen instances will show in QGuiApplication::screens(), with
different resolutions (not necessarily ideal), with the HDMI screen
placed to the right of the DisplayPort one in the virtual desktop.

NB! future Qt versions are expected to improve logging; there are
also continuous improvements to the multiscreen support in general, so
do not be surprised if something looks different with newer Qt
versions

If there are additional, unexpected outputs (some boards tend to
report LVDS as connected even when it is not, for instance), disable
it via the config file.

Same goes if the resolution or the virtual desktop layout is not as
expected: provide a config file. As an example, check
example_rcar_h2_config.json. There we configure HDMI and VGA to 720p
and get rid of an unwanted third output, and we manually specify the
physical size for the VGA output as it does not get reported
automatically. To use this configuration in Qt applications, just set
the environment variable
QT_QPA_EGLFS_KMS_CONFIG=example_rcar_h2_config.json



Full example log:

```
nvidia@nvidia:~$ ./quickmwtest
qt.qpa.egldeviceintegration: EGL device integration plugin keys: ("eglfs_kms_egldevice")
qt.qpa.egldeviceintegration: EGL device integration plugin keys (sorted): ("eglfs_kms_egldevice")
qt.qpa.egldeviceintegration: Trying to load device EGL integration "eglfs_kms_egldevice"
qt.qpa.eglfs.kms: New DRM/KMS on EGLDevice integration created
qt.qpa.egldeviceintegration: Using EGL device integration "eglfs_kms_egldevice"

qt.qpa.eglfs.kms: Found 1 EGL devices
qt.qpa.eglfs.kms: Opening DRM device drm-nvdc
qt.qpa.eglfs.kms: Creating display
qt.qpa.eglfs.kms: "DP1" mode count: 10
qt.qpa.eglfs.kms: mode 0 1920 x 1200 @ 59 hz
qt.qpa.eglfs.kms: mode 1 720 x 400 @ 70 hz
qt.qpa.eglfs.kms: mode 2 640 x 480 @ 59 hz
qt.qpa.eglfs.kms: mode 3 800 x 600 @ 60 hz
qt.qpa.eglfs.kms: mode 4 1024 x 768 @ 60 hz
qt.qpa.eglfs.kms: mode 5 1280 x 960 @ 60 hz
qt.qpa.eglfs.kms: mode 6 1280 x 1024 @ 60 hz
qt.qpa.eglfs.kms: mode 7 1600 x 1200 @ 60 hz
qt.qpa.eglfs.kms: mode 8 1680 x 1050 @ 59 hz
qt.qpa.eglfs.kms: mode 9 1920 x 1080 @ 60 hz
qt.qpa.eglfs.kms: Selected mode 0 : 1920 x 1200 @ 59 hz for output "DP1"
qt.qpa.eglfs.kms: Physical size is QSizeF(0, 0) mm for output "DP1"
qt.qpa.eglfs.kms: Creating new global mouse cursor
qt.qpa.eglfs.kms: "HDMI1" mode count: 11
qt.qpa.eglfs.kms: mode 0 1920 x 1080 @ 59 hz
qt.qpa.eglfs.kms: mode 1 640 x 480 @ 59 hz
qt.qpa.eglfs.kms: mode 2 800 x 600 @ 56 hz
qt.qpa.eglfs.kms: mode 3 800 x 600 @ 60 hz
qt.qpa.eglfs.kms: mode 4 1024 x 768 @ 60 hz
qt.qpa.eglfs.kms: mode 5 1280 x 800 @ 59 hz
qt.qpa.eglfs.kms: mode 6 1280 x 960 @ 60 hz
qt.qpa.eglfs.kms: mode 7 1280 x 1024 @ 60 hz
qt.qpa.eglfs.kms: mode 8 1440 x 900 @ 59 hz
qt.qpa.eglfs.kms: mode 9 1600 x 1200 @ 60 hz
qt.qpa.eglfs.kms: mode 10 1680 x 1050 @ 59 hz
qt.qpa.eglfs.kms: Selected mode 0 : 1920 x 1080 @ 59 hz for output "HDMI1"
qt.qpa.eglfs.kms: Physical size is QSizeF(0, 0) mm for output "HDMI1"

qt.qpa.eglfs.kms: Sorted screen list: QVector(OrderedScreen(0x44a3d0 : 2147483647), OrderedScreen(0x44b700 : 2147483647))
qt.qpa.eglfs.kms: Adding screen 0x44a3d0 to QPA with geometry QRect(0,0 1920x1200)
qt.qpa.eglfs.kms: Adding screen 0x44b700 to QPA with geometry QRect(1920,0 1920x1080)

qt.qpa.input: evdevkeyboard: Using device discovery
...
qt.qpa.input: Found mouse at "/dev/input/event0"
qt.qpa.input: Found matching devices ("/dev/input/event0")
qt.qpa.input: Adding mouse at "/dev/input/event0"
qt.qpa.input: create mouse handler for "/dev/input/event0" ""
qt.qpa.input: evdevtouch: Using device discovery
...

Application sees 2 screens
(QScreen(0x449d80, name="DP1"), QScreen(0x449d40, name="HDMI1"))

Creating new QQuickView for screen 0x449d80
qt.scenegraph.general: threaded render loop
qt.scenegraph.general: Using sg animation driver
qt.scenegraph.general: Animation Driver: using vsync: 16.95 ms
qt.qpa.eglfs.kms: Creating stream
qt.qpa.eglfs.kms: Created stream 0x4d6ba1 on display 0x1
qt.qpa.eglfs.kms: Output has 6 layers
qt.qpa.eglfs.kms: Searching for id: 1000
qt.qpa.eglfs.kms:   [0] layer 0x4481d1 - crtc 1000
qt.qpa.eglfs.kms:   [1] layer 0x449ec1 - crtc 1001
qt.qpa.eglfs.kms:   [2] layer 0x449f01 - plane 4001
qt.qpa.eglfs.kms:   [3] layer 0x449f41 - plane 4002
qt.qpa.eglfs.kms:   [4] layer 0x449f81 - plane 4004
qt.qpa.eglfs.kms:   [5] layer 0x449fc1 - plane 4005
qt.qpa.eglfs.kms: Using layer 0x4481d1
qt.qpa.eglfs.kms: Stream producer format is QSurfaceFormat(version 2.0, options QFlags(), depthBufferSize 24, redBufferSize 8, greenBufferSize 8, blueBufferSize 8, alphaBufferSize 0, stencilBufferSize 8, samples 0, swapBehavior 0, swapInterval 1, profile  0)
qt.qpa.eglfs.kms: Creating stream producer surface of size 1920x1200
qt.qpa.eglfs.kms: Created stream producer surface 0x4d8081
qt.scenegraph.general: Using sg animation driver
qt.scenegraph.general: Animation Driver: using vsync: 16.95 ms
qt.scenegraph.general: texture atlas dimensions: 2048x2048
qt.scenegraph.general: R/G/B/A Buffers:    8 8 8 0
qt.scenegraph.general: Depth Buffer:       24
qt.scenegraph.general: Stencil Buffer:     8
qt.scenegraph.general: Samples:            0
qt.scenegraph.general: GL_VENDOR:          NVIDIA Corporation
qt.scenegraph.general: GL_RENDERER:        GP10B (nvgpu)/integrated
qt.scenegraph.general: GL_VERSION:         OpenGL ES 3.2 NVIDIA 367.00
qt.scenegraph.general: GL_EXTENSIONS:      ...
qt.scenegraph.general: Max Texture Size:  32768
qt.scenegraph.general: Debug context:     false
qt.qpa.eglfs.kms: Setting mode

Creating new QQuickView for screen 0x449d40
qt.qpa.eglfs.kms: Creating stream
qt.qpa.eglfs.kms: Created stream 0x5bbec1 on display 0x1
qt.qpa.eglfs.kms: Output has 6 layers
qt.qpa.eglfs.kms: Searching for id: 1001
qt.qpa.eglfs.kms:   [0] layer 0x4481d1 - crtc 1000
qt.qpa.eglfs.kms:   [1] layer 0x449ec1 - crtc 1001
qt.qpa.eglfs.kms:   [2] layer 0x449f01 - plane 4001
qt.qpa.eglfs.kms:   [3] layer 0x449f41 - plane 4002
qt.qpa.eglfs.kms:   [4] layer 0x449f81 - plane 4004
qt.qpa.eglfs.kms:   [5] layer 0x449fc1 - plane 4005
qt.qpa.eglfs.kms: Using layer 0x449ec1
qt.qpa.eglfs.kms: Stream producer format is QSurfaceFormat(version 2.0, options QFlags(), depthBufferSize 24, redBufferSize 8, greenBufferSize 8, blueBufferSize 8, alphaBufferSize 0, stencilBufferSize 8, samples 0, swapBehavior 0, swapInterval 1, profile  0)
qt.qpa.eglfs.kms: Creating stream producer surface of size 1920x1080
qt.qpa.eglfs.kms: Created stream producer surface 0x5bece1
qt.scenegraph.general: Using sg animation driver
qt.scenegraph.general: Animation Driver: using vsync: 16.95 ms
qt.scenegraph.general: texture atlas dimensions: 2048x2048
qt.qpa.eglfs.kms: Setting mode
```
