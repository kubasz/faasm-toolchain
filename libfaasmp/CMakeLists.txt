cmake_minimum_required(VERSION 3.0)
project(libfaasmp)

# ------ Faasm OpenMP library ------ #

set(CMAKE_CXX_STANDARD 17)

set(PUBLIC_HEADERS
        faasmp/faasmp.h
        faasmp/reduction.h
        )

set(LIB_FILES
        faasmp.cpp
        )

if (CMAKE_SYSTEM_NAME STREQUAL "Wasm")
    message(STATUS "Libfaasmp WebAssembly build")

    set(LIBOMP_ASMFILES "z_Linux_asm.S")
    set_source_files_properties(${LIBOMP_ASMFILES} PROPERTIES LANGUAGE C)
    set(LIB_FILES ${LIBOMP_ASMFILES} ${LIB_FILES})

    add_library(faasmp STATIC ${LIB_FILES})
    set_target_properties(faasmp PROPERTIES PUBLIC_HEADER "${PUBLIC_HEADERS}")

    # Install in WASM sysroot
    install(TARGETS faasmp
            ARCHIVE DESTINATION ${CMAKE_SYSROOT}/lib/wasm32-wasi
            LIBRARY DESTINATION ${CMAKE_SYSROOT}/lib/wasm32-wasi
            PUBLIC_HEADER DESTINATION ${CMAKE_SYSROOT}/include/faasmp
            )

    # Add imports
    install(FILES faasmp.imports
            DESTINATION ${CMAKE_SYSROOT}/lib/wasm32-wasi
            RENAME libfaasmp.imports)

    # Add omp header
    install(FILES omp.h
            DESTINATION ${CMAKE_SYSROOT}/include)

else ()
    message(STATUS "Libfaasmp native build")

    if(BUILD_SHARED_LIBS)
        add_library(faasmp SHARED ${LIB_FILES})
    else()
        add_library(faasmp STATIC ${LIB_FILES})
    endif()

    set_target_properties(faasmp PROPERTIES PUBLIC_HEADER "${PUBLIC_HEADERS}")

    install(TARGETS faasmp
            ARCHIVE DESTINATION lib
            LIBRARY DESTINATION lib
            PUBLIC_HEADER DESTINATION include/faasmp
            )
endif ()
