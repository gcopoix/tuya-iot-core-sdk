# Wrapper for the [Tuya IoT Core SDK for Embedded C](https://developer.tuya.com/en/docs/iot/link-sdk?id=Kaiuyfihfgkr5) SDK

This wrapper modifies the original [Tuya SDK](https://github.com/tuya/tuya-iot-core-sdk) so it can be used as single library for own projects.

## The modifications
The following modifications/enhancements are applied to the original Tuya SDK:

- the [examples](https://github.com/tuya/tuya-iot-core-sdk/tree/main/examples) aren't built with the ALL target \
  (we are only interested in the Tuya SDK as library)
- the [platform/posix layer](https://github.com/tuya/tuya-iot-core-sdk/tree/main/platform/posix) is only included
  in the Tuya SDK for native build. \
  For a [cross compile](CMakeLists.txt#L14-L22) only [network_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/network_wrapper.c) is kept as it connects the Tuya SDK to mbedtls and we want to keep it. \
  More information about cross compiling [later in this document](README.md#cross-compiling). \
  (need to implement an own [mbedtls_sockets_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/mbedtls_sockets_wrapper.c) in your project for your target in this case)
- an [overall library](CMakeLists.txt#L25-L37) named `tuya-iot-core-sdk` is build from the libraries
   - link_core
   - utils_modules
   - platform_port
   - middleware_implementation
- This `overall library` can be installed if build as SHARED_LIBRARY


## Usage

As we now have a single Tuya SDK library, we can conveniently use and embed it to our project by

1. [add_subdirectory(<this folder>)](https://cmake.org/cmake/help/latest/command/add_subdirectory.html):
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

2. [FetchContent_Declare(<this folder>) and FetchContent_MakeAvailable()](https://cmake.org/cmake/help/latest/module/FetchContent.html):
	```
	make_minimum_required( VERSION 3.11.0 )
	project( my_project LANGUAGES C )

	include( FetchContent )

	# add Tuya IoT Core SDK
	FetchContent_Declare( tuya-iot-core-sdk SOURCE_DIR ${CMAKE_SOURCE_DIR}/tuya-iot-core-sdk )
	FetchContent_MakeAvailable( tuya-iot-core-sdk )

	# my project as executable
	# (we can simply use / start from a copy of a Tuya SDK example source file we copy to our local project
	#  e.g. tuya-iot-core-sdk/examples/data_model_basic_demo/data_model_basic_demo.c)
	add_executable( my_project
	    data_model_basic_demo.c
	)

	# link Tuya SDK as dependency to my project
	add_dependencies( mydemo tuya-iot-core-sdk )
	target_link_libraries( mydemo tuya-iot-core-sdk )
	```

## Cross Compiling
With the modifications above the Tuya SDK library now can be cross compiled. \
To fulfil the dependencies for a cross compiled Tuya SDK please implement the platform wrappers for your own target:
- [system_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/system_wrapper.c)
- [storage_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/storage_wrapper.c)
- If using mbedtls from the Tuya SDK:
   - [mbedtls_sockets_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/mbedtls_sockets_wrapper.c) \
     (`network_wrapper.c` from the Tuya SDK is kept as it connects the Tuya SDK to mbedtls)
- if using an own TLS implementation:
   - own [network_wrapper.c](https://github.com/tuya/tuya-iot-core-sdk/blob/main/platform/posix/network_wrapper.c) \
     (no `mbedtls_sockets_wrapper.c` required, no mbedtls required from the Tuya SDK)


An ESP-IDF wrapper implementation is mentioned e.g. [here](https://github.com/tuya/tuya-iot-core-sdk/issues/12)

More information about cross compiling in the [cmake documentation](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html).
