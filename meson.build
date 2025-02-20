project('highs', 'cpp',
  version : '1.6.0',
  default_options : ['warning_level=1',
                     'cpp_std=c++17',
                     'wrap_mode=forcefallback'])


# Add C++ compiler options
_args = [] # Extra arguments
_deps = [] # Dependencies
_linkto = [] # All the sub-libraries
_incdirs = [] # All the includes

add_languages('c', required: true)
cc = meson.get_compiler('c')
cppc = meson.get_compiler('cpp')

# Platform detection
host_system = host_machine.system()
is_windows = host_system == 'windows'
is_mingw = is_windows and cc.get_id() == 'gcc'

# Conditional arguments
if host_system == 'linux'
  _args += '-Wno-return-type'
  _args += '-Wno-switch'
  _args += '-Wno-unused-variable'
  _args += '-Wno-unused-const-variable'
endif

if cppc.get_id() == 'msvc'
  add_project_arguments(
      '/wd4018',  # Disable warning: 'expression' : signed/unsigned mismatch
      '/wd4061',  # Disable warning: enumerator 'identifier' in switch of enum 'enumeration' is not explicitly handled by a case label
      '/wd4100',  # Disable warning: 'identifier' : unreferenced formal parameter
      '/wd4101',  # Disable warning: 'identifier' : unreferenced local variable
      '/wd4127',  # Disable warning: conditional expression is constant
      '/wd4189',  # Disable warning: 'identifier' : local variable is initialized but not referenced
      '/wd4244',  # Disable warning: 'conversion' conversion from 'type1' to 'type2', possible loss of data
      '/wd4245',  # Disable warning: 'conversion' conversion from 'type1' to 'type2', signed/unsigned mismatch
      '/wd4267',  # Disable warning: 'conversion' conversion from 'size_t' to 'type', possible loss of data
      '/wd4324',  # Disable warning: 'structure' structure was padded due to alignment specifier
      '/wd4365',  # Disable warning: 'action' : conversion from 'type_1' to 'type_2', signed/unsigned mismatch
      '/wd4389',  # Disable warning: 'modifier' : signed/unsigned mismatch
      '/wd4456',  # Disable warning: declaration of 'identifier' hides previous local declaration
      '/wd4457',  # Disable warning: declaration of 'identifier' hides function parameter
      '/wd4458',  # Disable warning: declaration of 'identifier' hides class member
      '/wd4459',  # Disable warning: declaration of 'identifier' hides global declaration
      '/wd4514',  # Disable warning: 'function' : unreferenced inline function has been removed
      '/wd4701',  # Disable warning: potentially uninitialized local variable 'name' used
      '/wd4820',  # Disable warning: 'bytes' bytes padding added after construct 'member_name'
      language: 'cpp',
  )
  _args += '-D_CRT_SECURE_NO_WARNINGS'
endif

cpu_family = host_machine.cpu_family()

if cpu_family in ['x86_64', 'i686'] and not is_windows
    add_project_arguments('-mpopcnt', language: 'cpp')
endif
if cpu_family in ['ppc64', 'powerpc64'] and not meson.is_cross_build()
    add_project_arguments('-mpopcntd', language: 'cpp')
endif

if is_mingw
  # For mingw-w64, don't use LTO 
  add_project_arguments('-fno-use-linker-plugin', language: ['c', 'cpp'])
endif

# --------------------- Dependencies
if not is_windows
  # libm for Unix systems
  m_dep = cppc.find_library('m', required: false)
  _deps += m_dep
  # For building with clang
  _deps += [declare_dependency(link_args: '-lstdc++')]
endif

# Required
threads_dep = dependency('threads',
                        required: true)
_deps += threads_dep

# Optional
zlib_dep = dependency('zlib',
                      required: get_option('use_zlib'))
if zlib_dep.found()
  _deps += zlib_dep
  _incdirs += include_directories(['extern/zstr'])
endif

# Include Sources
_incdirs += include_directories([
  'extern/',
  'extern/pdqsort/',
  'extern/filereaderlp/',
])

# Optional arguments
if get_option('fast_build')
  add_project_arguments(cc.get_supported_arguments('-fno-omit-frame-pointer'), language : ['c', 'cpp'])
endif

if get_option('debug_sol')
  _args += ['-DHIGHS_DEBUGSOL']
endif

if cppc.get_id() == 'msvc'
  # Don't depend on VCRUNTIME140_1.dll
  # https://cibuildwheel.readthedocs.io/en/stable/faq/#windows-importerror-dll-load-failed-the-specific-module-could-not-be-found
  add_project_arguments('/d2FH4-', language : ['cpp', 'c'])
  if get_option('with_stdcall')
    add_project_arguments(cppc.get_supported_arguments('/Gz'), language : ['cpp', 'c'])
  endif
endif

# --------------------- Library

subdir('src') # defines highslib
_linkto += highslib

# --------------------- Tests

if get_option('with_tests')
  subdir('check')
  _deps += dependency('catch2', required: true)
endif

# --------------------- Bindings

if get_option('with_pybind11')
  subdir('highspy')
endif
