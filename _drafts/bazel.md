bazel 可以多语言、增量构建。

优点：

1. 加速构建和测试
2. 支持多平台多语言

大规模、多语言、跨平台的复杂项目适合 bazel

缺点：

1. 上手难度高
2. 不适合单语言中小规模
3. 现有项目改造成本高
4. 不同构建系统兼容性差

# Workspace

包含 workspace 文件的目录被认为是一个工作区的根目录，在工作区内，所有源文件、外部依赖和构建过程中生成的文件，都属于该工作区。

workspace 文件中有：

1. 工作区名称
2. 引用的本地目录的外部 Bazel 仓库
    - `local_repository(name, path, repo_mapping)`
3. 从外部下载的 Bazel 仓库
    - `http_archive(name, sha256, strip_prefix, type, url, urls)`
4. 注册工具链

# 包

包含 BUILD 文件的文件夹就是 Bazel 的包（Package）

包是仓库中代码组织的原子单元，一个拥有 BUILD 文件的目录被定义为一个包。包是一组相关文件和它们依赖的集合。

BUILD 文件里有：

1. 加载 bzl 文件中的宏定义和规则
2. 定义制品对象
3. 定义变量
4. 定义测试用例对象

# Bazel 安装

[bazel build install](https://bazel.build/install)

可以通过 cc_library 构建 Bazel 的 C 语言