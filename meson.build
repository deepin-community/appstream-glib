project('appstream-glib', 'c',
  version : '0.8.2',
  license : 'LGPL-2.1+',
  default_options : ['warning_level=1', 'c_std=c99'],
  meson_version : '>=0.46.0'
)

cc = meson.get_compiler('c')

as_version = meson.project_version()
varr = as_version.split('.')
as_major_version = varr[0]
as_minor_version = varr[1]
as_micro_version = varr[2]

# Check the right platform.

platform_win32 = false
platform_osx   = false

host_os = host_machine.system()

platform_win32 = (host_os.startswith('mingw') or
                  host_os.startswith('cygwin') or
                  host_os.startswith('windows'))
platform_osx = host_os.startswith('darwin')

conf = configuration_data()
conf.set('AS_MAJOR_VERSION_CONF', as_major_version)
conf.set('AS_MINOR_VERSION_CONF', as_minor_version)
conf.set('AS_MICRO_VERSION_CONF', as_micro_version)
conf.set_quoted('PACKAGE_VERSION', as_version)

# this refers to the plugin API version
# this is not in any way related to a package or soname version
as_plugin_version = '5'

# libtool versioning - this applies to libpackagekit
#
# See http://sources.redhat.com/autobook/autobook/autobook_91.html#SEC91 for details
#
# - If interfaces have been changed or added, but binary compatibility
#   has been preserved, change:
#      CURRENT += 1
#      REVISION = 0
#      AGE += 1
# - If binary compatibility has been broken (eg removed or changed
#   interfaces), change:
#      CURRENT += 1
#      REVISION = 0
#      AGE = 0
# - If the interface is the same as the previous version, but bugs are
#   fixed, change:
#      REVISION += 1
lt_current = '8'
lt_revision = '10'
lt_age = '0'
lt_version = '@0@.@1@.@2@'.format(lt_current, lt_age, lt_revision)

# set plugindir for builder
plugindir = join_paths(get_option('prefix'),
                       get_option('libdir'),
                       'asb-plugins-' + as_plugin_version)

glib_ver = '>= 2.58.0'
gio = dependency('gio-2.0', version : glib_ver)
gmodule = dependency('gmodule-2.0', version : glib_ver)
if platform_win32
  giowindows = dependency('gio-windows-2.0', version : glib_ver)
elif platform_osx
  giounix = dependency('gio-unix-2.0', version : glib_ver)
else
  giounix = dependency('gio-unix-2.0', version : glib_ver)
  uuid = dependency('uuid')
endif
libarchive = dependency('libarchive')
libcurl = dependency('libcurl', version : '>= 7.56.0')
json_glib = dependency('json-glib-1.0', version : '>= 1.1.2')
gdkpixbuf = dependency('gdk-pixbuf-2.0', version : '>= 2.31.5')

# builder (default enabled)
if get_option('builder')
  gmodule = dependency('gmodule-2.0')
  if get_option('alpm')
    alpm = dependency('libalpm')
    conf.set('HAVE_ALPM', 1)
  endif
  if get_option('fonts')
    gdk = dependency('gdk-3.0')
    conf.set('HAVE_FONTS', 1)
    freetype = dependency('freetype2', version : '>= 9.10.0')
    fontconfig = dependency('fontconfig')
  endif
endif

# rpm is used for vercmp and to get the filelist in the builder
if get_option('rpm')
  rpm = dependency('rpm')
  conf.set('HAVE_RPM', 1)
endif

# support loading yaml files
if get_option('dep11')
  yaml = dependency('yaml-0.1')
  conf.set('AS_BUILD_DEP11', 1)
endif

# support stemming of search tokens
if get_option('stemmer')
  stemmer = cc.find_library('stemmer')
  conf.set('HAVE_LIBSTEMMER', 1)
endif

# use gperf for faster string -> enum matching
gperf = find_program('gperf', required : true)

gnome = import('gnome')
i18n = import('i18n')

conf.set_quoted('GETTEXT_PACKAGE', meson.project_name())
conf.set_quoted('LOCALEDIR', get_option('localedir'))
conf.set_quoted('LOCALSTATEDIR', get_option('localstatedir'))
configure_file(
  output : 'config.h',
  configuration : conf
)

add_global_arguments('-DAS_COMPILATION', language : 'c')

# Needed for realpath()
add_project_arguments('-D_XOPEN_SOURCE=500', language : 'c')

subdir('libappstream-glib')
subdir('data')
subdir('docs')
subdir('po')

if get_option('builder')
  # depends on data
  subdir('libappstream-builder')
endif

# depends on libappstream-builder
subdir('client')

if meson.version().version_compare('<0.41.0')
  archiver = find_program('git', required : false)
  if archiver.found()
    run_target('dist',
      # git config tar.tar.xz.command "xz -c"
      command: [
        'git', 'archive',
        '--prefix=' + meson.project_name() + '-' + meson.project_version() + '/',
        'HEAD',
        '--format=tar.xz',
        '--output',
        meson.project_name() + '-' + meson.project_version() + '.tar.xz'
      ]
    )
  else
    message('git not found, you will not be able to run `ninja dist`')
  endif
endif
