if(UNIX)
    find_package(GTest REQUIRED)

    add_executable(${PROJECT_NAME}-test unix_serial_tests.cc)
    target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME} ${GTEST_LIBRARIES})
    add_test(NAME ${PROJECT_NAME}-test COMMAND ${PROJECT_NAME}-test)

    if(NOT APPLE)
        target_link_libraries(${PROJECT_NAME}-test util)
    endif()

    if(NOT APPLE)  # these tests are unreliable on macOS
      add_executable(${PROJECT_NAME}-test-timer unit/unix_timer_tests.cc)
      target_link_libraries(${PROJECT_NAME}-test-timer ${PROJECT_NAME} ${GTEST_LIBRARIES})
      add_test(NAME ${PROJECT_NAME}-test-timer COMMAND ${PROJECT_NAME}-test-timer)
    endif()
endif()
