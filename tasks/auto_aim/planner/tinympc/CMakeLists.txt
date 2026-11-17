add_library(tinympcstatic STATIC
    admm.cpp
    tiny_api.cpp
    rho_benchmark.cpp
)

set_property(TARGET tinympcstatic PROPERTY POSITION_INDEPENDENT_CODE ON)

# target_link_libraries(tinympcstatic PUBLIC Eigen)
target_include_directories(tinympcstatic PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/..)
target_include_directories(tinympcstatic PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../../include/Eigen)



