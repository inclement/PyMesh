# General lib locations
LINK_DIRECTORIES(/opt/local/lib)

# Inlclude CGAL
IF (CGAL_FOUND)
    INCLUDE(${CGAL_USE_FILE})
ELSE (CGAL_FOUND)
    MESSAGE(FATAL_ERROR "CGAL is required for Wires module")
ENDIF (CGAL_FOUND)

# tetgen
IF (TETGEN_FOUND)
    INCLUDE_DIRECTORIES(${TETGEN_INCLUDE_DIRS})
    ADD_DEFINITIONS(-DTETLIBRARY)
ENDIF (TETGEN_FOUND)

INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/tools)

FILE(GLOB SRC_FILES *.cpp)
FILE(GLOB INC_FILES *.h)
FILE(GLOB DEP_FILES *.i)

ADD_SUBDIRECTORY(Attributes)
ADD_SUBDIRECTORY(WireNetwork)
ADD_SUBDIRECTORY(Tiler)
ADD_SUBDIRECTORY(Misc)
ADD_SUBDIRECTORY(Inflator)
ADD_SUBDIRECTORY(Interfaces)
ADD_SUBDIRECTORY(Parameters)

ADD_LIBRARY(lib_wires SHARED ${SRC_FILES} ${INC_FILES})
TARGET_LINK_LIBRARIES(lib_wires Mesh lib_MeshUtils lib_convex_hull
    lib_triangle_wrapper lib_boolean lib_tetgen_wrapper)
SET_TARGET_PROPERTIES(lib_wires PROPERTIES OUTPUT_NAME "wires")

IF(NOT WITHOUT_SWIG)
    SET(SWG_FILES PyWires.i)
    SET(SWIG_MODULE_PyWires_EXTRA_DEPS ${DEP_FILES} ${INC_FILES})

    # Generate wrapper for python
    SET_SOURCE_FILES_PROPERTIES(${SWG_FILES} PROPERTIES CPLUSPLUS ON)
    SET_SOURCE_FILES_PROPERTIES(${SWG_FILES} PROPERTIES CMAKE_SWIG_FLAGS "-includeall")
    SWIG_ADD_MODULE(PyWires python ${SWG_FILES} ${INC_FILES})
    SWIG_LINK_LIBRARIES(PyWires ${PYTHON_LIBRARIES} Mesh lib_wires)

    # Usage: make wires
    ADD_CUSTOM_TARGET(wires DEPENDS lib_wires _PyWires)
ELSE(NOT WITHOUT_SWIG)
    ADD_CUSTOM_TARGET(wires DEPENDS lib_wires)
ENDIF(NOT WITHOUT_SWIG)
