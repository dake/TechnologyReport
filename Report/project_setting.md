## Xcode 项目配置规范

### 1.基于 TCKit 基础库，Configurations 的作用与配置
 1.1 默认配置包含 Debug 跟 Release ，如果使用了 jenkins，则对应添加 DebugCI 跟 ReleaseCI
 
 1.2 基于 TCKit 库依赖 Publish 进行打包，Configuration 中需要添加 Publish，才能保证 TCKit 库的环境正确

 1.3 与第三方库协作的配置原则，第三方库 Configurations 中一般默认只有 Debug 跟 Release，原则就是当主 Project 打包配置选用 Publish而第三方库不包含 Publish，默认第三方库使用 release

1.4 对于 Preprocessor Macros中 Publish、Release、ReleaseCI 需要主动添加 NDEBUG 标志，因为许多三方的 C 库对该字段依赖，以防止系统断言在生产环境生效

### 2. 静态库 .a 与 framework 的选择
区别：静态库如果多个 app 使用，则在链接的时候需要完全拷贝到二进制文件中，浪费内存。我们手动打的 framework 都是静态库，作用与.a一致，只不过还额外包含了.h 和一些 source 文件，推荐使用 framework

#### 另外，推荐使用多个小的 framework，避免导致分配大额连续内存导致内存压力

### 3. 小型项目推荐使用 Vendor + Submodule 集成第三方源代码（Project）