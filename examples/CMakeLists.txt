# Needed to find libheif/heif_version.h while compiling the library
include_directories(${libheif_BINARY_DIR} ${libheif_SOURCE_DIR}/libheif/api ${libheif_SOURCE_DIR}/libheif)

# Centraliza el directorio de salida de los ejecutables
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

if (MSVC)
    set(getopt_sources
            ../extra/getopt.c
            ../extra/getopt.h
            ../extra/getopt_long.c
            )
    include_directories("../extra")
endif ()

if (WITH_HEADER_COMPRESSION)
    add_compile_definitions(WITH_HEADER_COMPRESSION=1)
endif ()

add_executable(heif-info ${getopt_sources}
        heif_info.cc
        common.cc
        common.h)
target_link_libraries(heif-info PRIVATE heif heifio)
install(TARGETS heif-info RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES heif-info.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)


add_executable(heif-dec ${getopt_sources}
        heif_dec.cc
        common.cc
        common.h)
target_link_libraries(heif-dec PRIVATE heif heifio)
target_include_directories(heif-dec PRIVATE ${libheif_SOURCE_DIR})
install(TARGETS heif-dec RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES heif-dec.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)

# create symbolic link from the old name `heif-convert` to `heif-dec`
if(NOT WIN32)
  install(CODE "execute_process(COMMAND ${CMAKE_COMMAND} -E create_symlink heif-dec${CMAKE_EXECUTABLE_SUFFIX} \$ENV{DESTDIR}${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}/heif-convert${CMAKE_EXECUTABLE_SUFFIX})")
else()
  install(CODE "
    file(REMOVE_RECURSE ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}/heif-convert${CMAKE_EXECUTABLE_SUFFIX})
    file(COPY ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}/heif-dec${CMAKE_EXECUTABLE_SUFFIX}
         DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}
         FILE_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)")
endif()


add_executable(heif-enc ${getopt_sources}
        heif_enc.cc
        benchmark.h
        benchmark.cc
        common.cc
        common.h)
target_link_libraries(heif-enc PRIVATE heif heifio)
target_include_directories(heif-enc PRIVATE ${libheif_SOURCE_DIR})
install(TARGETS heif-enc RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES heif-enc.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)


if (PNG_FOUND)
    add_executable(heif-thumbnailer ${getopt_sources}
            heif_thumbnailer.cc
            common.cc
            common.h)
    target_link_libraries(heif-thumbnailer PRIVATE heif heifio)
    target_include_directories(heif-thumbnailer PRIVATE ${libheif_SOURCE_DIR})

    install(TARGETS heif-thumbnailer RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
    install(FILES heif-thumbnailer.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)
endif ()


find_package(SDL2 NO_MODULE)

if (SDL2_FOUND)
    add_executable(heif-view ${getopt_sources}
            heif_view.cc
            sdl.cc
            sdl.hh
            common.cc
            common.h)
    target_link_libraries(heif-view PRIVATE heif SDL2 winmm)
    target_include_directories(heif-view PRIVATE ${libheif_SOURCE_DIR} ${SDL2_INCLUDE_DIRS})
    install(TARGETS heif-view RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endif ()


add_executable(heif-test ${getopt_sources}
        heif_test.cc
        common.cc
        common.h)
target_link_libraries(heif-test PRIVATE heif heifio)
