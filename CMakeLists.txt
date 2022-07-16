find_package(Protobuf REQUIRED)
if(Protobuf_FOUND)
    include_directories(${Protobuf_INCLUDE_DIRS})
    message("-- Found Protobuf lib: ${Protobuf_LIBRARIES}")
else()
    message(FATAL_ERROR "Could not find the Protobuf library")
endif()

set(PROTO_SRC ${CMAKE_CURRENT_SOURCE_DIR}/invest-api/src/docs/contracts)

file(GLOB PROTO_INPUT  ${PROTO_SRC}/*.proto)
FOREACH(INPUT_FILE ${PROTO_INPUT})
    protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${INPUT_FILE})
    set(PROTO_GEN ${PROTO_GEN} ${PROTO_SRCS} ${PROTO_HDRS})
ENDFOREACH()

SET_SOURCE_FILES_PROPERTIES(${PROTO_GEN} PROPERTIES GENERATED TRUE)

file(GLOB TINKOFF_SRC src/*)
add_library(tinkoff ${TINKOFF_SRC} ${PROTO_GEN})
target_include_directories(tinkoff PRIVATE include)
target_include_directories(tinkoff PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(tinkoff ${Protobuf_LIBRARIES})

# ad-hoc (issue: https://github.com/Tinkoff/investAPI/issues/84)
add_custom_target(rename_namespace
    VERBATIM COMMAND bash -c "sed -i '' 's/namespace public /namespace public_ /g; s/::public::/::public_::/g' ${CMAKE_CURRENT_BINARY_DIR}/*.pb.*"
)
add_dependencies(tinkoff rename_namespace)