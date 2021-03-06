set(ZIP FALSE CACHE BOOL "Zip most resources (useful for stand-alone builds, deprecated for linux packages and git builds)")

function(create_zip output_file input_files working_dir all_files)
	if (NOT all_files)
		set(all_files "${input_files}")
	endif()

	add_custom_command(
		COMMAND ${CMAKE_COMMAND} -E tar "cf" "${output_file}" --format=zip -- ${input_files}
		WORKING_DIRECTORY "${working_dir}"
		OUTPUT  "${output_file}"
		DEPENDS ${input_files}
		COMMENT "Zipping to ${output_file}"
	)
endfunction()


file(GLOB RAW_FILES_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/raw/*")
file(GLOB ZIP_FILES "${CMAKE_CURRENT_SOURCE_DIR}/zip/*")

if (NOT ZIP)
	set(RAW_FILES_DIRS ${RAW_FILES_DIRS} ${ZIP_FILES})
endif()

foreach(ITEM ${RAW_FILES_DIRS})
   IF( IS_DIRECTORY "${ITEM}" )
      LIST( APPEND RAW_DIRS "${ITEM}" )
   ELSE()
      LIST( APPEND RAW_FILES "${ITEM}" )
   ENDIF()
endforeach()

install(FILES ${RAW_FILES} DESTINATION ${DATADIR} COMPONENT DATA) # Install unpacked files
install(DIRECTORY ${RAW_DIRS} DESTINATION ${DATADIR} COMPONENT DATA) # Install unpacked dirs

if(ZIP)
	file(GLOB_RECURSE ZIP_FILES_ALL "${CMAKE_CURRENT_SOURCE_DIR}/zip/*")
	create_zip("${CMAKE_CURRENT_BINARY_DIR}/native_data.zip" "${ZIP_FILES}" "${CMAKE_CURRENT_SOURCE_DIR}/zip" "${ZIP_FILES_ALL}")
	install(FILES "${CMAKE_CURRENT_BINARY_DIR}/native_data.zip" DESTINATION ${DATADIR} RENAME "data000.zip" COMPONENT DATA) # Install zip
	add_custom_target("cubosphere-zip" ALL DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/native_data.zip")
endif()

add_subdirectory(desktop)
