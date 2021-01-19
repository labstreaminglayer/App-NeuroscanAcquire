cmake_minimum_required(VERSION 3.5)
project(NeuroscanAcquire
	DESCRIPTION "For use https://compumedicsneuroscan.com/ Neuroscan Amplifiers"
	HOMEPAGE_URL "https://github.com/labstreaminglayer/App-NeuroscanAcquire"
	LANGUAGES C CXX
	VERSION 0.2)

find_package(LSL REQUIRED
	HINTS ${LSL_INSTALL_ROOT}
	"${CMAKE_CURRENT_LIST_DIR}/../../LSL/liblsl/build/"
	"${CMAKE_CURRENT_LIST_DIR}/../../LSL/liblsl/build/install"
	"${CMAKE_CURRENT_LIST_DIR}/../../LSL/liblsl/out/build/x64-Release"
	"${CMAKE_CURRENT_LIST_DIR}/../../LSL/liblsl/out/install/x64-Release"
	PATH_SUFFIXES share/LSL
)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
find_package(Qt5 REQUIRED COMPONENTS Widgets Network)

find_package(Threads REQUIRED)

add_executable(${PROJECT_NAME} MACOSX_BUNDLE WIN32
	main.cpp
	widget.cpp
	widget.h
	widget.ui
	nsatransporter.h
	nsatransporter.cpp
)
target_link_libraries(${PROJECT_NAME}
	PRIVATE
	Qt5::Widgets
	Qt5::Network
	Threads::Threads
	LSL::lsl
)
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)
# target_compile_features(${PROJECT_NAME} PRIVATE cxx_auto_type cxx_lambda_init_captures)

installLSLApp(${PROJECT_NAME})

if (WIN32)
	get_target_property(QT5_QMAKE_EXECUTABLE Qt5::qmake IMPORTED_LOCATION)
	get_filename_component(QT5_WINDEPLOYQT_EXECUTABLE ${QT5_QMAKE_EXECUTABLE} PATH)
	set(QT5_WINDEPLOYQT_EXECUTABLE "${QT5_WINDEPLOYQT_EXECUTABLE}/windeployqt.exe")

	add_custom_command(
		TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${QT5_WINDEPLOYQT_EXECUTABLE} --qmldir
			${CMAKE_SOURCE_DIR} $<TARGET_FILE_DIR:${PROJECT_NAME}>)
	add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  
			$<TARGET_FILE:LSL::lsl>
			$<TARGET_FILE_DIR:${PROJECT_NAME}>)
endif(WIN32)

LSLGenerateCPackConfig()
