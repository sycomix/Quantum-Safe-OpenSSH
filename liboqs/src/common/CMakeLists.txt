# SPDX-License-Identifier: MIT

if(CMAKE_C_COMPILER_ID STREQUAL "GNU" OR
   CMAKE_C_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wbad-function-cast)
endif()
if(CMAKE_C_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wcast-qual)
    add_compile_options(-Wnarrowing)
    add_compile_options(-Wconversion)
endif()

if(OQS_USE_AES_OPENSSL)
    set(AES_IMPL aes/aes_ossl.c)
else()
   set(AES_IMPL aes/aes.c aes/aes_c.c)
    if (OQS_USE_AES_INSTRUCTIONS)
      set(AES_IMPL ${AES_IMPL} aes/aes128_ni.c)
      set(AES_IMPL ${AES_IMPL} aes/aes256_ni.c)
      set_source_files_properties(aes/aes128_ni.c PROPERTIES COMPILE_FLAGS -maes)
      set_source_files_properties(aes/aes256_ni.c PROPERTIES COMPILE_FLAGS -maes)
   endif()
endif()

if(OQS_USE_SHA2_OPENSSL)
    set(SHA2_IMPL sha2/sha2_ossl.c)
else()
    set(SHA2_IMPL sha2/sha2_c.c)
endif()

set(SHA3_IMPL sha3/sha3_c.c)
if(OQS_USE_SHA3_OPENSSL)
    set(SHA3_IMPL ${SHA3_IMPL} sha3/sha3_ossl.c)
endif()
if(OQS_USE_AVX2_INSTRUCTIONS AND
   NOT OQS_PORTABLE_BUILD_FIXME)
    set(SHA3_IMPL ${SHA3_IMPL} sha3/sha3x4.c)
    add_compile_options(-mavx2)
endif()

add_library(common OBJECT ${AES_IMPL}
                          ${SHA2_IMPL}
                          ${SHA3_IMPL}
                          common.c
                          pqclean_shims/nistseedexpander.c
                          rand/rand.c
                          rand/rand_nist.c)
if(OQS_USE_OPENSSL)
    target_include_directories(common PRIVATE ${OPENSSL_INCLUDE_DIR})
endif()
