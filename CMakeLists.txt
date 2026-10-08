# drift, made by swedishsplidney :)

cmake_minimum_required(VERSION 3.20)

if(NOT DEFINED TERNIMAL_WIDTH)
    set(TERMINAL_WIDTH 64)
endif()

if(NOT DEFINED TERMINAL_HEIGHT)
    set(TERMINAL_HEIGHT 28)
endif()

if(NOT DEFINED TEST_FRAMES)
    set(TEST_FRAMES 0)
endif()

if(NOT DEFINED HEADLESS)
    set(HEADLESS OFF)
endif()

set(STATE_FILE "${CMAKE_CURRENT_LIST_DIR}/.engine_state.cmake")
set(RESOLUTION_SCALE 100)

if(EXISTS "${STATE_FILE}")
    include("${STATE_FILE}")
else()
    set(GAME_MODE "PLAYING")
    set(PLAYER_X 3200)
    set(PLAYER_Y 5500)
    set(PLAYER_ANGLE 0)
    set(SPEED 400)
    set(SCORE 0)
    set(SPEED_MOD 100)
endif()

set(TIME_SCALE 100)

if(GAME_MODE STREQUAL "PLAYING")
    if(DEFINED INPUT)
        set(TIME_SCALE 20)

        if(INPUT STREQUAL "A")
            math(EXPR PLAYER_ANGLE "(${PLAYER_ANGLE} - 15 + 360) % 360")
        elseif(INPUT STREQUAL "D")
            math(EXPR PLAYER_ANGLE "(${PLAYER_ANGLE} + 15) % 360")
        endif()
    endif()

    # calc movement distance
    math(EXPR DIST_STEP "((${SPEED} * ${SPEED_MOD}) / 100 * ${TIME_SCALE}) / 100")

    # point step integration
    if(PLAYER_ANGLE EQUAL 0)
        math(EXPR PLAYER_Y "${PLAYER_Y} - ${DIST_STEP}")
    elseif(PLAYER_ANGLE EQUAL 15)
        math(EXPR PLAYER_X "${PLAYER_X} + (${DIST_STEP} / 4)")
        math(EXPR PLAYER_Y "${PLAYER_Y} - ${DIST_STEP}")
    elseif(PLAYER_ANGLE EQUAL 345)
        math(EXPR PLAYER_X "${PLAYER_X} - (${DIST_STEP} / 4)")
        math(EXPR PLAYER_Y "${PLAYER_Y} - ${DIST_STEP}")
    else()
        math(EXPR PLAYER_Y "${PLAYER_Y} - ${DIST_STEP}")
    endif()

    math(EXPR SCORE "${SCORE} + 1")
endif()

file(WRITE "${STATE_FILE}" "
set(GAME_MODE \"${GAME_MODE}\")
set(PLAYER_X \"${PLAYER_X}\")
set(PLAYER_Y \"${PLAYER_Y}\")
set(PLAYER_ANGLE \"${PLAYER_ANGLE}\")
set(SPEED \"${SPEED}\")
set(SCORE \"${SCORE}\")
set(SPEED_MOD \"${SPEED_MOD}\")
")

message(STATUS " -- engine tick -- ")
message(STATUS "score/frame: ${SCORE}")
message(STATUS "heading angle: ${PLAYER_ANGLE}°")
message(STATUS "time dilation: ${TIME_SCALE}%")
message(STATUS "coordinates: X:${PLAYER_X}, Y:${PLAYER_Y}")