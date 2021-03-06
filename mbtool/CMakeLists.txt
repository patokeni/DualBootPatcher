set_source_files_properties(
    daemon_v3.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-missing-declarations"
)

set_source_files_properties(
    packages.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-format-nonliteral"
)

set_source_files_properties(
    sepolpatch.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-keyword-macro"
)

set_source_files_properties(
    auditd.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-gnu-statement-expression -Wno-zero-length-array"
)

set_source_files_properties(
    external/audit/libaudit.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-cast-align -Wno-conversion -Wno-gnu-statement-expression -Wno-old-style-cast -Wno-shorten-64-to-32 -Wno-sign-conversion"
)

set_source_files_properties(
    src/util/legacy_property_service.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-shorten-64-to-32 -Wno-sign-conversion"
)

set_source_files_properties(
    src/util/property_service.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-gnu-statement-expression -Wno-shorten-64-to-32 -Wno-sign-conversion"
)

set_source_files_properties(
    ${CMAKE_SOURCE_DIR}/external/pugixml/src/pugixml.cpp
    PROPERTIES
    COMPILE_FLAGS
    "-Wno-double-promotion -Wno-sign-conversion"
)

if(${MBP_BUILD_TARGET} STREQUAL android-system)
    # Generate validcerts.cpp
    configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/validcerts.cpp.in
        ${CMAKE_CURRENT_BINARY_DIR}/gen/validcerts.cpp
    )

    add_executable(
        mbtool
        appsync.cpp
        appsyncmanager.cpp
        auditd.cpp
        daemon.cpp
        daemon_v3.cpp
        emergency.cpp
        init.cpp
        main.cpp
        mount_fstab.cpp
        multiboot.cpp
        packages.cpp
        properties.cpp
        reboot.cpp
        romconfig.cpp
        roms.cpp
        sepolpatch.cpp
        signature.cpp
        switcher.cpp
        uevent_dump.cpp
        wipe.cpp
        external/audit/libaudit.cpp
        src/initwrapper/cutils/uevent.cpp
        src/initwrapper/devices.cpp
        src/initwrapper/uevent_listener.cpp
        src/util/legacy_property_service.cpp
        src/util/property_service.cpp
        src/util/uevent_thread.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/gen/validcerts.cpp
        ${CMAKE_SOURCE_DIR}/external/pugixml/src/pugixml.cpp
    )
    add_executable(
        mbtool_recovery
        archive_util.cpp
        backup.cpp
        bootimg_util.cpp
        image.cpp
        installer.cpp
        installer_util.cpp
        main.cpp
        multiboot.cpp
        ramdisk_patcher.cpp
        rom_installer.cpp
        romconfig.cpp
        roms.cpp
        sepolpatch.cpp
        signature.cpp
        switcher.cpp
        update_binary.cpp
        update_binary_tool.cpp
        utilities.cpp
        wipe.cpp
        src/util/legacy_property_service.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/gen/validcerts.cpp
    )

    foreach(target mbtool mbtool_recovery)
        # Includes
        target_include_directories(
            ${target}
            PRIVATE
            .
            include
            ${CMAKE_SOURCE_DIR}/external/flatbuffers/include
            ${CMAKE_SOURCE_DIR}/external/minizip
            ${CMAKE_SOURCE_DIR}/external/pugixml/src
        )

        # Disable some pugixml features
        target_compile_definitions(
            ${target}
            PRIVATE
            -DPUGIXML_NO_EXCEPTIONS
            -DPUGIXML_NO_STL
            -DPUGIXML_NO_XPATH
        )
    endforeach()

    target_compile_definitions(
        mbtool_recovery
        PRIVATE
        -DRECOVERY
    )

    set_target_properties(
        mbtool mbtool_recovery
        PROPERTIES
        LINK_FLAGS "-static"
        LINK_SEARCH_START_STATIC ON
    )

    android_link_allow_multiple_definitions(mbtool mbtool_recovery)

    target_link_libraries(
        mbtool
        PRIVATE
        interface.global.CXXVersion
        mbutil-static
        mbsign-static
        mbdevice-static
        mblog-static
        mbbootimg-static
        mbcommon-static
        libminizip
        rapidjson
        Procps::Procps
    )
    target_link_libraries(
        mbtool_recovery
        PRIVATE
        interface.global.CXXVersion
        mbutil-static
        mbsign-static
        mblog-static
        mbdevice-static
        mbbootimg-static
        mbcommon-static
        libminizip
        rapidjson
        LibArchive::LibArchive
        Procps::Procps # TODO
    )

    install(
        TARGETS mbtool mbtool_recovery
        RUNTIME DESTINATION "${BIN_INSTALL_DIR}/"
        COMPONENT Applications
    )
endif()
