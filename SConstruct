import os
from itertools import chain

def GetAbsolutePaths(filenames):
	return map(lambda f: File(f).abspath, filenames)

def buildGoogleTest(env):
	env.Append(CPPPATH = ['#/thirdparty/googletest/googletest/include', '#/thirdparty/googletest/googlemock/include', '#/thirdparty/googletest/googletest', '#/thirdparty/googletest/googlemock'])
	googletest_files = Split('#/thirdparty/googletest/googletest/src/gtest-all.cc #/thirdparty/googletest/googlemock/src/gmock-all.cc')
	return env.StaticLibrary(googletest_files)

# Build script options
AddOption('--build-type', dest='build_type', default='Release', metavar='BUILD_TYPE', help='Build type = {Debug, Release}')
AddOption('--compiler', dest='compiler', metavar='COMPILER', help='Compiler = g++, clang++, etc.')
AddOption('--sanitize', dest='sanitize', metavar='TYPE', help='Clang sanitizer = {thread, address, undefined, ...}')
AddOption('--analyze', dest='analyze', action='store_true', default=False, help='Allow static analyzer by passing some environment variables to compiler')

env_params = {}
if GetOption('compiler'):
	env_params['CXX'] = GetOption('compiler')

env = Environment(**env_params)

if GetOption('analyze'):
	env['CC'] = os.getenv('CC')
	env['CXX'] = os.getenv('CXX')
	env['ENV'].update(x for x in os.environ.items() if x[0].startswith('CCC_'))

forced_includes = GetAbsolutePaths(Split('#/test/helgrind_annotations.h'))

compiler = os.path.basename(env.subst(env['CXX']))
if compiler.startswith('g++') or compiler.startswith('clang') or compiler.startswith('c++-analyzer'):
	env.Append(CPPFLAGS = Split('-std=c++11 -Werror=all -Werror=extra -Werror=pedantic -pedantic -pedantic-errors'))
	env.Append(CPPFLAGS = list(chain(map(lambda f: ['-include', f], forced_includes))))
	env.Append(CPPDEFINES = 'RETHREAD_HAS_POLL')
	env.Append(LINKFLAGS = Split('-pthread'))

	if GetOption('build_type') == 'Debug':
		env.Append(CPPFLAGS = Split('-ggdb -fno-omit-frame-pointer'))
	elif GetOption('build_type') == 'Release':
		env.Append(CPPFLAGS = Split('-O2'))
	else:
		raise RuntimeError('Unknown build type: {}'.format(GetOption('build_type')))

	if GetOption('sanitize'):
		env.Append(CPPFLAGS = ['-fsanitize=' + GetOption('sanitize')])
		env.Append(LINKFLAGS = ['-fsanitize=' + GetOption('sanitize')])
elif compiler.startswith('cl'):
	env.Append(CPPFLAGS = Split('-EHsc'))
else:
	raise RuntimeError('Unknown compiler: {}'.format(compiler))

source_files = Split('test/test.cpp')
object_files = env.Object(source_files)
for f in forced_includes:
	object_files = env.Depends(object_files, f)

test_runner = env.Program('test_runner', object_files, LIBS = [buildGoogleTest(env)])
env.Default(test_runner)

test_execution = env.Command('test', None, test_runner[0].abspath)
env.Depends(test_execution, test_runner)
