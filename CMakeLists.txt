CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT ( OutOfCoreSparseVoxelOctree )

SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

FIND_PACKAGE(OpenMP REQUIRED )
FIND_PACKAGE(GLM REQUIRED)

SET(Trimesh2_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/trimesh2/include" CACHE PATH "Path to Trimesh2 includes")
FIND_FILE(Trimesh2_TriMesh_h TriMesh.h ${Trimesh2_INCLUDE_DIR})
IF(NOT Trimesh2_TriMesh_h)
	message(SEND_ERROR "Can't find TriMesh.h in ${Trimesh2_INCLUDE_DIR}")
ENDIF()
MARK_AS_ADVANCED(Trimesh2_TriMesh_h)
INCLUDE_DIRECTORIES(${Trimesh2_INCLUDE_DIR})

SET(Trimesh2_LINK_DIR "${CMAKE_SOURCE_DIR}/trimesh2/lib.Linux64" CACHE PATH "Path to Trimesh2 libraries")
FIND_LIBRARY(Trimesh2_LIBRARY trimesh ${Trimesh2_LINK_DIR})
IF(NOT Trimesh2_LIBRARY)
	message(SEND_ERROR "Can't find libtrimesh.a in ${Trimesh2_LINK_DIR}")
ENDIF()
MARK_AS_ADVANCED(Trimesh2_LIBRARY)

IF (MSVC)
  ADD_DEFINITIONS ( -openmp )
ELSE ()
  # Apple and Linux both uses GCC
  ADD_DEFINITIONS ( -fopenmp )
ENDIF (MSVC)

INCLUDE_DIRECTORIES (
  ./src/libs/libtri/include
  ./src/libs/libmorton/include
  )


SET(SVO_BUILDER_SRCS
  ./src/svo_builder/main.cpp
  ./src/svo_builder/OctreeBuilder.cpp
  ./src/svo_builder/partitioner.cpp
  ./src/svo_builder/voxelizer.cpp
)
ADD_EXECUTABLE ( svo_builder ${SVO_BUILDER_SRCS} )
ADD_EXECUTABLE ( svo_builder_binary ${SVO_BUILDER_SRCS} )
SET_TARGET_PROPERTIES(svo_builder_binary PROPERTIES
	COMPILE_FLAGS "-DBINARY_VOXELIZATION ${SHARED_FLAGS}"
)


SET(TRI_CONVERT_SRCS
  ./src/tri_convert/tri_convert.cpp
)
ADD_EXECUTABLE ( tri_convert ${TRI_CONVERT_SRCS} )
ADD_EXECUTABLE ( tri_convert_binary ${TRI_CONVERT_SRCS} )
SET_TARGET_PROPERTIES(tri_convert_binary PROPERTIES
	COMPILE_FLAGS "-DBINARY_VOXELIZATION ${SHARED_FLAGS}"
)

TARGET_LINK_LIBRARIES ( svo_builder
  gomp
)
TARGET_LINK_LIBRARIES ( svo_builder_binary
  gomp
)
TARGET_LINK_LIBRARIES ( tri_convert
  ${Trimesh2_LIBRARY}
  gomp
)
TARGET_LINK_LIBRARIES ( tri_convert_binary
  ${Trimesh2_LIBRARY}
  gomp
)
