cmake_minimum_required(VERSION 3.10)
project(flex_tests LANGUAGES CXX)

if(BUILD_TESTING)
    find_package(FLEX REQUIRED)

    function(add_flex_test TESTNAME LEXFILE INPUTFILE) # MAINFILE
        set(TEST_L "${CMAKE_CURRENT_SOURCE_DIR}/${LEXFILE}")
        set(GENERATED_C "${CMAKE_CURRENT_BINARY_DIR}/${TESTNAME}_lex.yy.c")

        if(WIN32)
            set(WINCOMPAT "--wincompat")
        else()
            set(WINCOMPAT "")
        endif()

        add_custom_command(
            OUTPUT "${GENERATED_C}"
            COMMAND ${FLEX_EXECUTABLE} ${WINCOMPAT} -o "${GENERATED_C}" "${TEST_L}"
            MAIN_DEPENDENCY "${TEST_L}"
            COMMENT "Generating ${GENERATED_C} from ${TEST_L}"
            VERBATIM
        )

        add_executable(${TESTNAME} "${GENERATED_C}") # "${CMAKE_CURRENT_SOURCE_DIR}/${MAINFILE}"
        # target_link_libraries(${TESTNAME} PRIVATE fl)
        target_compile_features(${TESTNAME} PRIVATE cxx_std_11)
        if(MSVC)
            # Undefine macros that break the MSVC C++ headers when compiling C++ sources.
            # Use /U to remove any problematic macro definitions for C++ compilation.
            target_compile_options(${TESTNAME} PRIVATE
                $<$<COMPILE_LANGUAGE:CXX>:/Uinline>
                $<$<COMPILE_LANGUAGE:CXX>:/Urestrict>
                $<$<COMPILE_LANGUAGE:CXX>:/U__extension__>
            )
        endif()
        target_include_directories(${TESTNAME} PRIVATE
            "${CMAKE_SOURCE_DIR}/flex/src"
            "${CMAKE_SOURCE_DIR}/flex/tests"
            "$<$<BOOL:${CMAKE_INSTALL_PREFIX}>:${CMAKE_INSTALL_PREFIX}>"
        )
        add_custom_target(${TESTNAME}_generate DEPENDS "${GENERATED_C}")
        add_dependencies(${TESTNAME} ${TESTNAME}_generate)

        # copy the input file into the build dir so tests run from build tree
        configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${INPUTFILE}"
            "${CMAKE_CURRENT_BINARY_DIR}/${INPUTFILE}"
            COPYONLY)

        add_test(NAME ${TESTNAME}_run
            COMMAND ${TESTNAME} "${CMAKE_CURRENT_BINARY_DIR}/${INPUTFILE}")
        set_tests_properties(${TESTNAME}_run PROPERTIES PASS_REGULAR_EXPRESSION "TEST RETURNING OK.")
    endfunction()

    add_flex_test(cxx_yywrap cxx_yywrap.ll cxx_yywrap.txt)
endif()
