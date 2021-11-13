```cmake
enable_testing()
```

```cmake
# function combines 'add_executable' and 'add_test' functions
function(add_executable_and_test test_name)
    # ${ARGN} : all optional arguments following the 'test_name', for source files
    add_executable(${test_name} ${ARGN})
    add_test(
        NAME ${test_name}
        COMMAND ${test_name}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
endfunction()


# function collecting all unit test targets, created by 'add_test'
# to use them in 'add_custom_target::check' target
function(get_all_test_targets var)
    macro(get_all_targets_recursive targets dir)
        get_property(subdirectories DIRECTORY ${dir} PROPERTY SUBDIRECTORIES)
        foreach(subdir ${subdirectories})
        get_all_targets_recursive(${targets} ${subdir})
        endforeach()

        get_property(current_targets DIRECTORY ${dir} PROPERTY TESTS)
        list(APPEND ${targets} ${current_targets})
    endmacro()

    set(targets)
    get_all_targets_recursive(targets ${CMAKE_CURRENT_SOURCE_DIR})
    set(${var} ${targets} PARENT_SCOPE)
endfunction()


# function collecting all targets in current CMakeList and calls 'target_link_libraries' for them
function(link_all_current_libraries)
    get_property(current_targets DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY BUILDSYSTEM_TARGETS)

    foreach(test_target ${current_targets})
        target_link_libraries(${test_target} ${ARGN})
    endforeach()
endfunction()
```
...

```cmake

add_subdirectory(sub_dir)

get_all_test_targets(all_test_targets)

add_custom_target(
    check
    COMMAND CTEST_OUTPUT_ON_FAILURE=1 GTEST_COLOR=1 ${CMAKE_CTEST_COMMAND}
    DEPENDS
        ${all_test_targets}  # all unit tests are here
)

```
