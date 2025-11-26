cmake_minimum_required(VERSION 3.10)
project(flex_tests LANGUAGES C)

find_package(FLEX REQUIRED)

function(add_flex_test TESTNAME LEXFILE INPUTFILE)
    set(TEST_L "${CMAKE_CURRENT_SOURCE_DIR}/${LEXFILE}")
    set(GENERATED_C "${CMAKE_CURRENT_BINARY_DIR}/${TESTNAME}_lex.yy.c")

    if(WIN32)
        set(WINCOMPAT " --wincompat")
    else()
        set(WINCOMPAT "")
    endif()

    add_custom_command(
        OUTPUT "${GENERATED_C}"
        COMMAND ${FLEX_EXECUTABLE}${WINCOMPAT} -o "${GENERATED_C}" "${TEST_L}"
        MAIN_DEPENDENCY "${TEST_L}"
        COMMENT "Generating ${GENERATED_C} from ${TEST_L}"
        VERBATIM
    )

    add_executable(${TESTNAME} "${GENERATED_C}")
    add_custom_target(${TESTNAME}_generate DEPENDS "${GENERATED_C}")
    add_dependencies(${TESTNAME} ${TESTNAME}_generate)

    # copy the input file into the build dir so tests run from build tree
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${INPUTFILE}"
        "${CMAKE_CURRENT_BINARY_DIR}/${INPUTFILE}"
        COPYONLY)

    enable_testing()
    add_test(NAME ${TESTNAME}_run
        COMMAND ${TESTNAME} "${CMAKE_CURRENT_BINARY_DIR}/${INPUTFILE}")
    set_tests_properties(${TESTNAME}_run PROPERTIES PASS_REGULAR_EXPRESSION "TEST RETURNING OK.")
endfunction()

add_flex_test(basic basic.rules basic.txt)
