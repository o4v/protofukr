cmake_minimum_required(VERSION 3.22)
project(protofukr VERSION 1.0.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if(MSVC)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")   # no VC++ redistributable needed
endif()

# JUCE: use a local checkout if given (-DJUCE_DIR=...), otherwise download it.
set(JUCE_DIR "" CACHE PATH "Path to a JUCE checkout (optional)")
if(JUCE_DIR AND EXISTS "${JUCE_DIR}/CMakeLists.txt")
    add_subdirectory("${JUCE_DIR}" JUCE)
else()
    include(FetchContent)
    FetchContent_Declare(JUCE
        GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
        GIT_TAG 8.0.4
        GIT_SHALLOW TRUE)
    FetchContent_MakeAvailable(JUCE)
endif()

option(PROTOFUKR_TESTS "Build the offline DSP test runner" OFF)

juce_add_plugin(protofukr
    COMPANY_NAME "protofukr"
    BUNDLE_ID "com.protofukr.protofukr"
    PLUGIN_MANUFACTURER_CODE Pfkr
    PLUGIN_CODE Pfk1
    FORMATS VST3
    PRODUCT_NAME "protofukr"
    IS_SYNTH FALSE
    NEEDS_MIDI_INPUT FALSE
    NEEDS_MIDI_OUTPUT FALSE
    IS_MIDI_EFFECT FALSE
    EDITOR_WANTS_KEYBOARD_FOCUS TRUE
    VST3_CATEGORIES Fx Distortion
    COPY_PLUGIN_AFTER_BUILD FALSE)

juce_add_binary_data(protofukr_data SOURCES
    Resources/DelaGothicOne-Latin.ttf
    Resources/VT323-Regular.ttf)

set(PF_SOURCES
    Source/Parameters.cpp
    Source/PluginProcessor.cpp
    Source/PluginEditor.cpp)

target_sources(protofukr PRIVATE ${PF_SOURCES})
target_include_directories(protofukr PRIVATE Source)

set(PF_DEFS
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_CAN_REPLACE_VST2=0
    JUCE_USE_MP3AUDIOFORMAT=1
    JUCE_DISPLAY_SPLASH_SCREEN=0
    JUCE_REPORT_APP_USAGE=0)
target_compile_definitions(protofukr PUBLIC ${PF_DEFS})

target_link_libraries(protofukr
    PRIVATE
        protofukr_data
        juce::juce_audio_processors
        juce::juce_audio_formats
        juce::juce_dsp
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags)

if(PROTOFUKR_TESTS)
    juce_add_console_app(pf_test PRODUCT_NAME "pf_test")
    target_sources(pf_test PRIVATE tests/OfflineTest.cpp ${PF_SOURCES})
    target_include_directories(pf_test PRIVATE Source)
    target_compile_definitions(pf_test PRIVATE ${PF_DEFS} JucePlugin_Name="protofukr" JUCE_STANDALONE_APPLICATION=1 JUCE_MODAL_LOOPS_PERMITTED=1)
    target_link_libraries(pf_test PRIVATE protofukr_data juce::juce_audio_processors juce::juce_audio_formats juce::juce_dsp
        juce::juce_recommended_config_flags)
endif()
