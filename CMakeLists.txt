cmake_minimum_required(VERSION 3.6)
project(endless_sky)

# Set version numbers to match repo
set(VERSION_MAJOR 0)
set(VERSION_MINOR 9)
set(VERSION_PATCH 4)

# Set flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
# set(CMAKE_VERBOSE_MAKEFILE on)

# Define source files
file(GLOB SOURCE_FILES "${CMAKE_SOURCE_DIR}/source/*.h" "${CMAKE_SOURCE_DIR}/source/*.cpp" )
add_executable(endless_sky ${SOURCE_FILES})

# Set cmake module path
if(APPLE)
    set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake_modules;${CMAKE_MODULE_PATH}")
endif()

# Define build path
if(CMAKE_CXX_FLAGS_DEBUG)
    set(BUILD_PATH "${CMAKE_SOURCE_DIR}/cmake-build-debug")
else()
    set(BUILD_PATH "${CMAKE_SOURCE_DIR}/cmake-build-release")
endif()

# Set Endless Sky's Windows Dev-64 and Include Paths
if(WIN32)
    # Download the 64-bit dev libraries (http://endless-sky.github.io/win64-dev.zip) and note the location here
    set(ENDLESS_SKY_DEV64 "C:/Projects/andyspurgeon/EndlessSky/endless-sky-win64-dev")

    include_directories("${ENDLESS_SKY_DEV64}/include")
    include_directories("${ENDLESS_SKY_DEV64}/include/AL")
    include_directories("${ENDLESS_SKY_DEV64}/include/GL")
    include_directories("${ENDLESS_SKY_DEV64}/include/SDL2")
endif()

# mingw32 (Windows Only)
if(WIN32)
    add_library(_mingw UNKNOWN IMPORTED)
    set_property(TARGET _mingw PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libmingw32.a")
endif()

# png
if(WIN32)
    add_library(_png UNKNOWN IMPORTED)
    set_property(TARGET _png PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libpng.dll.a")
elseif(APPLE)
    find_package(PNG REQUIRED)
    include_directories("/usr/local/include/libpng16")
endif()

# jpeg (Windows Only)
if(WIN32)
    add_library(_jpeg UNKNOWN IMPORTED)
    set_property(TARGET _jpeg PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libjpeg.dll.a")
endif()

# turbojpeg
if(WIN32)
    add_library(_turbojpeg UNKNOWN IMPORTED)
    set_property(TARGET _turbojpeg PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libturbojpeg.dll.a")
elseif(APPLE)
    # Since homebrew installs libjpeg-turbo as "keg" (to prevent libjpeg conflicts), set include/lib folders manually
    set(LibJpegTurbo_INCLUDE_DIR "/usr/local/opt/jpeg-turbo/include/")
    set(LibJpegTurbo_LIBRARY "/usr/local/opt/jpeg-turbo/lib/libturbojpeg.a")
    find_package(LibJpegTurbo)
    include_directories(${LibJpegTurbo_INCLUDE_DIR})
endif()

# SDL2
if(WIN32)
    add_library(_sdl2main UNKNOWN IMPORTED)
    set_property(TARGET _sdl2main PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libsdl2main.a")
    add_library(_sdl2 UNKNOWN IMPORTED)
    set_property(TARGET _sdl2 PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libsdl2.dll.a")
elseif(APPLE)
    find_package(SDL2)
    include_directories(${SDL2_INCLUDE_DIR})
endif()

# OpenGL
if(WIN32)
    add_library(_opengl UNKNOWN IMPORTED)
    set_property(TARGET _opengl PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libopengl32.a")
elseif(APPLE)
    find_package(OpenGL)
    include_directories(${OPENGL_INCLUDE_DIR})
endif()

# GLEW (Windows Only)
if(WIN32)
    add_library(_glew UNKNOWN IMPORTED)
    set_property(TARGET _glew PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libglew32.dll.a")
endif()

# OpenAL
if(WIN32)
    add_library(_openal UNKNOWN IMPORTED)
    set_property(TARGET _openal PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libopenal32.dll.a")
elseif(APPLE)
    find_package(OpenAL)
    include_directories(${OPENAL_INCLUDE_DIR})
endif()

# LibMad
if(WIN32)
    add_library(_libmad UNKNOWN IMPORTED)
    set_property(TARGET _libmad PROPERTY IMPORTED_LOCATION "${ENDLESS_SKY_DEV64}/lib/libmad.dll.a")
elseif(APPLE)
    find_package(LIBMAD)
    include_directories(${LIBMAD_INCLUDE_DIR})
endif()

# Link Libraries
if(WIN32)
    # See http://tech.yipp.ca/sdl/how-to-fix-libsdla-undefined-reference/ for reasons for "-lwinmm -lgdi32 -ldxguid"
    target_link_libraries(endless_sky _mingw _png _jpeg _turbojpeg _sdl2main _sdl2 _opengl _libmad _glew _openal -lwinmm -lgdi32 -ldxguid)
elseif(APPLE)
    target_link_libraries(endless_sky ${PNG_LIBRARIES} ${LibJpegTurbo_LIBRARY} ${SDL2_LIBRARY} ${OPENGL_LIBRARIES} ${OPENAL_LIBRARY} ${LIBMAD_LIBRARY})
endif()

# Copy Folders/Files Needed for Execution
#[[
set(ES_BUILD_OUTPUT_FOLDER "${endless_sky_SOURCE_DIR}/build")
if(CMAKE_CXX_FLAGS_DEBUG)
    # Copy data/images/sounds folders
    message(WARNING "Copying 'data', 'images', and 'sounds' folders to: ${ES_BUILD_OUTPUT_FOLDER}/Debug/")
    file(COPY "${endless_sky_SOURCE_DIR}/data" DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Debug/data")
    file(COPY "${endless_sky_SOURCE_DIR}/images" DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Debug/images")
    file(COPY "${endless_sky_SOURCE_DIR}/sounds" DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Debug/sounds")
    # On windows, copy DLLs
    if(WIN32)
        message(WARNING "Copying DLLs to: ${ES_BUILD_OUTPUT_FOLDER}/Debug/")
        file(GLOB WIN_DLL "${ENDLESS_SKY_DEV64}/bin/*.dll")
        file(COPY ${WIN_DLL} DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Debug/")
    endif()
elseif(CMAKE_CXX_FLAGS_RELEASE)
    # Copy data/images/sounds folders
    message(WARNING "Copying 'data', 'images', and 'sounds' folders to: ${ES_BUILD_OUTPUT_FOLDER}/Release/")
    file(COPY "${endless_sky_SOURCE_DIR}/data" DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Release/data")
    file(COPY "${endless_sky_SOURCE_DIR}/images" DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Release/images")
    file(COPY "${endless_sky_SOURCE_DIR}/sounds" DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Release/sounds")
    # On windows, copy DLLs
    if(WIN32)
        message(WARNING "Copying DLLs to: ${ES_BUILD_OUTPUT_FOLDER}/Release/")
        file(GLOB WIN_DLL "${ENDLESS_SKY_DEV64}/bin/*.dll")
        file(COPY ${WIN_DLL} DESTINATION "${ES_BUILD_OUTPUT_FOLDER}/Release/")
    endif()
endif()
]]

# OSX Bundle
if(APPLE)
    # Use template Info.plist in bundle
    set_target_properties(${PROJECT_NAME} PROPERTIES MACOSX_BUNDLE_INFO_PLIST "${CMAKE_MODULE_PATH}/Info.plist.in")

    # Copy assets into the bundle
    set(BUNDLE_PATH "${BUILD_PATH}/${PROJECT_NAME}.app")
    install(DIRECTORY "${CMAKE_SOURCE_DIR}/data/" DESTINATION "${BUNDLE_PATH}/Contents/Resources/data")
    install(DIRECTORY "${CMAKE_SOURCE_DIR}/images/" DESTINATION "${BUNDLE_PATH}/Contents/Resources/images")
    install(DIRECTORY "${CMAKE_SOURCE_DIR}/sounds/" DESTINATION "${BUNDLE_PATH}/Contents/Resources/sounds")

    # TODO: Make the bundle drag-and-drop by installing frameworks..

    # Create custom target (for use in CLion) to install resources, dependant on app itself (builds both)
    add_custom_target(install_${PROJECT_NAME}_bundle
            $(MAKE) install
            DEPENDS ${PROJECT_NAME}
            COMMENT "Installing ${PROJECT_NAME}")
endif()