# Wrapper for the [Tuya IoT Core SDK for Embedded C](https://developer.tuya.com/en/docs/iot/link-sdk?id=Kaiuyfihfgkr5) SDK

This wrapper modifies the original [Tuya SDK](https://github.com/tuya/tuya-iot-core-sdk) so it conveniently can be used as single library for own projects / targets.

## The modifications
The following modifications/enhancements are applied to the original Tuya SDK:

- the [examples](https://github.com/tuya/tuya-iot-core-sdk/tree/main/examples) from the SDK [aren't built](CMakeLists.txt#L8-L11) with the ALL target \
  (we are only interested in the Tuya SDK as library).
- the [platform/posix layer](https://github.com/tuya/tuya-iot-core-sdk/tree/main/platform/posix) is only included
  in the Tuya SDK for native build. This enables the possibilty to cross compile the Tuya SDK. \
  For cross compiling only [network_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/network_wrapper.c) is [kept](CMakeLists.txt#L14-L22) as it connects the Tuya SDK to mbedtls and we want to keep it. \
  More information about cross compiling [later in this document](README.md#cross-compiling).
- an overall library named [tuya-iot-core-sdk](CMakeLists.txt#L58-L78) is build from the original Tuya SDK libraries
   - [link_core](https://github.com/tuya/tuya-iot-core-sdk/blob/main/src/CMakeLists.txt#L6-L9)
   - [utils_modules](https://github.com/tuya/tuya-iot-core-sdk/blob/main/utils/CMakeLists.txt#L5-L7)
   - [platform_port](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/CMakeLists.txt#L4-L10)
   - [middleware_implementation](https://github.com/tuya/tuya-iot-core-sdk/blob/main/middleware/CMakeLists.txt#L5-L10)
- This overall library `tuya-iot-core-sdk` can be [installed](CMakeLists.txt#L80-L91). \
  Pass [CMAKE_INSTALL_PREFIX](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) when configuring this project to set the tuya-iot-core-sdk library/header installation directory of your choice.


## Usage

As we now have a single Tuya SDK library, we can conveniently use and embed it to our project

1. as git submodule in your project and [FetchContent_Declare(<this folder>) / FetchContent_MakeAvailable()](https://cmake.org/cmake/help/latest/module/FetchContent.html):
	```
	make_minimum_required( VERSION 3.11.0 )
	project( my_project LANGUAGES C )

	include( FetchContent )

	# add Tuya IoT Core SDK
	FetchContent_Declare( tuya-iot-core-sdk
	    SOURCE_DIR ${CMAKE_SOURCE_DIR}/tuya-iot-core-sdk
	)
	FetchContent_MakeAvailable( tuya-iot-core-sdk )

	# my project as executable
	# (we can simply use / start from a copy of a Tuya SDK example source file we copy to our local project
	#  e.g. tuya-iot-core-sdk/examples/data_model_basic_demo/data_model_basic_demo.c)
	add_executable( my_project
	    data_model_basic_demo.c
	)

	# link Tuya SDK as dependency to my project
	add_dependencies( my_project tuya-iot-core-sdk )
	target_link_libraries( my_project tuya-iot-core-sdk )
	```

2. directly by [FetchContent_Declare(<this folder>) / FetchContent_MakeAvailable()](https://cmake.org/cmake/help/latest/module/FetchContent.html) (populate at configure time):
	```
	make_minimum_required( VERSION 3.11.0 )
	project( my_project LANGUAGES C )

	include( FetchContent )

	# add Tuya IoT Core SDK
	FetchContent_Declare( tuya-iot-core-sdk
	    GIT_REPOSITORY https://github.com/gcopoix/tuya-iot-core-sdk.git
	    GIT_TAG        <commit hash>
	)
	FetchContent_MakeAvailable( tuya-iot-core-sdk )

	# my project as executable
	# (we can simply use / start from a copy of a Tuya SDK example source file we copy to our local project
	#  e.g. tuya-iot-core-sdk/examples/data_model_basic_demo/data_model_basic_demo.c)
	add_executable( my_project
	    data_model_basic_demo.c
	)

	# link Tuya SDK as dependency to my project
	add_dependencies( my_project tuya-iot-core-sdk )
	target_link_libraries( my_project tuya-iot-core-sdk )
	```
3. as git submodule in your project and [add_subdirectory(<this folder>)](https://cmake.org/cmake/help/latest/command/add_subdirectory.html):
	```
	make_minimum_required( VERSION 3.5.0 )
	project( my_project LANGUAGES C )

	# add Tuya IoT Core SDK
	add_subdirectory( tuya-iot-core-sdk )

	# my project as executable
	# (we can simply use / start from a copy of a Tuya SDK example source file we copy to our local project
	#  e.g. tuya-iot-core-sdk/examples/data_model_basic_demo/data_model_basic_demo.c)
	add_executable( my_project
	    data_model_basic_demo.c
	)

	# link Tuya SDK as dependency to my project
	add_dependencies( my_project tuya-iot-core-sdk )
	target_link_libraries( my_project tuya-iot-core-sdk )
	```

## Cross Compiling
With the modifications above the Tuya SDK library now can be cross compiled. \
To fulfill the dependencies for a cross compiled Tuya SDK please implement the platform wrappers for your own target:
- [system_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/system_wrapper.c)
  (implementing [system_interface.h](https://github.com/tuya/tuya-iot-core-sdk/blob/main/interface/system_interface.h))
- [storage_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/storage_wrapper.c)
  (implementing [storage_interface.h](https://github.com/tuya/tuya-iot-core-sdk/blob/main/interface/storage_interface.h))
- If using [mbedtls](https://github.com/tuya/tuya-iot-core-sdk/tree/main/libraries/mbedtls) from the Tuya SDK (preferred option):
   - [mbedtls_sockets_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/mbedtls_sockets_wrapper.c) implementing [mbedtls/net_sockets.h](https://github.com/tuya/tuya-iot-core-sdk/blob/main/libraries/mbedtls/include/mbedtls/net_sockets.h) \
     ([network_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/network_wrapper.c) from the Tuya SDK is still part of this library as it connects the Tuya SDK to mbedtls)
- if using an own TLS implementation:
   - own [network_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/network_wrapper.c) implementing [network_interface.h](https://github.com/tuya/tuya-iot-core-sdk/blob/main/interface/network_interface.h) \
     (no `mbedtls_sockets_wrapper.c` required, no mbedtls used from the Tuya SDK in this case)

An ESP-IDF wrapper implementation is mentioned e.g. [here](https://github.com/tuya/tuya-iot-core-sdk/issues/12)

More information about cross compiling in the [cmake documentation](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html).
