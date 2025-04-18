add_library(ffmpeg.avcodec.config INTERFACE)
add_library(ffmpeg.avcodec.headers INTERFACE)
ff_create_component(avcodec.core)


list(APPEND FFMPEG_AVCODEC_CODEC_LIST)
list(APPEND FFMPEG_AVCODEC_PARSER_LIST)
list(APPEND FFMPEG_AVCODEC_COMPONENTS avcodec.core)


list(APPEND FFMPEG_AVCODEC_CORE_SOURCES
    ac3_parser.c
    adts_parser.c
    allcodecs.c
    avdct.c
    avfft.c
    bitstream.c
    bitstream_filters.c
    bsf.c
    codec_desc.c
    codec_par.c
    decode.c
    dv_profile.c
    encode.c
    get_buffer.c
    imgconvert.c
    mathtables.c
    threadprogress.c
    mediacodec.c
    options.c
    packet.c
    parsers.c
    profiles.c
    qsv_api.c
    raw.c
    refstruct.c
    version.c
    vlc.c
    xiph.c
)

set(FFMPEG_AVCODEC_GENERATED_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(AVCODEC_GENERATED_INCLUDE_DIRECTORY "${FFMPEG_AVCODEC_GENERATED_DIRECTORY}/include")
set(PREPEND_HACK_DIR "${FFMPEG_AVCODEC_GENERATED_DIRECTORY}/prepend_hack")
file(REMOVE_RECURSE ${PREPEND_HACK_DIR})

list(APPEND FFMPEG_AVCODEC_CORE_SOURCES_PREPENDED
    utils.c "#include \"libavutil/internal.h\""
    avcodec.c "#include \"libavutil/internal.h\""
    parser.c "#include \"libavutil/internal.h\""
)

gen_prepend_hack(FFMPEG_AVCODEC_CORE_SOURCES_PREPENDED PREPEND_HACK_DIR "avcodec" FFMPEG_AVCODEC_CORE_SOURCES)

include(avcodec_headers.cmake)


include(h264.cmake)
include(bmp.cmake)


gen_headers(FFMPEG_AVCODEC_HEADERS CMAKE_CURRENT_SOURCE_DIR "${AVCODEC_GENERATED_INCLUDE_DIRECTORY}/libavcodec")

#generating bsf_list.c
ffmpeg_message(WARNING "avcodec" "TODO: generate libavcodec/bsf_list.c more properly")
file(GENERATE OUTPUT "${AVCODEC_GENERATED_INCLUDE_DIRECTORY}/libavcodec/bsf_list.c" CONTENT "const FFBitStreamFilter *bitstream_filters[] = {NULL};")


#generating parser_list.c
string(APPEND FFMPEG_AVCODEC_PARSER_LIST_CONTENT "const AVCodecParser *parser_list[] = {\n")
foreach(AVPARSER_NAME ${FFMPEG_AVCODEC_PARSER_LIST})
    string(APPEND FFMPEG_AVCODEC_PARSER_LIST_CONTENT "&${AVPARSER_NAME}\,\n")
endforeach()
string(APPEND FFMPEG_AVCODEC_PARSER_LIST_CONTENT "NULL\n")
string(APPEND FFMPEG_AVCODEC_PARSER_LIST_CONTENT "}\;\n")
file(GENERATE OUTPUT "${AVCODEC_GENERATED_INCLUDE_DIRECTORY}/libavcodec/parser_list.c" CONTENT ${FFMPEG_AVCODEC_PARSER_LIST_CONTENT})


#generating codec_list.c
string(APPEND FFMPEG_AVCODEC_CODEC_LIST_CONTENT "const FFCodec *codec_list[] = {\n")
foreach(AVCODEC_NAME ${FFMPEG_AVCODEC_CODEC_LIST})
    string(APPEND FFMPEG_AVCODEC_CODEC_LIST_CONTENT "&${AVCODEC_NAME}\,\n")
endforeach()
string(APPEND FFMPEG_AVCODEC_CODEC_LIST_CONTENT "NULL\n")
string(APPEND FFMPEG_AVCODEC_CODEC_LIST_CONTENT "}\;\n")
file(GENERATE OUTPUT "${AVCODEC_GENERATED_INCLUDE_DIRECTORY}/libavcodec/codec_list.c" CONTENT ${FFMPEG_AVCODEC_CODEC_LIST_CONTENT})




target_compile_definitions(ffmpeg.avcodec.config INTERFACE
    HAVE_FAST_64BIT=0
    CONFIG_FRAME_THREAD_ENCODER=0
    CONFIG_MPEGVIDEODEC=0
    CONFIG_GRAY=0
)

target_include_directories(ffmpeg.avcodec.headers INTERFACE
    ${AVCODEC_GENERATED_INCLUDE_DIRECTORY}
)

ff_component_sources(avcodec.core FFMPEG_AVCODEC_CORE_SOURCES)

foreach(FFMPEG_AVCODEC_COMPONENT ${FFMPEG_AVCODEC_COMPONENTS})
    ff_component_link_library(${FFMPEG_AVCODEC_COMPONENT} PRIVATE ffmpeg.avutil.config)
    ff_component_link_library(${FFMPEG_AVCODEC_COMPONENT} PRIVATE ffmpeg.avutil.headers)
    ff_component_link_library(${FFMPEG_AVCODEC_COMPONENT} PRIVATE ffmpeg.avcodec.config)
    ff_component_link_library(${FFMPEG_AVCODEC_COMPONENT} PRIVATE ffmpeg.avcodec.headers)
endforeach()

set(FFMPEG_AVCODEC_COMPONENTS ${FFMPEG_AVCODEC_COMPONENTS} PARENT_SCOPE)


foreach(FFMPEG_AVCODEC_COMPONENT ${FFMPEG_AVCODEC_COMPONENTS})
    ffmpeg_message(STATUS "avcodec" "Built component ${FFMPEG_AVCODEC_COMPONENT}")
endforeach()
