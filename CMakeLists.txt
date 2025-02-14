cmake_minimum_required(VERSION 3.5)

project(kdiskmark VERSION 2.2.0 LANGUAGES CXX)

set(CMAKE_PROJECT_HOMEPAGE_URL "https://github.com/JonMagon/KDiskMark")
set(CMAKE_PROJECT_DESCRIPTION "A simple open source disk benchmark tool for Linux distros")

set(APPLICATION_NAME "KDiskMark")
set(DESKTOP_FILE kdiskmark.desktop)

configure_file(src/cmake.h.in cmake.h)
include_directories(${PROJECT_BINARY_BIN})

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CPACK_PACKAGE_VENDOR ${APPLICATION_NAME})
set(CPACK_PACKAGE_INSTALL_DIRECTORY ${APPLICATION_NAME})
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_CONTACT jonmagon@gmail.com)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)

set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)
set(CPACK_DEBIAN_PACKAGE_GENERATE_SHLIBS_POLICY >=)
set(CPACK_RPM_FILE_NAME RPM-DEFAULT)
set(CPACK_RPM_PACKAGE_LICENSE GPLv3)
set(CPACK_RPM_PACKAGE_GROUP Applications/System)
set(CPACK_DEBIAN_PACKAGE_DEPENDS "fio (>= 3.1)")
set(CPACK_RPM_COMPRESSION_TYPE lzma)
set(CPACK_NSIS_DISPLAY_NAME ${APPLICATION_NAME})

option(BUILD_WITH_PAGECACHE_CLEARING_SUPPORT "Cache clearing functionality" ON)
option(PERFORM_PAGECACHE_CLEARING_USING_KF5AUTH "Determines whether a helper will be built" ON)

find_package(Qt5 COMPONENTS Widgets LinguistTools REQUIRED)

find_package(ECM REQUIRED NO_MODULE)
list(APPEND CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(CPack)
include(GNUInstallDirs)
include(ECMInstallIcons)
include(KDEInstallDirs)

set(TS_FILES
    data/translations/${PROJECT_NAME}_cs_CZ.ts
    data/translations/${PROJECT_NAME}_de_DE.ts
    data/translations/${PROJECT_NAME}_es_MX.ts
    data/translations/${PROJECT_NAME}_fr_FR.ts
    data/translations/${PROJECT_NAME}_hi_IN.ts
    data/translations/${PROJECT_NAME}_it_IT.ts
    data/translations/${PROJECT_NAME}_pl_PL.ts
    data/translations/${PROJECT_NAME}_pt_BR.ts
    data/translations/${PROJECT_NAME}_ru_RU.ts
    data/translations/${PROJECT_NAME}_sk_SK.ts
    data/translations/${PROJECT_NAME}_uk_UA.ts
    data/translations/${PROJECT_NAME}_zh_CN.ts
)

qt5_create_translation(QM_FILES ${CMAKE_SOURCE_DIR} ${TS_FILES})

add_executable(${PROJECT_NAME}
    ${QM_FILES}
    data/icons/flags/flags.qrc
    src/main.cpp
    src/mainwindow.cpp
    src/mainwindow.h
    src/mainwindow.ui
    src/benchmark.cpp
    src/benchmark.h
    src/about.cpp
    src/about.h
    src/about.ui
    src/appsettings.h
    src/appsettings.cpp
    src/settings.cpp
    src/settings.h
    src/settings.ui
    src/global.cpp
    src/global.h
    src/diskdriveinfo.cpp
    src/diskdriveinfo.h
)

target_link_libraries(${PROJECT_NAME} PRIVATE Qt5::Widgets)

if(BUILD_WITH_PAGECACHE_CLEARING_SUPPORT)
    target_compile_definitions(${PROJECT_NAME} PRIVATE BUILD_WITH_PAGECACHE_CLEARING_SUPPORT)
    if(PERFORM_PAGECACHE_CLEARING_USING_KF5AUTH)
        target_compile_definitions(${PROJECT_NAME} PRIVATE PERFORM_PAGECACHE_CLEARING_USING_KF5AUTH)
        find_package(KF5Auth REQUIRED NO_MODULE)
        target_link_libraries(${PROJECT_NAME} PRIVATE KF5::Auth)
    endif()
endif()

set(APP_ICONS
    data/icons/16-apps-kdiskmark.png
    data/icons/24-apps-kdiskmark.png
    data/icons/32-apps-kdiskmark.png
    data/icons/48-apps-kdiskmark.png
    data/icons/64-apps-kdiskmark.png
    data/icons/128-apps-kdiskmark.png
    data/icons/256-apps-kdiskmark.png
    data/icons/512-apps-kdiskmark.png
)

install(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES ${QM_FILES} DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/translations)
install(FILES data/${DESKTOP_FILE} DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/applications)

ecm_install_icons(ICONS ${APP_ICONS} DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/icons)

# Helper

if(BUILD_WITH_PAGECACHE_CLEARING_SUPPORT AND PERFORM_PAGECACHE_CLEARING_USING_KF5AUTH)
    add_executable(${PROJECT_NAME}_helper
        src/helper.cpp
        src/helper.h
    )

    target_link_libraries(${PROJECT_NAME}_helper KF5::Auth)

    install(TARGETS ${PROJECT_NAME}_helper DESTINATION ${KAUTH_HELPER_INSTALL_DIR})
    kauth_install_helper_files(${PROJECT_NAME}_helper org.jonmagon.kdiskmark root)
    kauth_install_actions(org.jonmagon.kdiskmark data/kdiskmark.actions)
endif()
