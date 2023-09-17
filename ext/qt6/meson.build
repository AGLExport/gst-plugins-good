sources = [
  'gstplugin.cc',
  'gstqt6element.cc',
  'gstqsg6glnode.cc',
  'gstqt6glutility.cc',
  'gstqml6glsink.cc',
  'qt6glitem.cc',
]

moc_headers = [
  'qt6glitem.h',
  'gstqsg6glnode.h',
]

qt6qml_dep = dependency('', required: false)
qt6_option = get_option('qt6')
qt6_egl = get_option('qt-egl')
qt6_wayland = get_option('qt-wayland')
qt6_x11 = get_option('qt-x11')
qt6_method = get_option('qt-method')

if qt6_option.disabled()
  subdir_done()
endif

if not have_gstgl
  if qt6_option.enabled()
    error('qt6 qmlglsink plugin is enabled, but gstreamer-gl-1.0 was not found')
  endif
  subdir_done()
endif

if not add_languages('cpp', native: false, required: qt6_option)
  subdir_done()
endif

qt6_mod = import('qt6')
if not qt6_mod.has_tools()
  if qt6_option.enabled()
    error('qt6 qmlglsink plugin is enabled, but qt specific tools were not found')
  endif
  subdir_done()
endif

qt6qml_dep = dependency('qt6', modules : ['Core', 'Gui', 'Qml', 'Quick'],
                        method: qt6_method, required: qt6_option, static: host_system == 'ios')
if not qt6qml_dep.found()
  subdir_done()
endif

optional_deps = []
qt_defines = []
have_qpa_include = false
have_qt_windowing = false

# Look for the QPA platform native interface header
qpa_header_path = join_paths(qt6qml_dep.version(), 'QtGui')
qpa_header = join_paths(qpa_header_path, 'qpa/qplatformnativeinterface.h')
need_qpa_include = qt6_option.enabled() and (host_system == 'android' or qt6_wayland.enabled())
if cxx.has_header(qpa_header, dependencies : qt6qml_dep, required: need_qpa_include)
  qt_defines += '-DHAVE_QT_QPA_HEADER'
  qt_defines += '-DQT_QPA_HEADER=' + '<@0@>'.format(qpa_header)
  have_qpa_include = true
  message('Found QtGui QPA header in ' + qpa_header_path)
endif

## Try to come up with all the platform/winsys combinations that will work

# X11 windowing
if gst_gl_have_window_x11 and gst_gl_have_platform_glx
  # FIXME: automagic
  qt5x11extras = dependency('qt6', modules : ['X11Extras'], required : false)
  if qt5x11extras.found()
    optional_deps += qt6x11extras
    qt_defines += ['-DHAVE_QT_X11']
    have_qt_windowing = true
  endif
endif

# EGL support
if gst_gl_have_platform_egl
  # Embedded linux (e.g. i.MX6) with or without windowing support
  #if qt6_egl.allowed()
  #  qt_defines += ['-DHAVE_QT_EGLFS']
  #  have_qt_windowing = true
  #endif
  if have_qpa_include
    # Wayland windowing
    if gst_gl_have_window_wayland
      qt6waylandextras = dependency('qt6', modules : ['WaylandClient'], method: qt6_method, required: qt6_wayland)
      if qt6waylandextras.found()
        optional_deps += [qt6waylandextras]
        qt_defines += ['-DHAVE_QT_WAYLAND']
        have_qt_windowing = true
      endif
    endif
  endif
endif

if qt6_option.require(have_qt_windowing).allowed()
  # Build it!
  moc_files = qt6_mod.preprocess(moc_headers : moc_headers)
  gstqml6gl = library('gstqml6', sources, moc_files,
    cpp_args : gst_plugins_good_args + qt_defines,
    link_args : noseh_link_args,
    include_directories: [configinc, libsinc],
    dependencies : [gst_dep, gstvideo_dep, gstgl_dep, qt6qml_dep, optional_deps],
    override_options : ['cpp_std=c++17'],
    install: true,
    install_dir : plugins_install_dir)
  pkgconfig.generate(gstqml6gl, install_dir : plugins_pkgconfig_install_dir)
endif
