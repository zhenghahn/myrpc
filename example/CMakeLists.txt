#add_subdirectory(callee)
#add_subdirectory(caller)


# 编译服务端（callee）
add_executable(server callee/Kserver.cc user.pb.cc)
# 链接核心库+全局依赖+protobuf生成的代码
target_link_libraries(server
    krpc_core  # 链接src目录编译的krpc_core核心库
    ${LIBS}    # 全局依赖库
)

# 编译客户端（caller）
add_executable(client caller/Kclient.cc user.pb.cc)
# 链接核心库+全局依赖+protobuf生成的代码
target_link_libraries(client
    krpc_core
    ${LIBS}
)

# 重命名可执行文件（避免和其他target重名，可选）
set_target_properties(server PROPERTIES OUTPUT_NAME "rpc_server")
set_target_properties(client PROPERTIES OUTPUT_NAME "rpc_client")
# 最终输出到bin目录时，改回server/client（可选）
# set_target_properties(rpc_server PROPERTIES OUTPUT_NAME "server")
# set_target_properties(rpc_client PROPERTIES OUTPUT_NAME "client")
