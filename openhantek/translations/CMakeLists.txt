# See this question on stackoverflow on why the following line is relevant
# https://stackoverflow.com/q/37960897
set_directory_properties(PROPERTIES CLEAN_NO_CUSTOM 1)

find_package(Qt5LinguistTools QUIET)

if (NOT Qt5LinguistTools_FOUND)
    message(STATUS "Will not build translations! Please install Qt5LinguistTools")
	# Copy empty translations.qrc file to not make the build fail because of this.
	configure_file("${CMAKE_CURRENT_LIST_DIR}/translations.qrc" "${CMAKE_BINARY_DIR}/translations.qrc" @ONLY)
    return()
endif()

file(GLOB_RECURSE SRCS "${CMAKE_SOURCE_DIR}/*.cpp")
file(GLOB_RECURSE UIS "${CMAKE_SOURCE_DIR}/*.ui")
file(GLOB TS_FILES "*.ts")
qt5_create_translation(TRANSLATION_BIN_FILES ${SRC} ${UIS} ${TS_FILES})
#qt5_create_translation(TRANSLATION_BIN_FILES ${CMAKE_SOURCE_DIR} ${TS_FILES})
# Even though this make look cleaner, it does not update the translation file
# because is not dependent on them. Hence it updates only when the directory
# ${CMAKE_SOURCE_DIR} is udpated or any of its files.
# This would be a strange behavior!

# We prepare the translations.qrc file and insert all available compiled translation files now.
set(QRC_ITEMS "")
foreach(TRANSLATION_FILE ${TRANSLATION_BIN_FILES})
	get_filename_component(FILENAME "${TRANSLATION_FILE}" NAME)
	set(QRC_ITEMS "${QRC_ITEMS}\n<file alias=\"${FILENAME}\">${TRANSLATION_FILE}</file>")
endforeach()

set(TRANSLATION_QRC "${CMAKE_BINARY_DIR}/translations.qrc")
set(TRANSLATION_QRC "${TRANSLATION_QRC}" PARENT_SCOPE)
configure_file("${CMAKE_CURRENT_LIST_DIR}/translations.qrc" ${TRANSLATION_QRC} @ONLY)
QT5_ADD_RESOURCES(TRANSLATION_CPP "${TRANSLATION_QRC}")
add_custom_target(translations DEPENDS ${TRANSLATION_BIN_FILES})
