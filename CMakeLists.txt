cmake_minimum_required(VERSION 2.8)
project(Tundra)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

include_directories(lua/src src)

if(WIN32)
	add_definitions(-DWINVER=0x0600 -D_WIN32_WINNT=0x0600 -DWIN32_LEAN_AND_MEAN -DNOMINMAX -D_CRT_SECURE_NO_WARNINGS -D_MBCS)
endif()

if(MSVC)
	set(COMPILER_WARNING_FLAGS "/wd4624 /wd4127 /wd4100 /wd4324")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMPILER_WARNING_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMPILER_WARNING_FLAGS}")
endif()

if(MSVC_IDE)
	set(CMAKE_SUPPRESS_REGENERATION TRUE)
endif()

macro(prepend_path SOURCES PATH)
    unset(TMP)
    foreach(SOURCE ${${SOURCES}})
        list(APPEND TMP ${PATH}/${SOURCE})
    endforeach()
    set(${SOURCES} ${TMP})
endmacro()

set(LUA_SOURCES
	lapi.c lauxlib.c lbaselib.c lcode.c
	ldblib.c ldebug.c ldo.c ldump.c
	lfunc.c lgc.c linit.c liolib.c
	llex.c lmathlib.c lmem.c loadlib.c
	lobject.c lopcodes.c loslib.c lparser.c
	lstate.c lstring.c lstrlib.c ltable.c
	ltablib.c ltm.c lundump.c lvm.c
	lzio.c
)
prepend_path(LUA_SOURCES lua/src)

set(LIBTUNDRA_SOURCES
	BinaryWriter.cpp BuildQueue.cpp Common.cpp DagGenerator.cpp
	Driver.cpp FileInfo.cpp Hash.cpp HashTable.cpp
	IncludeScanner.cpp JsonParse.cpp MemAllocHeap.cpp
	MemAllocLinear.cpp MemoryMappedFile.cpp PathUtil.cpp
	ScanCache.cpp Scanner.cpp SignalHandler.cpp StatCache.cpp
	TargetSelect.cpp Thread.cpp dlmalloc.c
	ExecUnix.cpp ExecWin32.cpp DigestCache.cpp FileSign.cpp
	HashSha1.cpp HashFast.cpp ConditionVar.cpp ReadWriteLock.cpp
)
if(NOT WIN32)
	set(LIBTUNDRA_SOURCES ${LIBTUNDRA_SOURCES} TerminalIo.cpp)
endif()
prepend_path(LIBTUNDRA_SOURCES src)

set(T2LUA_SOURCES
	LuaMain.cpp LuaInterface.cpp LuaInterpolate.cpp LuaJsonWriter.cpp
	LuaPath.cpp LuaProfiler.cpp
)
prepend_path(T2LUA_SOURCES src)

set(T2INSPECT_SOURCES
	InspectMain.cpp
)
prepend_path(T2INSPECT_SOURCES src)

set(UNITTEST_SOURCES
	TestHarness.cpp Test_BitFuncs.cpp Test_Buffer.cpp Test_Djb2.cpp Test_Hash.cpp
	Test_IncludeScanner.cpp Test_Json.cpp Test_MemAllocLinear.cpp Test_Pow2.cpp
	Test_TargetSelect.cpp test_PathUtil.cpp
)
prepend_path(UNITTEST_SOURCES src)

set(TUNDRA_SOURCES
	Main.cpp
)
prepend_path(TUNDRA_SOURCES src)

add_library(tundralua ${LUA_SOURCES})
add_library(tundra ${LIBTUNDRA_SOURCES})
add_executable(tundra2 ${TUNDRA_SOURCES})
target_link_libraries(tundra2 tundra)
add_executable(t2-lua ${T2LUA_SOURCES})
target_link_libraries(t2-lua tundra tundralua)
add_executable(t2-inspect ${T2INSPECT_SOURCES})
target_link_libraries(t2-inspect tundra)
add_executable(t2-unittest ${T2INSPECT_SOURCES})
target_link_libraries(t2-unittest tundra)
