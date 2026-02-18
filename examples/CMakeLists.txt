# usage:    cmake -G "MinGW Makefiles" -DBOARD=raspberry_pi_pico -DPICO_SDK_FETCH_FROM_GIT=1 -DFAMILY=rp2040 -DCMAKE_BUILD_TYPE=Release -DPICO_NO_UF2=1 . -Bbuild
#           cmake --build ./build
cmake_minimum_required(VERSION 3.20)
set(CMAKE_TOOLCHAIN_FILE arm-none-eabi-gcc.cmake)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mcpu=cortex-m0plus -mthumb -Wno-error=conversion")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mcpu=cortex-m0plus -mthumb -Wno-error=conversion")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -mcpu=cortex-m0plus -mthumb")

include(${CMAKE_CURRENT_SOURCE_DIR}/../hw/bsp/family_support.cmake)

project(tinyusb_examples C CXX ASM)

set(EXAMPLES_LIST
  device
  dual
  host
  typec
  )
set(MAPJSON_PATTERNS "")

foreach (example ${EXAMPLES_LIST})
  add_subdirectory(${example})
  list(APPEND MAPJSON_PATTERNS "${CMAKE_BINARY_DIR}/${example}/*/*.map.json")
endforeach ()

# Post-build: run metrics.py on all map.json files
find_package(Python3 REQUIRED COMPONENTS Interpreter)
add_custom_target(tinyusb_metrics
  COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/../tools/metrics.py
  combine -f tinyusb/src -j -o ${CMAKE_BINARY_DIR}/metrics
  ${MAPJSON_PATTERNS}
  COMMENT "Generating average code size metrics"
  VERBATIM
  )

#add_custom_command(TARGET tinyusb_metrics POST_BUILD
#  COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/../tools/metrics.py compare ${TOP}/cmake-build/cmake-build-${BOARD}/metrics.json ${CMAKE_BINARY_DIR}/metrics.json
#  COMMENT "Generating average code size metrics"
#  VERBATIM
#  )
