# ProtoBuf

## 编译 user.go 文件

```bash
# protoc 编译器的 grpc 插件会处理 service 字段定义的 UserInfoService
# 使 service 能编码、解码 message
protoc -I . --go_out=plugins=grpc:. ./activity.proto
```