##xcode 项目配置规范

### 1.基于TCKit基础库，Configurations的作用与配置
####1.1 默认配置包含debug跟release，如果使用了jenkins，则对应添加DebugCI跟ReleaseCI
####1.2 基于TCKit库依赖Publish进行打包，Configuration中需要添加Publish，才能保证TCKit库的环境正确

####1.3 与第三方库协作的配置原则，第三方库Configurations中一般默认只有debug跟release，原则就是当主Project打包配置选用Publish而第三方库不包含Publish，默认第三方库使用release

####1.4 对于Preprocessor Macros中 Publish、Release、ReleaseCI 需要主动添加NDEBUG标志，因为许多三方的C库对该字段依赖，以防止系统断言在生产环境生效

###2 静态库.a与framework的选择
####区别：静态库如果多个app使用，则在链接的时候需要完全拷贝到二进制文件中，浪费内存。我们手动打的framework都是静态库，作用与.a一致，只不过还额外包含了.h和一些source文件，推荐使用framework

####另外，推荐使用多个小的framework，避免导致分配大额连续内存导致内存压力

###3 小型项目推荐使用Vendor+submodule集成第三方源代（project）