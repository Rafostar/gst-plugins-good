gtk_versions = [3, 4]
gtk_sources = [
  'gstgtkbasesink.c',
  'gstgtkutils.c',
  'gstplugin.c',
  'gtkgstbasewidget.c',
]
gtk_lib_added = false

foreach gtk_ver : gtk_versions
  gtkv = 'gtk' + gtk_ver.to_string()

  if gtk_ver > 3
    allow_experiments = get_option(gtkv + '-experiments')
    if not allow_experiments
      continue
    endif
  endif

  gtk_state = get_option(gtkv)
  if gtk_state.disabled()
    continue
  endif

  min_ver = gtk_ver >= 4 ? '3.99.2'          : '3.24.0'
  pkg_dep = gtk_ver >= 4 ? gtkv              : 'gtk+-3.0'
  x11_dep = gtk_ver >= 4 ? gtkv + '-x11'     : 'gtk+-x11-3.0'
  way_dep = gtk_ver >= 4 ? gtkv + '-wayland' : 'gtk+-wayland-3.0'

  gtk_dep = dependency(pkg_dep, required : gtk_state)

  if not gtk_dep.found() or not gtk_dep.version().version_compare('>=' + min_ver)
    continue
  endif

  lib_sources = []
  gtk_defines = []
  optional_deps = []
  have_gtk_gl_windowing = false

  lib_sources += gtk_sources
  if gtk_ver == 3
    lib_sources += [
      'gstgtksink.c',
      'gtkgstwidget.c',
    ]
  endif

  if have_gstgl
    if gst_gl_have_window_x11 and gst_gl_have_platform_glx
      # FIXME: automagic
      gtk_x11_dep = dependency(x11_dep, required : false)
      if gtk_x11_dep.found()
        optional_deps += [gtk_x11_dep, gstglx11_dep]
        have_gtk_gl_windowing = true
      endif
    endif

    if gst_gl_have_window_wayland and gst_gl_have_platform_egl
      # FIXME: automagic
      gtk_wayland_dep = dependency(way_dep, required : false)
      if gtk_wayland_dep.found()
        optional_deps += [gtk_wayland_dep, gstglegl_dep, gstglwayland_dep]
        have_gtk_gl_windowing = true
      endif
    endif
  endif

  if gtk_ver > 3 and not have_gtk_gl_windowing
    continue
  endif

  if have_gtk_gl_windowing
    lib_sources += [
      'gstgtkglsink.c',
      'gtkgstglwidget.c',
    ]
    optional_deps += [gstgl_dep, gstglproto_dep]
    gtk_defines += ['-DGST_USE_UNSTABLE_API', '-DHAVE_GTK_GL']
    if gtk_ver == 4
      gtk_defines += '-DBUILD_FOR_GTK4'
    endif
  endif

  lib_name = 'gstgtk'
  if gtk_ver > 3
    lib_name += gtk_ver.to_string()
  endif

  # GStreamer loads all plugins at once. To avoid GTK versions conflicts,
  # do not install more than one plugin version on "auto".
  # The "gtk4" option should be set to "enabled" for packaging.
  is_gstgtk_install = not gtk_lib_added or not gtk_state.auto()

  gstgtk = library(lib_name,
    lib_sources,
    c_args : gst_plugins_good_args + gtk_defines,
    link_args : noseh_link_args,
    include_directories : [configinc],
    dependencies : [gtk_dep, gstvideo_dep, gstbase_dep, libm] + optional_deps,
    install : is_gstgtk_install,
    install_dir : plugins_install_dir,
  )
  pkgconfig.generate(gstgtk, install_dir : plugins_pkgconfig_install_dir)
  plugins += [gstgtk]
  gtk_lib_added = true
endforeach
