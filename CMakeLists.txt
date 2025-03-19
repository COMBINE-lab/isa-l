## Adapted from https://github.com/mxmlnkn/rapidgzip/blob/master/src/CMakeLists.txt
project(isal_inflate
    LANGUAGES C CXX 
)

set(REQUIRED_ASSEMBLER "" CACHE STRING "Represents the assembler required to build isa-l")

if(CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "x86_64")
    message(STATUS "Host detected as x86_64 --- checking availability of nasm compiler")
    enable_language(ASM_NASM)
    if(CMAKE_ASM_NASM_COMPILER_LOADED)
        set(ASSEMBLER "NASM")
    else()
        message(FATAL_ERROR "could not find assembler to build isal-l")
    endif()
elseif((CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "arm64") OR (CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "aarch64"))
    message(STATUS "Host detected as arm64 or aarch64 --- checking availability of assembler")
    enable_language(ASM)
    if(CMAKE_ASM_COMPILER_LOADED)
        set(ASSEMBLER "ASM")
    else()
        message(FATAL_ERROR "could not find assembler to build isal-l")
    endif()
endif()

set(ISAL_HOME "${CMAKE_CURRENT_SOURCE_DIR}")
add_library(isal_inflate STATIC
    # Files suffixed with _test, _perf, _example can be ignored!
    ${ISAL_HOME}/include/igzip_lib.h
    ${ISAL_HOME}/include/unaligned.h
    ${ISAL_HOME}/include/reg_sizes.asm
    ${ISAL_HOME}/include/multibinary.asm
    ${ISAL_HOME}/igzip/igzip_inflate.c
    ${ISAL_HOME}/igzip/igzip.c
    ${ISAL_HOME}/igzip/hufftables_c.c
    ${ISAL_HOME}/igzip/igzip_checksums.h
    ${ISAL_HOME}/igzip/igzip_inflate_multibinary.asm
    ${ISAL_HOME}/igzip/igzip_decode_block_stateless_01.asm
    ${ISAL_HOME}/igzip/igzip_decode_block_stateless_04.asm
    #${ISAL_HOME}/igzip/igzip_decode_block_stateless.asm
    ${ISAL_HOME}/igzip/rfc1951_lookup.asm
    ${ISAL_HOME}/igzip/igzip_wrapper.h
    ${ISAL_HOME}/igzip/static_inflate.h
    ${ISAL_HOME}/igzip/stdmac.asm

    # Compression
    #${ISAL_HOME}/igzip/igzip_base_aliases.c
    ${ISAL_HOME}/igzip/encode_df.c
    ${ISAL_HOME}/igzip/igzip_deflate_hash.asm
    ${ISAL_HOME}/igzip/igzip_icf_base.c
    ${ISAL_HOME}/igzip/igzip_icf_body.c
    ${ISAL_HOME}/igzip/igzip_base.c
    ${ISAL_HOME}/igzip/igzip_body.asm
    ${ISAL_HOME}/igzip/igzip_multibinary.asm
    ${ISAL_HOME}/igzip/igzip_update_histogram_01.asm
    ${ISAL_HOME}/igzip/igzip_update_histogram_04.asm
    #${ISAL_HOME}/igzip/igzip_update_histogram.asm

    #${ISAL_HOME}/igzip/bitbuf2.asm
    #${ISAL_HOME}/igzip/data_struct2.asm
    ${ISAL_HOME}/igzip/encode_df_04.asm
    ${ISAL_HOME}/igzip/encode_df_06.asm
    #${ISAL_HOME}/igzip/heap_macros.asm
    #${ISAL_HOME}/igzip/huffman.asm
    #${ISAL_HOME}/igzip/igzip_compare_types.asm
    ${ISAL_HOME}/igzip/igzip_finish.asm
    ${ISAL_HOME}/igzip/igzip_gen_icf_map_lh1_04.asm
    ${ISAL_HOME}/igzip/igzip_gen_icf_map_lh1_06.asm
    ${ISAL_HOME}/igzip/igzip_icf_body_h1_gr_bt.asm
    ${ISAL_HOME}/igzip/igzip_icf_finish.asm
    ${ISAL_HOME}/igzip/igzip_set_long_icf_fg_04.asm
    ${ISAL_HOME}/igzip/igzip_set_long_icf_fg_06.asm
    #${ISAL_HOME}/igzip/inflate_data_structs.asm
    #${ISAL_HOME}/igzip/lz0a_const.asm
    #${ISAL_HOME}/igzip/options.asm
    ${ISAL_HOME}/igzip/proc_heap.asm
    #${ISAL_HOME}/igzip/rfc1951_lookup.asm

    ${ISAL_HOME}/igzip/adler32_avx2_4.asm
    ${ISAL_HOME}/igzip/adler32_sse.asm
    ${ISAL_HOME}/igzip/adler32_base.c
    #${ISAL_HOME}/igzip/encode_df.c
    ${ISAL_HOME}/igzip/flatten_ll.c
    #${ISAL_HOME}/igzip/generate_custom_hufftables.c
    #${ISAL_HOME}/igzip/generate_static_inflate.c
    ${ISAL_HOME}/igzip/huff_codes.c
    #${ISAL_HOME}/igzip/hufftables_c.c
    #${ISAL_HOME}/igzip/igzip_base_aliases.c
    #${ISAL_HOME}/igzip/igzip_base.c
    #${ISAL_HOME}/igzip/igzip_icf_base.c
    #${ISAL_HOME}/igzip/igzip_icf_body.c
    #${ISAL_HOME}/igzip/proc_heap_base.c

    ${ISAL_HOME}/crc/crc_multibinary.asm
    ${ISAL_HOME}/crc/crc32_gzip_refl_by16_10.asm
    ${ISAL_HOME}/crc/crc32_gzip_refl_by8_02.asm
    ${ISAL_HOME}/crc/crc32_gzip_refl_by8.asm
    ${ISAL_HOME}/crc/crc_base.c
    #${ISAL_HOME}/crc/crc_base_aliases.c
)
set_target_properties(isal_inflate PROPERTIES LANGUAGE "C ${ASSEMBLER}")
target_compile_options(isal_inflate PRIVATE
    "$<$<OR:$<COMPILE_LANG_AND_ID:CXX,Clang>,$<COMPILE_LANG_AND_ID:C,Clang>>:-Wno-sign-compare;-Wno-unused-parameter;-Wno-shadow;-Wno-incompatible-pointer-types-discards-qualifiers;-Wno-implicit-fallthrough>"
    "$<$<OR:$<COMPILE_LANG_AND_ID:CXX,GNU>,$<COMPILE_LANG_AND_ID:C,GNU>>:-Wno-sign-compare;-Wno-unused-parameter;-Wno-shadow;-Wno-incompatible-pointer-types-discards-qualifiers;-Wno-implicit-fallthrough>"
)
target_include_directories(isal_inflate
    PUBLIC
        ${ISAL_HOME}/include
    PRIVATE
        ${ISAL_HOME}/igzip
)
endif()
