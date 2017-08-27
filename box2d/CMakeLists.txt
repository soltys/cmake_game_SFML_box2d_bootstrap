file(GLOB_RECURSE box2d_SOURCES "src/*.cpp")
include_directories("include")
add_library(box2d ${box2d_SOURCES})

