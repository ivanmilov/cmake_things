## Create a custom target to check changes and generate set of files

<details open><summary>main CMakelists.txt</summary>

```cmake
add_executable(my_targer main.cpp)
target_link_libraries(my_targer cxxopts pthread ...)

add_dependencies(my_targer generate_test_config)
```
</details>

<details open><summary>test CMakelists.txt</summary>

```cmake
...

add_executable(config_test config_test.cpp)
target_link_libraries(config_test gtest gtest_main)

add_executable(tcp_socket tcp_socket_test.cpp)
target_link_libraries(tcp_socket gtest gtest_main)

add_test(
    NAME config_test
    COMMAND config_test
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

add_test(
    NAME tcp_socket
    COMMAND tcp_socket
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

# just run 'make check' to run all tests specified in 'DEPENDS' (will build all the dependencies automatically)
add_custom_target(
    check
    COMMAND CTEST_OUTPUT_ON_FAILURE=1 GTEST_COLOR=1 ${CMAKE_CTEST_COMMAND}
    DEPENDS
            generate_test_config # see test_config_generator
            config_test
            tcp_socket)

```
</details>

<details open><summary>generate_test_config CMakelists.txt</summary>

```cmake
# TLTR:
# If there is no test config in tests/config/ they will be generated out of the corresponding config_template.hjson
# If any config_template.hjson has been changed -> test configs will be regenerated

# Collect all config template files to set dependence for config generation command
FILE(GLOB_RECURSE config_templates
    CONFIGURE_DEPENDS
    ${CMAKE_SOURCE_DIR}/configs/*_template.hjson
)

SET(config_directories "")
FOREACH(tmpl ${config_templates})
    get_filename_component(dir ${tmpl} DIRECTORY)
    LIST(APPEND config_directories ${dir})
ENDFOREACH()

SET(generated_configs "")
FOREACH(dir ${config_directories})
    LIST(APPEND generated_configs ${dir}/config.json)
ENDFOREACH()

# Generate configs out of template files (to have properly generated "source" entity)
add_custom_command(
    OUTPUT ${generated_configs}
    COMMENT "Generating test configs"
    DEPENDS ${config_templates} # configs will be regenerated if template has changed
)
FOREACH(dir ${config_directories})
    add_custom_command(
        OUTPUT ${generated_configs}
        APPEND COMMAND python3 ${CMAKE_SOURCE_DIR}/tests/generate_config_files.py -p -l -i ${dir}
    )
ENDFOREACH()

add_custom_target(
    generate_test_config
    DEPENDS ${generated_configs}
)

```
</details>
