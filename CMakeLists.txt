cmake_minimum_required(VERSION 3.8.0)
if(POLICY CMP0092)
  cmake_policy(SET CMP0092 NEW)
endif()

project(pcsc-cpp)

# The libpcsc-cpp library itself.

add_library(${PROJECT_NAME}
  STATIC
  include/${PROJECT_NAME}/${PROJECT_NAME}.hpp
  include/${PROJECT_NAME}/${PROJECT_NAME}-utils.hpp
  include/${PROJECT_NAME}/comp_winscard.hpp
  include/flag-set-cpp/flag_set.hpp
  include/magic_enum/magic_enum.hpp
  src/Context.hpp
  src/Reader.cpp
  src/SCardCall.hpp
  src/SmartCard.cpp
  src/listReaders.cpp
  src/utils.cpp
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
  include
)

target_compile_features(${PROJECT_NAME}
  PUBLIC
  cxx_std_17
)

target_compile_options(${PROJECT_NAME} PRIVATE
  $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX>
  $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -pedantic -Werror>
)

target_compile_definitions(${PROJECT_NAME} PRIVATE
  $<$<CXX_COMPILER_ID:MSVC>:WIN32_LEAN_AND_MEAN;UNICODE>
)

target_link_libraries(${PROJECT_NAME} PRIVATE
  $<$<CXX_COMPILER_ID:MSVC>:Ws2_32>
)

# PC/SC API dependencies.
if(WIN32)
  # no extra includes required
  target_link_libraries(${PROJECT_NAME} PRIVATE winscard)
elseif(APPLE)
  # no extra includes required
  target_link_libraries(${PROJECT_NAME} PRIVATE "-framework PCSC")
else()
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(PCSC libpcsclite REQUIRED IMPORTED_TARGET)
  target_link_libraries(${PROJECT_NAME} PRIVATE PkgConfig::PCSC)
endif()
