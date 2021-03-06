set(mltrtaudio_src consumer_rtaudio.cpp)
set(mltrtaudio_lib mlt Threads::Threads)
set(mltrtaudio_def)
if(TARGET PkgConfig::rtaudio)
    list(APPEND mltrtaudio_lib PkgConfig::rtaudio)
else()
    list(APPEND mltrtaudio_src RtAudio.cpp)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR})
    if(APPLE)
        list(APPEND mltrtaudio_lib CoreAudio CoreFoundation)
        list(APPEND mltrtaudio_def __MACOSX_CORE__)
    elseif(WIN32)
        list(APPEND mltrtaudio_lib ole32 dsound winmm ksuser)
        list(APPEND mltrtaudio_def __WINDOWS_DS__ __WINDOWS_WASAPI__)
    else()
        if(TARGET PkgConfig::alsa)
            list(APPEND mltrtaudio_lib PkgConfig::alsa)
            list(APPEND mltrtaudio_def __LINUX_ALSA__)
        endif()
        if(TARGET PkgConfig::libpulse-simple)
            list(APPEND mltrtaudio_lib PkgConfig::libpulse-simple)
            list(APPEND mltrtaudio_def __LINUX_PULSE__)
        endif()
        if(NOT (TARGET PkgConfig::alsa OR TARGET PkgConfig::libpulse-simple))
            list(APPEND mltrtaudio_lib ossaudio)
            list(APPEND mltrtaudio_def __LINUX_OSS__)
        endif()
    endif()
endif()
add_library(mltrtaudio MODULE ${mltrtaudio_src})
target_link_libraries(mltrtaudio ${mltrtaudio_lib})
target_compile_definitions(mltrtaudio PRIVATE ${mltrtaudio_def})
# Create module in parent directory, for the benefit of "source setenv".
set_target_properties(mltrtaudio PROPERTIES LIBRARY_OUTPUT_DIRECTORY ..)
install(TARGETS mltrtaudio LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/mlt)
file(GLOB yml *.yml)
install(FILES ${yml} DESTINATION ${CMAKE_INSTALL_DATADIR}/mlt/rtaudio)
