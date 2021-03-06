message(STATUS "Building Unit Tests ${UNITTEST}")

add_custom_command ( OUTPUT test_model_small.h5
                     COMMAND bash -c "python3 ../keras_export/generate_test_models.py small test_model_small.h5"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)

add_custom_command ( OUTPUT test_model_sequential.h5
                     COMMAND bash -c "python3 ../keras_export/generate_test_models.py sequential test_model_sequential.h5"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)

if(NOT DEFINED ENV{TRAVIS}) # Do not use full model on travis since it takes too long.
add_custom_command ( OUTPUT test_model_full.h5
                     COMMAND bash -c "python3 ../keras_export/generate_test_models.py full test_model_full.h5"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)
endif()

add_custom_command ( OUTPUT readme_example_model.h5
                     COMMAND bash -c "python3 ../test/readme_example_generate.py"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)

add_custom_command ( OUTPUT test_model_small.json
                     DEPENDS test_model_small.h5
                     COMMAND bash -c "python3 ../keras_export/convert_model.py test_model_small.h5 test_model_small.json"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)

add_custom_command ( OUTPUT test_model_sequential.json
                     DEPENDS test_model_sequential.h5
                     COMMAND bash -c "python3 ../keras_export/convert_model.py test_model_sequential.h5 test_model_sequential.json"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)

if(NOT DEFINED ENV{TRAVIS})
add_custom_command ( OUTPUT test_model_full.json
                     DEPENDS test_model_full.h5
                     COMMAND bash -c "python3 ../keras_export/convert_model.py test_model_full.h5 test_model_full.json"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)
endif()

add_custom_command ( OUTPUT readme_example_model.json
                     DEPENDS readme_example_model.h5
                     COMMAND bash -c "python3 ../keras_export/convert_model.py readme_example_model.h5 readme_example_model.json"
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/)

find_package(doctest REQUIRED)

macro(_add_test _NAME)
    add_executable(${_NAME} ${_NAME}.cpp)
    add_test(NAME ${_NAME} COMMAND ${_NAME})
    target_include_directories(${_NAME} SYSTEM PUBLIC ${doctest_INCLUDE_DIR})
    target_link_libraries(${_NAME} ${CMAKE_THREAD_LIBS_INIT})
endmacro()

_add_test(test_model_small_test)
_add_test(test_model_sequential_test)
if(NOT DEFINED ENV{TRAVIS})
_add_test(test_model_full_test)
endif()
_add_test(readme_example_main)

if(NOT DEFINED ENV{TRAVIS})
add_custom_target(unittest
    DEPENDS test_model_small.json
    DEPENDS test_model_sequential.json
    DEPENDS test_model_full.json
    DEPENDS readme_example_model.json

    COMMAND test_model_small_test
    COMMAND test_model_sequential_test
    COMMAND test_model_full_test
    COMMAND readme_example_main

    COMMENT "Running unittests\n\n"
    VERBATIM
)
else()
add_custom_target(unittest
    DEPENDS test_model_small.json
    DEPENDS test_model_sequential.json
    DEPENDS readme_example_model.json

    COMMAND test_model_small_test
    COMMAND test_model_sequential_test
    COMMAND readme_example_main

    COMMENT "Running unittests\n\n"
    VERBATIM
)
endif()