project('gdk-pixbuf', 'c',
  version: '2.43.4',
  license: 'LGPL-2.1-or-later',
  default_options: [
    'buildtype=debugoptimized',
    'warning_level=1',
    'c_std=gnu99',
  ],
  meson_version: '>= 1.5',
)

add_project_arguments([ '-D_POSIX_C_SOURCE=200809L', '-D_DEFAULT_SOURCE', '-D_XOPEN_SOURCE=700' ], language: 'c')

cc = meson.get_compiler('c')
host_system = host_machine.system()

# Versioning
gdk_pixbuf_version = meson.project_version()
version_arr = gdk_pixbuf_version.split('.')
gdk_pixbuf_version_major = version_arr[0].to_int()
gdk_pixbuf_version_minor = version_arr[1].to_int()
gdk_pixbuf_version_micro = version_arr[2].to_int()

gdk_pixbuf_api_version = '2.0'
gdk_pixbuf_binary_version = '2.10.0'

gdk_pixbuf_api_name = '@0@-@1@'.format(meson.project_name(), gdk_pixbuf_api_version)

if gdk_pixbuf_version_minor.is_odd()
  gdk_pixbuf_interface_age = 0
else
  gdk_pixbuf_interface_age = gdk_pixbuf_version_micro
endif

gdk_pixbuf_binary_age = 100 * gdk_pixbuf_version_minor + gdk_pixbuf_version_micro

# maintaining compatibility with the previous libtool versioning
# current = binary - interface
# revision = interface
soversion = 0
current = gdk_pixbuf_binary_age - gdk_pixbuf_interface_age
revision = gdk_pixbuf_interface_age
libversion = '@0@.@1@.@2@'.format(soversion, current, revision)
age = gdk_pixbuf_binary_age - gdk_pixbuf_interface_age
current_minus_age = current - age
darwin_versions = ['@0@'.format(current + 1), '@0@.@1@'.format(current + 1, gdk_pixbuf_interface_age)]

# Paths
gdk_pixbuf_prefix = get_option('prefix')
gdk_pixbuf_libdir = gdk_pixbuf_prefix / get_option('libdir')
gdk_pixbuf_bindir = gdk_pixbuf_prefix / get_option('bindir')
gdk_pixbuf_includedir = gdk_pixbuf_prefix / get_option('includedir')
gdk_pixbuf_datadir = gdk_pixbuf_prefix / get_option('datadir')
gdk_pixbuf_mandir = gdk_pixbuf_prefix / get_option('mandir')
gdk_pixbuf_localedir = gdk_pixbuf_prefix / get_option('localedir')
gdk_pixbuf_libexecdir = gdk_pixbuf_prefix / get_option('libexecdir')
gdk_pixbuf_loaderdir = gdk_pixbuf_libdir / 'gdk-pixbuf-@0@/@1@/loaders'.format(gdk_pixbuf_api_version, gdk_pixbuf_binary_version)
gdk_pixbuf_docsdir = gdk_pixbuf_datadir / 'doc'

# Dependencies
glib_req_version = '>= 2.56.0'
glib_dep = dependency('glib-2.0', version: glib_req_version,
                      fallback : ['glib', 'libglib_dep'])
gobject_dep = dependency('gobject-2.0', version: glib_req_version,
                         fallback : ['glib', 'libgobject_dep'])
gmodule_dep = dependency('gmodule-no-export-2.0', version: glib_req_version,
                         fallback : ['glib', 'libgmodule_dep'])
gio_dep = dependency('gio-2.0', version: glib_req_version,
                     fallback : ['glib', 'libgio_dep'])

# Configurations
gdk_pixbuf_conf = configuration_data()

check_headers = [
  'unistd.h',
  'sys/resource.h',
  'sys/time.h'
]

foreach h: check_headers
  if cc.has_header(h)
    gdk_pixbuf_conf.set('HAVE_' + h.underscorify().to_upper(), 1)
  endif
endforeach

# Look for the math library first, since we use it to test for round() and lrint()
mathlib_dep = cc.find_library('m', required: false)

# XXX: Remove the checks for round() and lrint() once GDK-Pixbuf is declared C99
if cc.has_function('round', dependencies: mathlib_dep)
  gdk_pixbuf_conf.set('HAVE_ROUND', 1)
endif

if cc.has_function('lrint', dependencies: mathlib_dep)
  gdk_pixbuf_conf.set('HAVE_LRINT', 1)
endif

intl_dep = cc.find_library('intl', required: false)
if cc.has_function('bind_textdomain_codeset', prefix: '#include <libintl.h>', dependencies: intl_dep)
  gdk_pixbuf_conf.set('HAVE_BIND_TEXTDOMAIN_CODESET', 1)
endif

if cc.has_function('setrlimit', prefix: '#include <sys/time.h>\n#include <sys/resource.h>')
  gdk_pixbuf_conf.set('HAVE_SETRLIMIT', 1)
endif

if cc.has_function('readlink', prefix: '#include <unistd.h>')
  gdk_pixbuf_conf.set('HAVE_READLINK', 1)
endif

# We use links() because sigsetjmp() is often a macro hidden behind other macros
gdk_pixbuf_conf.set('HAVE_SIGSETJMP',
  cc.links('''#define _POSIX_SOURCE
              #include <setjmp.h>
              int main (void) {
                sigjmp_buf env;
                sigsetjmp (env, 0);
                return 0;
              }''', name: 'sigsetjmp'),
)

# Common compiler and linker flags
common_cflags = []
common_ldflags = []

if cc.get_id() == 'msvc'
  # For Visual Studio, just force-include msvc_reommended_pragmas.h
  # so that we silence unwanted noise and track potential issues
  test_cflags = [ '-FImsvc_recommended_pragmas.h', '-utf-8' ]
  add_project_arguments(cc.get_supported_arguments(test_cflags), language: 'c')
elif cc.get_id() == 'gcc' or cc.get_id() == 'clang'
  test_cflags = [
    '-Wpointer-arith',
    '-Wformat=2',
    '-Wstrict-prototypes',
    '-Wnested-externs',
    '-Wold-style-definition',
    '-Wdeclaration-after-statement',
    '-Wunused',
    '-Wcast-align',
    '-Wmissing-noreturn',
    '-Wmissing-format-attribute',
    '-Wlogical-op',
    '-fno-strict-aliasing',
    '-Wno-int-conversion',
    '-Wno-uninitialized',
    '-Wno-discarded-qualifiers',
    '-Werror=undef',
    '-Werror=implicit',
    '-Werror=nonnull',
    '-Werror=init-self',
    '-Werror=main',
    '-Werror=missing-braces',
    '-Werror=sequence-point',
    '-Werror=return-type',
    '-Werror=trigraphs',
    '-Werror=array-bounds',
    '-Werror=write-strings',
    '-Werror=address',
    '-Werror=int-to-pointer-cast',
    '-Werror=pointer-to-int-cast',
    '-Werror=empty-body',
  ]

  # Ensure we have the correct bit packing on Windows
  if host_system == 'windows'
    test_cflags += '-mms-bitfields'
  endif
else
  test_cflags = []
endif

# Symbol visibility
if get_option('default_library') != 'static'
  if host_system == 'windows'
    gdk_pixbuf_conf.set('DLL_EXPORT', true)
    gdk_pixbuf_conf.set('_GDK_PIXBUF_EXTERN', '__declspec(dllexport) extern')
    if cc.get_id() != 'msvc'
      test_cflags += ['-fvisibility=hidden']
    endif
  else
    gdk_pixbuf_conf.set('_GDK_PIXBUF_EXTERN', '__attribute__((visibility("default"))) extern')
    test_cflags += ['-fvisibility=hidden']
  endif
endif

common_cflags += cc.get_supported_arguments(test_cflags)

if host_machine.system() == 'linux'
  # Additional linker flags
  test_ldflags = ['-Wl,-Bsymbolic', '-Wl,-z,relro', '-Wl,-z,now']
  common_ldflags += cc.get_supported_link_arguments(test_ldflags)
  gdk_pixbuf_conf.set('OS_LINUX', 1)
endif

if host_machine.system() == 'darwin'
  gdk_pixbuf_conf.set('OS_DARWIN', 1)
endif

android_opt = get_option('android').disable_auto_if(host_system != 'android')
jnigraphics_dep = cc.find_library('jnigraphics', required: android_opt)

# On non-Windows/macOS/android systems we always required shared-mime-info and GIO
# shared_mime_dep = []
shared_mime_dep = []
use_gio_mime = false
if get_option('gio_sniffing') and host_system not in ['windows', 'darwin', 'android']
  shared_mime_dep += dependency('shared-mime-info')
  gdk_pixbuf_conf.set('GDK_PIXBUF_USE_GIO_MIME', 1)
  use_gio_mime = true
endif

# Check if medialib is available
use_medialib = false
medialib_dep = cc.find_library('mlib', required: false)
if medialib_dep.found()
  if cc.has_function('mlib_ImageSetStruct', dependencies: medialib_dep)
    gdk_pixbuf_conf.set('USE_MEDIALIB', 1)

    if cc.has_function('mlib_VideoColorRGBint_to_BGRAint', dependencies: medialib_dep)
      gdk_pixbuf_conf.set('USE_MEDIALIB25', 1)
    endif

    use_medialib = true
  else
    medialib_dep = []
  endif
endif

gdk_pixbuf_deps = [
  mathlib_dep,
  glib_dep,
  gobject_dep,
  gmodule_dep,
  gio_dep,
  shared_mime_dep,
  medialib_dep,
  intl_dep,
]

# Check if we can build shared modules
if gmodule_dep.type_name() == 'pkgconfig'
  build_modules = gmodule_dep.get_variable(pkgconfig: 'gmodule_supported') == 'true'
else
  build_modules = subproject('glib').get_variable('g_module_impl') != '0'
endif
gdk_pixbuf_conf.set('USE_GMODULE', build_modules)

# Loader dependencies
enabled_loaders = []
loaders_deps = []

if jnigraphics_dep.found()
  loaders_deps += jnigraphics_dep
  enabled_loaders += 'android'
endif

# Check which loaders should be built into gdk-pixbuf
builtin_loaders = get_option('builtin_loaders')
if builtin_loaders.contains('default')
  if host_machine.system() == 'linux'
    builtin_loaders = ['glycin']
  elif enabled_loaders.contains('android')
    # android loaders are always builtin
  else
    builtin_loaders = ['png','jpeg']
  endif
endif

# If 'all' is specified for builtin_loaders, build all
# buildable loaders into gdk-pixbuf
builtin_all_loaders = false
if builtin_loaders.contains('all')
  builtin_all_loaders = true
elif builtin_loaders.contains('none')
  builtin_loaders = []
endif

is_msvc_like = cc.get_argument_syntax() == 'msvc'

png_opt = get_option('png').disable_auto_if(host_machine.system() == 'linux' or enabled_loaders.contains('android'))

if not png_opt.disabled()
  png_dep = dependency(is_msvc_like ? 'png' : 'libpng', required: false)

  # Finally, look for the dependency in a fallback subproject if allowed by
  # the --wrap-mode option. We don't directly call subproject() here because
  # that will bypass --wrap-mode and cause issues for distro packagers.
  # See: https://mesonbuild.com/Reference-manual.html#dependency
  if not png_dep.found()
    png_dep = dependency('libpng',
      fallback: ['libpng', 'libpng_dep'],
      required: png_opt,
    )
  endif

  if png_dep.found()
    enabled_loaders += 'png'
    loaders_deps += png_dep
  endif
endif

glycin_opt = get_option('glycin').enable_auto_if(host_machine.system() == 'linux')
glycin_dep = dependency('glycin-2', version: '>= 2.0.alpha.3', required: glycin_opt)
if glycin_dep.found()
  enabled_loaders += 'glycin'
  loaders_deps += [ glycin_dep ]
endif

gif_opt = get_option('gif').disable_auto_if(host_machine.system() == 'linux' or enabled_loaders.contains('android'))
if not gif_opt.disabled()
  enabled_loaders += 'gif'
endif

others_opt = get_option('others')
if not others_opt.disabled()
  # Keep sorted alphabetically
  enabled_loaders += [
    'ani',
    'bmp',
    'icns',
    'ico',
    'pnm',
    'qtif',
    'tga',
    'xbm',
    'xpm',
  ]
endif

# On Windows, check whether we are building the native Windows loaders
# (it is an "all-or-nothing" option for BMP, EMF, GIF, ICO, JPEG, TIFF and WMF)
# Note that we currently do not use the native Windows loaders to handle PNG and
# JPEG2000 images
if host_system == 'windows'
  native_windows_loaders = get_option('native_windows_loaders')
else
  native_windows_loaders = false
endif

if native_windows_loaders
  loaders_deps += cc.find_library('gdiplus')
  loaders_deps += cc.find_library('ole32')
  enabled_loaders += 'gdiplus'
endif

# Don't check and build the jpeg loader if native_windows_loaders is true
jpeg_opt = get_option('jpeg').disable_auto_if(host_machine.system() == 'linux' or enabled_loaders.contains('android'))
if not jpeg_opt.disabled() and not native_windows_loaders
  jpeg_dep = dependency(is_msvc_like ? 'jpeg' : 'libjpeg', required: false)

  # Finally, look for the dependency in a fallback
  if not jpeg_dep.found()
    jpeg_dep = dependency('libjpeg',
      fallback: ['libjpeg-turbo', 'jpeg_dep'],
      required: jpeg_opt,
    )
  endif

  if jpeg_dep.found()
    enabled_loaders += 'jpeg'
    loaders_deps += jpeg_dep

    if jpeg_dep.type_name() == 'internal'
      has_destroy_decompress = true
      has_simple_progression = true
    else
      has_destroy_decompress = cc.has_function('jpeg_destroy_decompress', dependencies: jpeg_dep)
      has_simple_progression = cc.has_function('jpeg_simple_progression', dependencies: jpeg_dep)
    endif

    if has_destroy_decompress and has_simple_progression
      gdk_pixbuf_conf.set('HAVE_PROGRESSIVE_JPEG', has_simple_progression)
    endif
  endif
endif

# Don't check and build the tiff loader if native_windows_loaders is true
tiff_opt = get_option('tiff').disable_auto_if(host_machine.system() == 'linux')
if not tiff_opt.disabled() and not native_windows_loaders
  # We currently don't have a fallback subproject, but this handles error
  # reporting if tiff_opt is enabled.
  tiff_dep = dependency(is_msvc_like ? 'tiff' : 'libtiff-4', required: tiff_opt)

  if tiff_dep.found()
    enabled_loaders += 'tiff'
    loaders_deps += tiff_dep
  endif
endif

# Determine whether we enable application bundle relocation support, and we use
# this always on Windows
if host_system == 'windows'
  relocatable = (get_option('default_library') != 'static')
else
  relocatable = get_option('relocatable')
endif

if relocatable
  add_project_arguments([ '-DGDK_PIXBUF_RELOCATABLE' ], language: 'c')
endif

gdk_pixbuf_conf.set_quoted('GETTEXT_PACKAGE', meson.project_name())

configure_file(output: 'config.h', configuration: gdk_pixbuf_conf)
add_project_arguments([ '-DHAVE_CONFIG_H=1' ], language: 'c')

root_inc = include_directories('.')

# Auxiliary scripts
gen_resources = find_program('build-aux/gen-resources.py')
gen_installed_test = find_program('build-aux/gen-installed-test.py')
gen_thumbnailer = find_program('build-aux/gen-thumbnailer.py')

# Needed by gen-resources.py
glib_compile_resources = find_program('glib-compile-resources')

gnome = import('gnome')

subdir('gdk-pixbuf')

# i18n
subdir('po')

if not meson.is_cross_build()
  if get_option('tests')
    subdir('tests')
  endif
  if get_option('thumbnailer').enabled()
    subdir('thumbnailer')
  endif
endif

# Documentation
subdir('docs')

if not meson.is_cross_build()
  meson.add_install_script('build-aux/post-install.py',
    gdk_pixbuf_bindir,
    gdk_pixbuf_libdir,
    gdk_pixbuf_binary_version,
  )
endif

if not meson.is_subproject()
  meson.add_dist_script('build-aux/dist.py')
endif

summary({
    'prefix': gdk_pixbuf_prefix,
    'libdir': gdk_pixbuf_libdir,
    'datadir': gdk_pixbuf_datadir,
    'libexecdir': gdk_pixbuf_libdir,
    'docdir': gdk_pixbuf_docsdir,
  },
  section: 'Directories',
)

summary({
    'Shared modules': build_modules,
    'Enabled loaders': enabled_loaders,
    'Builtin loaders': builtin_loaders,
    'Thumbnailer': get_option('thumbnailer').enabled(),
  },
  section: 'Loaders',
  bool_yn: true,
)

summary({
    'Debugging': get_option('debug'),
    'Optimization': get_option('optimization'),
    'GIO MIME sniffing': use_gio_mime,
    'MediaLib': use_medialib,
    'Introspection': build_gir,
    'Documentation': build_docs,
    'Manual pages': get_option('man'),
    'Relocatable': relocatable,
    'Build tests': get_option('tests'),
    'Installed tests': get_option('installed_tests'),
  },
  section: 'Build',
  bool_yn: true,
)
