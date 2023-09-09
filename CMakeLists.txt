cmake_minimum_required(VERSION 3.21)


# Read the JSON file. JSON file contains all information regarding project titles and named variables including developers.
file(READ ${CMAKE_CURRENT_SOURCE_DIR}/CMakeSetup.json CMakeSetup_STRING)

# extracting project name and devleoper names from the json file.
string(JSON Project_NAME GET ${CMakeSetup_STRING} 0 ProjectName)
string(JSON Project_DEVELOPER GET ${CMakeSetup_STRING} 0 Developer)
string(JSON Document_Generation GET ${CMakeSetup_STRING} 0 DocumentGeneration)

# setting project and project library variable names.
set(Project ${Project_NAME})
set(ProjectLib ${Project}_lib)

set(version_file "${CMAKE_CURRENT_BINARY_DIR}/version.cpp")
set(source_files "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp")

# settting the project name.
project(${Project} C CXX)



############ Find Package Stuffs########################################################
########################################################################################
# check if openMP compilers are available and link them
find_package(OpenMP 4.0)
if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()


############ CPack and Git Versioning ##################################################
########################################################################################

# Appends the cmake/modules path to MAKE_MODULE_PATH variable.
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})

# setting the directory of the source for doxygen documentation
set(PROJECT_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/src")
set(source_files "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp")


# Make a version file containing the current version from git.
if(EXISTS "${PROJECT_SOURCE_DIR}/versionForSourcePackage.cmake")
  include("${PROJECT_SOURCE_DIR}/versionForSourcePackage.cmake")
else()
  include(GetGitRevisionDescription)
  git_describe(VERSION --tags --dirty=-dirty)
endif()

#parse the version information into pieces.
string(REGEX REPLACE "^v([0-9]+)\\..*" "\\1" VERSION_MAJOR "${VERSION}")
string(REGEX REPLACE "^v[0-9]+\\.([0-9]+).*" "\\1" VERSION_MINOR "${VERSION}")
string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" VERSION_PATCH "${VERSION}")
string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.[0-9]+(.*)" "\\1" VERSION_SHA1 "${VERSION}")
set(VERSION_SHORT "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

set(CMAKE_INSTALL_PREFIX "/${Project_NAME}_${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")





# specifying the header directory
include_directories(
    ${ProjectLib} "${CMAKE_SOURCE_DIR}/src/headers"
    )



# generating executable program by specifying the location of the file

######### Generating Executables #######################################################
########################################################################################
add_executable(
    ${Project} src/main.cpp  ${version_file}
    )

add_subdirectory(src)

target_link_libraries(${Project} PUBLIC ${VTK_LIBRARIES} ${ProjectLib})

 

########### CPack Commands #############################################################
########################################################################################
set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")

# The following will transfer the version from git to the source package.
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${PROJECT_NAME}-${VERSION_SHORT}-Source")

# Exclude the build and .git directory from the source package.
set(CPACK_SOURCE_IGNORE_FILES "${PROJECT_SOURCE_DIR}/.git/;${PROJECT_BINARY_DIR}/;${CPACK_SOURCE_IGNORE_FILES}")


configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/version.cpp.in
                ${CMAKE_CURRENT_BINARY_DIR}/version.cpp)

configure_file(
   ${PROJECT_SOURCE_DIR}/versionForSourcePackage.cmake.in
   ${PROJECT_BINARY_DIR}/versionForSourcePackage.cmake @ONLY)



########################################################################################
# specifying install rules for dependency libraries
# install(DIRECTORY ${PROJECT_SOURCE_DIR}/dependencies/lib DESTINATION .)
#set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")



# specifying install rules for documentation 
string(TOLOWER ${Project_NAME} User_Guide)
include(InstallRequiredSystemLibraries)

# CPACK ruls
string(TIMESTAMP YEAR_MONTH "%Y_%m")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
# set(CPACK_GENERATOR "RPM;DEB;TGZ;TXZ;TZ;STGZ;NSIS;NSIS64") # NSIS 3.0 and above is not available for Ubuntu 18.04
set(CPACK_GENERATOR "NSIS64")
set(CPACK_PACKAGE_NAME ${Project})
set(CPACK_PACKAGE_RELEASE ${YEAR_MONTH})
set(CPACK_PACKAGE_CONTACT ${Project_DEVELOPER})
set(CPACK_PACKAGE_VENDOR "My Company")
set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${VERSION_SHORT}-${CPACK_PACKAGE_RELEASE}-${CMAKE_SYSTEM_PROCESSOR}")
include(CPack)