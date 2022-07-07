add_custom_target(plugin)
set(TARGET_NAME amirstan_plugin)
set(SHARED_TARGET ${TARGET_NAME}_shared)
set(STATIC_TARGET ${TARGET_NAME})

enable_language(CUDA)
set(TARGET_DIR ${CMAKE_CURRENT_SOURCE_DIR})

include_directories(common)

set(PLUGIN_LIST adaptivePoolPlugin
                batchedNMSPlugin
                carafeFeatureReassemblePlugin
                deformableConvPlugin
                deformablePoolPlugin
                delta2bboxPlugin
                gridAnchorDynamicPlugin
                gridSamplePlugin
                groupNormPlugin
                meshGridPlugin
                roiExtractorPlugin
                roiPoolPlugin
                torchBmmPlugin
                torchCumMaxMinPlugin
                torchCumPlugin
                torchEmbeddingPlugin
                torchGatherPlugin
                torchNMSPlugin
                torchUnfoldPlugin
                )

set(INFER_PLUGIN_SRC InferPlugin.cpp)

# set(INFER_PLUGIN_LIB ${TENSORRT_LIBRARY})

foreach(PLUGIN_NAME ${PLUGIN_LIST})
add_subdirectory(${PLUGIN_NAME})
set(INFER_PLUGIN_LIB ${INFER_PLUGIN_LIB} ${PLUGIN_NAME}_static)
endforeach()

if (WITH_DEEPSTREAM)                                                                                                        
    MESSAGE(STATUS "Adding NvDsInferParseMmdet")
    set(INFER_PLUGIN_LIB ${INFER_PLUGIN_LIB} ${TENSORRT_LIBRARY_INFER_PLUGIN} )
    set(INFER_PLUGIN_SRC ${INFER_PLUGIN_SRC} NvDsInferParseMmdet.cpp)
endif ()

cuda_add_library(${STATIC_TARGET} STATIC ${INFER_PLUGIN_SRC})
target_link_libraries(${STATIC_TARGET} ${INFER_PLUGIN_LIB})
target_link_libraries(${STATIC_TARGET} CUDA::cublas)
target_include_directories(${STATIC_TARGET} PRIVATE ${PROJECT_SOURCE_DIR}/common)

include(GNUInstallDirs)

install(TARGETS ${TARGET_NAME} ${INFER_PLUGIN_LIB}
    EXPORT ${TARGET_NAME}Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT ${TARGET_NAME}Targets
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${TARGET_NAME}
)