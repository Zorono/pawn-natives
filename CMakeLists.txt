cmake_minimum_required(VERSION 3.19)

if (NOT MAIN_RUN)
		project(dependencies)

		include(ExternalProject)
		include(FetchContent)

		# Get the absolute path and normalize it to avoid junction/space issues
		# Junctions with spaces breaks the build process....
		get_filename_component(BIN_DIR "${CMAKE_BINARY_DIR}" REALPATH)

		ExternalProject_Add(samp-plugin-sdk
				PREFIX "samp-plugin-sdk"
				GIT_REPOSITORY "https://github.com/Y-Less/samp-plugin-sdk.git"
				GIT_TAG "master"
				UPDATE_COMMAND ""
				CONFIGURE_COMMAND ""
				SOURCE_DIR "${BIN_DIR}/external/samp-plugin-sdk-src"
				BINARY_DIR "${BIN_DIR}/external/samp-plugin-sdk-build"
				STAMP_DIR  "${BIN_DIR}/external/samp-plugin-sdk-stamp"
				BUILD_COMMAND ""
				INSTALL_COMMAND ""
				CMAKE_ARGS
				    "-DCMAKE_POLICY_VERSION_MINIMUM=3.5"
		)

		if (NOT WIN32)
		        set(SAMPGDK_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/sampgdk_install")
				ExternalProject_Add(sampgdk
						PREFIX "sampgdk"
						GIT_REPOSITORY "https://github.com/Just-Archives/sampgdk.git"
						GIT_TAG "master"
						UPDATE_COMMAND ""
						SOURCE_DIR "${BIN_DIR}/external/sampgdk-src"
    					BINARY_DIR "${BIN_DIR}/external/sampgdk-build"
    					STAMP_DIR  "${BIN_DIR}/external/sampgdk-stamp"
						CMAKE_ARGS
						    "-DCMAKE_INSTALL_PREFIX=${SAMPGDK_INSTALL_DIR}"
							"-DCMAKE_POLICY_VERSION_MINIMUM=3.5"
							"-DSAMP_SDK_ROOT=${BIN_DIR}/external/samp-plugin-sdk-src/src/samp-plugin-sdk"
							"-DINSTALL_PDB=OFF"
                        BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --config Release
                        INSTALL_COMMAND ""
				)
		endif ()

		FetchContent_Declare(
			cmake_modules
			GIT_REPOSITORY https://github.com/Just-Archives/cmake-modules.git
			GIT_TAG main
			SOURCE_DIR "${BIN_DIR}/external/cmake-modules"
		)
		FetchContent_MakeAvailable(cmake_modules)

		ExternalProject_Add(subhook
				PREFIX "subhook"
				GIT_REPOSITORY "https://github.com/Just-Archives/subhook.git"
				GIT_TAG "main"
				UPDATE_COMMAND ""
				CONFIGURE_COMMAND ""
				SOURCE_DIR "${BIN_DIR}/external/subhook-src"
				BINARY_DIR "${BIN_DIR}/external/subhook-build"
				STAMP_DIR  "${BIN_DIR}/external/subhook-stamp"
				BUILD_COMMAND ""
				INSTALL_COMMAND ""
				CMAKE_ARGS
				    "-DCMAKE_POLICY_VERSION_MINIMUM=3.5"
		)

		if (WIN32)
				# Windows seems to go top-to-bottom.  Linux the other way...
				set(SAMPGDK_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/sampgdk_install")
				ExternalProject_Add(sampgdk
						PREFIX "sampgdk"
						GIT_REPOSITORY "https://github.com/Just-Archives/sampgdk.git"
						GIT_TAG "master"
						UPDATE_COMMAND ""
						SOURCE_DIR "${BIN_DIR}/external/sampgdk-src"
    					BINARY_DIR "${BIN_DIR}/external/sampgdk-build"
    					STAMP_DIR  "${BIN_DIR}/external/sampgdk-stamp"
						CMAKE_ARGS
						    "-DCMAKE_INSTALL_PREFIX=${SAMPGDK_INSTALL_DIR}"
							"-DCMAKE_POLICY_VERSION_MINIMUM=3.5"
							"-DSAMP_SDK_ROOT=${BIN_DIR}/external/samp-plugin-sdk-src/src/samp-plugin-sdk"
							"-DINSTALL_PDB=OFF"
                        BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --config Release
                        INSTALL_COMMAND ""
				)
		endif ()
else ()
		project(pawn-natives)

		# Now that we have them, use them.
		list(APPEND CMAKE_MODULE_PATH ${BIN_DIR}/external/cmake-modules)
		include(AMXConfig)
		include(AddSAMPPlugin)

		include_directories(
				${BIN_DIR}/external/samp-plugin-sdk-src
				${BIN_DIR}/external/samp-plugin-sdk-src/amx
				${BIN_DIR}/external/subhook-src
				${BIN_DIR}/external/sampgdk-src/src/sampgdk/include/
				${BIN_DIR}/external/sampgdk-src/src/sampgdk-build/include/
		)

		add_definitions(-D_CRT_SECURE_NO_WARNINGS)
		add_definitions(-DSUBHOOK_IMPLEMENTATION)
		add_definitions(-DHAVE_STDINT_H=1)

		add_samp_plugin(pawn-natives
				test.cpp
				${BIN_DIR}/external/subhook-src/subhook.c
				${BIN_DIR}/external/samp-plugin-sdk-src/amxplugin.cpp
				${BIN_DIR}/external/samp-plugin-sdk-src/amxplugin2.cpp
		)

		if (WIN32)
				target_link_libraries(pawn-natives
						${BIN_DIR}/external/sampgdk-src/src/sampgdk-build/Release/sampgdk4.lib
				)

				set_target_properties(pawn-natives PROPERTIES LINKER_LANGUAGE CXX)
		else ()
				target_link_libraries(pawn-natives
						${BIN_DIR}/external/sampgdk-src/src/sampgdk-build/libsampgdk.so
				)

				set(CMAKE_CXX_FLAGS "-std=gnu++0x -m32")
		endif ()
endif ()


