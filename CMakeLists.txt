# v3.14 required for FetchContent_MakeAvailable
cmake_minimum_required(VERSION 3.14)
if (DEFINED MCU)
	include(FetchContent)
	FetchContent_Declare(
		mcu_support
		GIT_REPOSITORY https://github.com/bolderflight/mcu-support.git
		GIT_TAG v1.0.0
		)
	FetchContent_MakeAvailable(mcu_support)

	# Setting up the toolchain
	set(CMAKE_TOOLCHAIN_FILE "${mcu_support_SOURCE_DIR}/cmake/cortex.cmake")

	# Project information
	project(ADS1115
		VERSION 5.6.0
		DESCRIPTION "Teensy ADS1115 sensor driver"
		LANGUAGES CXX
	       )

	# Grab the processor and set up definitions and compile options
	include(${mcu_support_SOURCE_DIR}/cmake/config_mcu.cmake)
	configMcu(${MCU} ${mcu_support_SOURCE_DIR})

	# Fetch core
	FetchContent_Declare(
		core
		GIT_REPOSITORY https://github.com/bolderflight/core.git
		GIT_TAG v3.0.2
		)
	FetchContent_MakeAvailable(core)

	# Fetch units
	FetchContent_Declare(
		units
		GIT_REPOSITORY https://github.com/bolderflight/units.git
		GIT_TAG v4.0.4
		)
	FetchContent_MakeAvailable(units)

	# Fetch eigen
	FetchContent_Declare(
		eigen
		GIT_REPOSITORY https://github.com/bolderflight/eigen.git
		GIT_TAG v3.0.2
		)
	FetchContent_MakeAvailable(eigen)

	# Add the library target
	add_library(ADS1115
		src/Adafruit_ADS1X15.cpp
		src/Adafruit_ADS1X15.h
		src/Adafruit_I2CDevice.h
		src/Adafruit_I2CDevice.cpp
		)

	# Link libraries
	target_link_libraries(ADS1115
		PUBLIC
		core
		units
		eigen
		)

	# Setup include directories
	target_include_directories(ADS1115 PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
		$<INSTALL_INTERFACE:include>
		)


	# Example and test if this project is built separately
	if (PROJECT_NAME STREQUAL CMAKE_PROJECT_NAME)
		# Add the executable target
		add_executable(ads1115_nonblocking_example examples/nonblocking/nonblocking.cc)

		# Add the includes
		target_include_directories(ads1115_nonblocking_example PUBLIC
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
			$<INSTALL_INTERFACE:include>
			)

		# Link libraries to the example target
		target_link_libraries(ads1115_nonblocking_example
			PRIVATE
			ADS1115
			)
		# Add hex and upload targets
		include(${mcu_support_SOURCE_DIR}/cmake/flash_mcu.cmake)
		FlashMcu(ads1115_nonblocking_example ${MCU} ${mcu_support_SOURCE_DIR})
	endif()
endif()
