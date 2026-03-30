cmake_minimum_required(VERSION 3.22)
project(TwoOpFM VERSION 1.1.0)

# -- Dependencies ----------------------------------------------------------
# Set these paths if JUCE / eurorack are not cloned as siblings of this repo.
#   cmake -B build -DJUCE_DIR=/path/to/JUCE -DEURORACK_DIR=/path/to/eurorack ...
set(JUCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../JUCE" CACHE PATH "Path to JUCE")
set(EURORACK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../eurorack" CACHE PATH "Path to eurorack")

add_subdirectory(${JUCE_DIR} JUCE)

set(EURORACK ${EURORACK_DIR})

juce_add_plugin(TwoOpFM
    COMPANY_NAME            "CorvidAudio"
    PLUGIN_MANUFACTURER_CODE Cvda
    PLUGIN_CODE             Twop
    FORMATS                 AU VST3 Standalone
    IS_SYNTH                TRUE
    NEEDS_MIDI_INPUT        TRUE
    NEEDS_MIDI_OUTPUT       FALSE
    AU_MAIN_TYPE            kAudioUnitType_MusicDevice
    AU_EXPORT_PREFIX        TwoOpFMAU
    BUNDLE_ID               com.CorvidAudio.TwoOpFM
    PRODUCT_NAME            "2-OP"
)

juce_generate_juce_header(TwoOpFM)

target_sources(TwoOpFM PRIVATE
    src/PluginProcessor.cpp
    src/PluginEditor.cpp
    ${EURORACK}/plaits/dsp/engine/fm_engine.cc
    ${EURORACK}/plaits/resources.cc
    ${EURORACK}/stmlib/dsp/units.cc
)

target_include_directories(TwoOpFM PRIVATE ${EURORACK})

target_compile_definitions(TwoOpFM PUBLIC
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_CAN_REPLACE_VST2=0
    # Bypass ARM-only VFP assembly in stmlib/dsp/dsp.h (Sqrt/Clip16/ClipU16)
    TEST=1
)

target_link_libraries(TwoOpFM PRIVATE
    juce::juce_audio_utils
    juce::juce_audio_devices
    juce::juce_audio_plugin_client
)
