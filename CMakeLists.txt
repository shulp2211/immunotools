project(igrec)
cmake_minimum_required(VERSION 2.8)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithAsserts" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

add_subdirectory(configs)
add_subdirectory(src)

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "/usr/local" CACHE PATH "default install path" FORCE)
endif()

# Expand tildas and convert to absolute path
FILE(TO_CMAKE_PATH "${CMAKE_INSTALL_PREFIX}" CMAKE_INSTALL_PREFIX)

set(INSTALL_DIR "share/igrec")

install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/VERSION"
              "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE"
              "${CMAKE_CURRENT_SOURCE_DIR}/README"
              "${CMAKE_CURRENT_SOURCE_DIR}/GPLv2.txt"
              "${CMAKE_CURRENT_SOURCE_DIR}/init.py"
        DESTINATION "${INSTALL_DIR}"
        COMPONENT runtime)

install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/manual.html"
              "${CMAKE_CURRENT_SOURCE_DIR}/diversity_analyzer_manual.html"
              "${CMAKE_CURRENT_SOURCE_DIR}/dsf_manual.html"
              "${CMAKE_CURRENT_SOURCE_DIR}/barcodedIgReC_manual.html"
              "${CMAKE_CURRENT_SOURCE_DIR}/igquast_manual.html"
        DESTINATION "${INSTALL_DIR}"
        COMPONENT manual)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/docs"
        DESTINATION "${INSTALL_DIR}"
        COMPONENT manual
        PATTERN "*.pdf" EXCLUDE)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/test_dataset"
                  "${CMAKE_CURRENT_SOURCE_DIR}/data"
                  "${CMAKE_CURRENT_SOURCE_DIR}/pipeline_makefiles"
                  "${CMAKE_CURRENT_SOURCE_DIR}/configs"
        DESTINATION "${INSTALL_DIR}"
        COMPONENT runtime
        PATTERN "*.template" EXCLUDE
        PATTERN "CMakeLists.txt" EXCLUDE
        PATTERN ".gitignore" EXCLUDE)

install(DIRECTORY "${CMAKE_BINARY_DIR}/bin"
        DESTINATION "${INSTALL_DIR}/build/release"
        COMPONENT runtime
        USE_SOURCE_PERMISSIONS
        PATTERN "test_*" EXCLUDE)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/py"
        DESTINATION "${INSTALL_DIR}"
        COMPONENT runtime
        USE_SOURCE_PERMISSIONS
        PATTERN "*.pyc" EXCLUDE
        PATTERN ".gitignore" EXCLUDE)

set(tools "igrec.py;dense_subgraph_finder.py;igquast.py;barcoded_igrec.py;diversity_analyzer.py;ig_simulator.py")
foreach(tool ${tools})
    install(PROGRAMS ${tool}
            DESTINATION "${INSTALL_DIR}"
            COMPONENT runtime)
endforeach()

foreach(tool ${tools})
    # Conctruct fake symlinks
    EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E create_symlink "../share/igrec/${tool}" "${CMAKE_BINARY_DIR}/${tool}")
    # ...and add them into the installation
    # I know, I know, I definitely know how much dirty it is
    # Unfortunately, cpack cannot trace symlinks automatically and cmake-install does not support smth like install(SYMLINK)
    install(FILES "${CMAKE_BINARY_DIR}/${tool}"
            DESTINATION "bin"
            COMPONENT runtime)
endforeach()
