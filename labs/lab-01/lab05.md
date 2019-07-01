# Lab 5: CMake tutorial part 1

**Step 1**


- tutorial.cxx:

```
// A simple program that computes the square root of a number
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <string>

#include "TutorialConfig.h"

int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

  double inputValue = std::stod(argv[1]);

  double outputValue = sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

- CMakeLists.txt:
```
  cmake_minimum_required(VERSION 3.3)
  project(Tutorial)

  # the version number.
  set(Tutorial_VERSION_MAJOR 1)
  set(Tutorial_VERSION_MINOR 0)
  
  set(CMAKE_CXX_STANDARD 11)
  set(CMAKE_CXX_STANDARD_REQUIRED True)

  # configure a header file to pass some of the CMake settings
  # to the source code
  configure_file(
    "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
    "${PROJECT_BINARY_DIR}/TutorialConfig.h"
    )

  # add the executable
  add_executable(Tutorial tutorial.cxx)

  # add the binary tree to the search path for include files
  # so that we will find TutorialConfig.h
  target_include_directories(Tutorial PUBLIC
                             "${PROJECT_BINARY_DIR}"
                             )
```
Running the Code:


**Step 2**

- CMakeLists.txt:
```
  cmake_minimum_required(VERSION 3.3)
  project(Tutorial)

  set(CMAKE_CXX_STANDARD 14)

  # the version number.
  set(Tutorial_VERSION_MAJOR 1)
  set(Tutorial_VERSION_MINOR 0)

  # should we use our own math functions
  option(USE_MYMATH "Use tutorial provided math implementation" ON)

  # configure a header file to pass some of the CMake settings
  # to the source code
  configure_file(
    "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
    "${PROJECT_BINARY_DIR}/TutorialConfig.h"
    )

  # add the MathFunctions library?
  if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS "MathFunctions")
    list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
  endif(USE_MYMATH)

  # add the executable
  add_executable(Tutorial tutorial.cxx)

  # add the binary tree to the search path for include files
  # so that we will find TutorialConfig.h
  target_include_directories(Tutorial PUBLIC
                             "${PROJECT_BINARY_DIR}"
                             "${EXTRA_INCLUDES}"
                             )
                             
  target_link_libraries(Tutorial "${EXTRA_LIBS}")
```

- tutorial.cxx:
```
// A simple program that computes the square root of a number
#include <cmath>
#include <iostream>
#include <string>

#include "TutorialConfig.h"

#ifdef USE_MYMATH
	#include "MathFunctions.h"
#endif

int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

  double inputValue = std::stod(argv[1]);

  #ifdef USE_MYMATH
    double outputValue = mysqrt(inputValue);
  #else
    double outputValue = sqrt(inputValue);
  #endif
  
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}

```

**Step 3**


- CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the MathFunctions library?
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif(USE_MYMATH)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ) 	   


```

- MathFunctions/CMakeLists.txt:
```
add_library(MathFunctions mysqrt.cxx)
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

**Step 4**

- CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)

set(CMAKE_INSTALL_PREFIX "${PROJECT_SOURCE_DIR}")

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the MathFunctions library?
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif(USE_MYMATH)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
                           
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include
        )
        
  # enable testing
  enable_testing()

  # does the application run
  add_test(NAME Runs COMMAND Tutorial 25)

  # does the usage message work?
  add_test(NAME Usage COMMAND Tutorial)
  set_tests_properties(Usage
    PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
    )

  # define a function to simplify adding tests
  function(do_test target arg result)
    add_test(NAME Comp${arg} COMMAND ${target} ${arg})
    set_tests_properties(Comp${arg}
      PROPERTIES PASS_REGULAR_EXPRESSION ${result}
      )
  endfunction(do_test)

  # do a bunch of result based tests
  do_test(Tutorial 25 "25 is 5")
  do_test(Tutorial -25 "-25 is [-nan|nan|0]")
  do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

- MathFunctions/CMakeLists.txt:
```
add_library(MathFunctions mysqrt.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )
          
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
Running ctest:
```


**Step 5**

- CMakeLists.txt:
```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)

set(CMAKE_INSTALL_PREFIX "${PROJECT_SOURCE_DIR}")

# does this system provide the log and exp functions?
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the MathFunctions library?
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

# add the install targets
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )

# enable testing
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

- MathFunctions/CMakeLists.txt:
```
add_library(MathFunctions mysqrt.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${Tutorial_BINARY_DIR}
          )

install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```




# Part 2 of Lab

**Makefile:**
```
all: prog_static prog_shared
	
prog_static: libstatic.a program.o
	gcc program.o libstatic.a -o prog_static
	
prog_shared: libshared.so program.o
	gcc program.o libshared.so -o prog_shared -Wl,-rpath='$ORIGIN'
	
program.o:
	gcc -c program.c -o program.o
	
libstatic.a:
	gcc -c -o static.o source/block.c
	ar rcs libstatic.a static.o
	
libshared.so:
	gcc -fPIC -c source/block.c -o shared.o
	gcc -shared -o libshared.so shared.o
```



**CMake file:**
```
cmake_minimum_required(VERSION 3.3)
project(Lab)

add_library(libstatic STATIC source/block.c)

add_executable(static program.c)
target_link_libraries(static PUBLIC libstatic)

add_library(libshared SHARED source/block.c)

add_executable(shared program.c)
target_link_libraries(shared PUBLIC libshared)
```


**CMake Makefile:**
```
# CMAKE generated file: DO NOT EDIT!
# Generated by "Unix Makefiles" Generator, CMake Version 3.10

# Default target executed when no arguments are given to make.
default_target: all

.PHONY : default_target

# Allow only one "make -f Makefile2" at a time, but pass parallelism.
.NOTPARALLEL:


#=============================================================================
# Special targets provided by cmake.

# Disable implicit rules so canonical targets will work.
.SUFFIXES:


# Remove some rules from gmake that .SUFFIXES does not remove.
SUFFIXES =

.SUFFIXES: .hpux_make_needs_suffix_list


# Suppress display of executed commands.
$(VERBOSE).SILENT:


# A target that is always out of date.
cmake_force:

.PHONY : cmake_force

#=============================================================================
# Set environment variables for the build.

# The shell in which to execute make rules.
SHELL = /bin/sh

# The CMake executable.
CMAKE_COMMAND = /usr/bin/cmake

# The command to remove a file.
RM = /usr/bin/cmake -E remove -f

# Escaping for special characters.
EQUALS = =

# The top-level source directory on which CMake was run.
CMAKE_SOURCE_DIR = /home/james/Dropbox/OSS/Labs/Lab-Example-CMake

# The top-level build directory on which CMake was run.
CMAKE_BINARY_DIR = /home/james/Dropbox/OSS/Labs/Lab-Example-CMake

#=============================================================================
# Targets provided globally by CMake.

# Special rule for the target rebuild_cache
rebuild_cache:
	@$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --cyan "Running CMake to regenerate build system..."
	/usr/bin/cmake -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR)
.PHONY : rebuild_cache

# Special rule for the target rebuild_cache
rebuild_cache/fast: rebuild_cache

.PHONY : rebuild_cache/fast

# Special rule for the target edit_cache
edit_cache:
	@$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --cyan "No interactive CMake dialog available..."
	/usr/bin/cmake -E echo No\ interactive\ CMake\ dialog\ available.
.PHONY : edit_cache

# Special rule for the target edit_cache
edit_cache/fast: edit_cache

.PHONY : edit_cache/fast

# The main all target
all: cmake_check_build_system
	$(CMAKE_COMMAND) -E cmake_progress_start /home/james/Dropbox/OSS/Labs/Lab-Example-CMake/CMakeFiles /home/james/Dropbox/OSS/Labs/Lab-Example-CMake/CMakeFiles/progress.marks
	$(MAKE) -f CMakeFiles/Makefile2 all
	$(CMAKE_COMMAND) -E cmake_progress_start /home/james/Dropbox/OSS/Labs/Lab-Example-CMake/CMakeFiles 0
.PHONY : all

# The main clean target
clean:
	$(MAKE) -f CMakeFiles/Makefile2 clean
.PHONY : clean

# The main clean target
clean/fast: clean

.PHONY : clean/fast

# Prepare targets for installation.
preinstall: all
	$(MAKE) -f CMakeFiles/Makefile2 preinstall
.PHONY : preinstall

# Prepare targets for installation.
preinstall/fast:
	$(MAKE) -f CMakeFiles/Makefile2 preinstall
.PHONY : preinstall/fast

# clear depends
depend:
	$(CMAKE_COMMAND) -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 1
.PHONY : depend

#=============================================================================
# Target rules for targets named shared

# Build rule for target.
shared: cmake_check_build_system
	$(MAKE) -f CMakeFiles/Makefile2 shared
.PHONY : shared

# fast build rule for target.
shared/fast:
	$(MAKE) -f CMakeFiles/shared.dir/build.make CMakeFiles/shared.dir/build
.PHONY : shared/fast

#=============================================================================
# Target rules for targets named libshared

# Build rule for target.
libshared: cmake_check_build_system
	$(MAKE) -f CMakeFiles/Makefile2 libshared
.PHONY : libshared

# fast build rule for target.
libshared/fast:
	$(MAKE) -f CMakeFiles/libshared.dir/build.make CMakeFiles/libshared.dir/build
.PHONY : libshared/fast

#=============================================================================
# Target rules for targets named libstatic

# Build rule for target.
libstatic: cmake_check_build_system
	$(MAKE) -f CMakeFiles/Makefile2 libstatic
.PHONY : libstatic

# fast build rule for target.
libstatic/fast:
	$(MAKE) -f CMakeFiles/libstatic.dir/build.make CMakeFiles/libstatic.dir/build
.PHONY : libstatic/fast

#=============================================================================
# Target rules for targets named static

# Build rule for target.
static: cmake_check_build_system
	$(MAKE) -f CMakeFiles/Makefile2 static
.PHONY : static

# fast build rule for target.
static/fast:
	$(MAKE) -f CMakeFiles/static.dir/build.make CMakeFiles/static.dir/build
.PHONY : static/fast

program.o: program.c.o

.PHONY : program.o

# target to build an object file
program.c.o:
	$(MAKE) -f CMakeFiles/shared.dir/build.make CMakeFiles/shared.dir/program.c.o
	$(MAKE) -f CMakeFiles/static.dir/build.make CMakeFiles/static.dir/program.c.o
.PHONY : program.c.o

program.i: program.c.i

.PHONY : program.i

# target to preprocess a source file
program.c.i:
	$(MAKE) -f CMakeFiles/shared.dir/build.make CMakeFiles/shared.dir/program.c.i
	$(MAKE) -f CMakeFiles/static.dir/build.make CMakeFiles/static.dir/program.c.i
.PHONY : program.c.i

program.s: program.c.s

.PHONY : program.s

# target to generate assembly for a file
program.c.s:
	$(MAKE) -f CMakeFiles/shared.dir/build.make CMakeFiles/shared.dir/program.c.s
	$(MAKE) -f CMakeFiles/static.dir/build.make CMakeFiles/static.dir/program.c.s
.PHONY : program.c.s

source/block.o: source/block.c.o

.PHONY : source/block.o

# target to build an object file
source/block.c.o:
	$(MAKE) -f CMakeFiles/libshared.dir/build.make CMakeFiles/libshared.dir/source/block.c.o
	$(MAKE) -f CMakeFiles/libstatic.dir/build.make CMakeFiles/libstatic.dir/source/block.c.o
.PHONY : source/block.c.o

source/block.i: source/block.c.i

.PHONY : source/block.i

# target to preprocess a source file
source/block.c.i:
	$(MAKE) -f CMakeFiles/libshared.dir/build.make CMakeFiles/libshared.dir/source/block.c.i
	$(MAKE) -f CMakeFiles/libstatic.dir/build.make CMakeFiles/libstatic.dir/source/block.c.i
.PHONY : source/block.c.i

source/block.s: source/block.c.s

.PHONY : source/block.s

# target to generate assembly for a file
source/block.c.s:
	$(MAKE) -f CMakeFiles/libshared.dir/build.make CMakeFiles/libshared.dir/source/block.c.s
	$(MAKE) -f CMakeFiles/libstatic.dir/build.make CMakeFiles/libstatic.dir/source/block.c.s
.PHONY : source/block.c.s

# Help Target
help:
	@echo "The following are some of the valid targets for this Makefile:"
	@echo "... all (the default if no target is provided)"
	@echo "... clean"
	@echo "... depend"
	@echo "... rebuild_cache"
	@echo "... shared"
	@echo "... edit_cache"
	@echo "... libshared"
	@echo "... libstatic"
	@echo "... static"
	@echo "... program.o"
	@echo "... program.i"
	@echo "... program.s"
	@echo "... source/block.o"
	@echo "... source/block.i"
	@echo "... source/block.s"
.PHONY : help



#=============================================================================
# Special targets to cleanup operation of make.

# Special rule to run CMake to check the build system integrity.
# No rule that depends on this can have commands that come from listfiles
# because they might be regenerated.
cmake_check_build_system:
	$(CMAKE_COMMAND) -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 0
.PHONY : cmake_check_build_system
```
- Executable sizes:
```
Static: 8464 bytes
Shared: 8296 bytes
```
