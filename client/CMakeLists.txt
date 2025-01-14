cmake_minimum_required(VERSION 2.8.12)

project(client)
include(CheckIncludeFile)
include(CompilerRuntime)
check_include_file("tgmath.h" HAVE_TGMATH_H)

# find and add source files
file(GLOB CLDLL_SOURCES "render/*.cpp")
list(APPEND CLDLL_SOURCES
	"../game_shared/bone_setup.cpp"
	"../game_shared/common.cpp"
	"../game_shared/ikcontext.cpp"
	"../game_shared/jigglebones.cpp"
	"../game_shared/material.cpp"
	"../game_shared/mathlib.cpp"
	"../game_shared/matrix.cpp"
	"../game_shared/meshdesc.cpp"
	"../game_shared/playermove.cpp"
	"../game_shared/procbones.cpp"
	"../game_shared/stringlib.cpp"
	"../game_shared/virtualfs.cpp"
	"../game_shared/trace.cpp"
)

list(APPEND CLDLL_SOURCES
	"ammo.cpp"
	"ammo_secondary.cpp"
	"ammohistory.cpp"
	"battery.cpp"
	"cdll_int.cpp"
	"death.cpp"
	"flashlight.cpp"
	"geiger.cpp"
	"health.cpp"
	"entity.cpp"
	"hud.cpp"
	"hud_msg.cpp"
	"hud_redraw.cpp"
	"hud_update.cpp"
	"hud_flashlight.cpp"
	"input.cpp"
	"input_xash3d.cpp"
	"input_mouse.cpp"
	"input_goldsource.cpp"
	"menu.cpp"
	"message.cpp"
	"motd.cpp"
	"parsemsg.cpp"
	"saytext.cpp"
	"scoreboard.cpp"
	"status_icons.cpp"
	"statusbar.cpp"
	"text_message.cpp"
	"train.cpp"
	"utils.cpp"
	"r_view.cpp"
)

add_library(${PROJECT_NAME} SHARED ${CLDLL_SOURCES})
target_include_directories(${PROJECT_NAME} PRIVATE
	"."
	"render/"
	"../dlls"
	"../dlls/wpn_shared"
	"../common"
	"../engine"
	"../pm_shared"
	"../game_shared"
	"../public"
)

if(HAVE_TGMATH_H)
	target_compile_definitions(${PROJECT_NAME} PRIVATE HAVE_TGMATH_H=1)
endif()

target_compile_definitions(${PROJECT_NAME} PRIVATE 
	CLIENT_WEAPONS=1 
	CLIENT_DLL=1
)

if(NOT MSVC)
	target_compile_options(${PROJECT_NAME} PRIVATE -fno-exceptions) # GCC/Clang flag
	target_compile_options(${PROJECT_NAME} PRIVATE -Wno-write-strings) # GCC/Clang flag
	target_compile_options(${PROJECT_NAME} PRIVATE -fvisibility=hidden) # GCC/Clang flag
	target_compile_definitions(${PROJECT_NAME} PRIVATE _LINUX=1 LINUX=1) # It seems enough for all non-Win32 systems
	target_compile_definitions(${PROJECT_NAME} PRIVATE stricmp=strcasecmp strnicmp=strncasecmp)
	if(NOT MINGW)
		target_compile_definitions(${PROJECT_NAME} PRIVATE _snprintf=snprintf _vsnprintf=vsnprintf)
	endif()
else()
	# disable annoying CRT warnings
	target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS=1 _CRT_NONSTDC_NO_DEPRECATE=1)
endif()

if(MSVC)
	# debug mode compiler flags
	target_compile_options(${PROJECT_NAME} PRIVATE $<$<CONFIG:Debug>:/Od>) # disable optimizing at all
	target_compile_options(${PROJECT_NAME} PRIVATE $<$<CONFIG:Debug>:/ZI>) # enable "Edit and Continue" MSVC feature
	target_link_options(${PROJECT_NAME} PRIVATE $<$<CONFIG:Debug>:
		/INCREMENTAL 
		/SAFESEH:NO
	>)

	# release mode compiler flags
	target_compile_options(${PROJECT_NAME} PRIVATE $<$<CONFIG:Release>:/Ob1>) # disable force function inlining, to prevent segfault in release mode
	target_compile_options(${PROJECT_NAME} PRIVATE $<$<CONFIG:Release>:/O1>) # enable optimizing to minimize size
endif()

# set static compiler runtime
set_compiler_runtime(${PROJECT_NAME} STATIC)

if(WIN32)
	target_link_libraries(${PROJECT_NAME} user32.lib)
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
	POSITION_INDEPENDENT_CODE 1)

# disable library prefix for Windows/iOS/Linux
if(APPLE OR WIN32 OR ${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
	set_target_properties(${PROJECT_NAME} PROPERTIES
		OUTPUT_NAME "client"
		PREFIX "")
endif()

install( TARGETS ${PROJECT_NAME}
	DESTINATION "${GAMEDIR}/${CLIENT_INSTALL_DIR}/"
	PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
	    GROUP_READ GROUP_EXECUTE
		WORLD_READ WORLD_EXECUTE 
)
