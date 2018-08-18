学习 http://wiki.ros.org/catkin/CMakeLists.txt 的笔记.

必须按照以下顺序来写,否则会出现编译问题.

1. Required CMake Version (cmake_minimum_required)

2. Package Name (project())

3. Find other CMake/Catkin packages needed for build (find_package())

4. Enable Python module support (catkin_python_setup())

5. Message/Service/Action Generators (add_message_files(), add_service_files(), add_action_files())

6. Invoke message/service/action generation (generate_messages())

7. Specify package build info export (catkin_package())

8. Libraries/Executables to build (add_library()/add_executable()/target_link_libraries())

9. Tests to build (catkin_add_gtest())

10. Install rules (install())

实例:
1. cmake_minimum_required(VERSION 2.8.3) catkin 需要 >=2.8.3 版本.
2. project(robot_brain) 接下来可用 ${PROJECT_NAME} 变量用来表示项目名称.
3. find_package(catkin REQUIRED) 每个包必须要有.
   find_package(catkin REQUIRED COMPONENTS nodelet) 如果还用到其他包的推荐写法,比如:nodelet的写法,此时变量 catkin_INCLUDE_DIRS 既有catkin的包含目录,也有nodelet包的包含目录.
   find_package(nodelet) 不推荐写法,此时变量catkin_INCLUDE_DIRS中没有nodelet包的包含目录.
   find_package() 函数的作用,如果找到了一个包,则会生成一些该包的变量,变量命名规则为 <包名>_属性, 如:
      <包名>_FOUND - 找到包值为 true, 否则为 false
      <包名>_INCLUDE_DIRS or <包名>_INCLUDES - 包导出的包含路径 
      <包名>_LIBRARIES or <包名>_LIBS - 包导出的库
      <包名>_DEFINITIONS - ?
   find_package(Boost REQUIRED COMPONENTS thread) 如果用到 Boost ,需要指明用到 Boost 的哪个模块,比如 thread 模块.

4. 看情况选择是否采用,如果包提供了一些 Python 模块,那么你应该新建一个 setup.py,然后调用 catkin_python_setup().

5. 有3种宏用来处理 message,service,action:
    add_message_files
    add_service_files
    add_action_files
    
6.  generate_messages(DEPENDENCIES std_msgs sensor_msgs) 用来激活生成.
    
    之前的 find_package(catkin REQUIRED COMPONENTS message_generation), 必须有 message_generation .
    随后的 catkin_package() 必须要有对 message_runtime 的运行时依赖!
        catkin_package( ... CATKIN_DEPENDS message_runtime ...)
    文件 package.xml 必须要有对 message_generation 的编译依赖, 对 message_runtime 的运行依赖.
    
    以下根据情况选择是否采用,详情看wiki:
      add_dependencies(some_target ${catkin_EXPORTED_TARGETS}) 
      add_dependencies(some_target ${${PROJECT_NAME}_EXPORTED_TARGETS})
      add_dependencies(some_target ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})


7. catkin_package() 该函数必须在  declaring any targets with add_library() or add_executable() 之前调用!该函数包含5个可选参数:
    INCLUDE_DIRS - 项目导出的头文件的位置（即cflags）
    LIBRARIES - 项目导出的库
    CATKIN_DEPENDS - 项目依赖的其他 catkin 项目 
    DEPENDS - 项目依赖的 非catking项目: Non-catkin CMake projects.
    CFG_EXTRAS - 其他配置选项
    例子:
        catkin_package(
           INCLUDE_DIRS include
           LIBRARIES ${PROJECT_NAME}
           CATKIN_DEPENDS roscpp nodelet 编译/运行所依赖的catkin包项
           DEPENDS eigen opencv) 编译/运行所系统依赖项
           
           
编译生成两种文件: 可执行目标文件,库目标文件.CMake内部要求目标文件的名字要独一无二,它不管目标文件的位置.不过可以通过以下方式重命名.
    set_target_properties(rviz_image_view
                          PROPERTIES OUTPUT_NAME image_view 重命名后的名字
                          PREFIX "")
include_directories(include ${Boost_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS}) 包含生成目标用到的头文件目录, *_INCLUDE_DIRS 变量是 find_package() 生成的. include 表示包目录下的 include 目录也是头文件目录.


8. add_executable(myProgram src/main.cpp src/some_file.cpp src/another_file.cpp) 从3个源文件生成一个可执行文件 myProgram.
   add_library(${PROJECT_NAME} ${${PROJECT_NAME}_SRCS}) 生成库目标文件${PROJECT_NAME},默认是 shared libraries.
   target_link_libraries(<executableTargetName>, <lib1>, <lib2>, ... <libN>) 生成可执行文件需要链接的库文件.
   
   Example 见 wiki.
   
9. 根据情况是否采用, 基于 gtest 的单元测试
    if(CATKIN_ENABLE_TESTING)
      catkin_add_gtest(myUnitTest test/utest.cpp)
    endif()
    
10. 可选,指定可安装目标.
    编译完成后,目标放在 catkin 工作空间的 devel 目录. 不过我们通常想把目标放在系统目录. 如果你想要你的代码有 make install 功能,你需要指定目标安装位置.
    这可通过 install() 函数实现,有四个参数可选:
        TARGETS - which targets to install
        ARCHIVE DESTINATION - Static libraries and DLL (Windows) .lib stubs
        LIBRARY DESTINATION - Non-DLL shared libraries and modules
        RUNTIME DESTINATION - Executable targets and DLL (Windows) style shared libraries        
    标准安装例子:
      install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
      )
    包含 Python bindings 的库的特殊安装例子:
      install(TARGETS python_module_library
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_PYTHON_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_PYTHON_DESTINATION}
      )
        
    安装 python 的可执行脚本,因为 python 没有 add_library() and add_executable(),需要使用下面的方式,在你的 CMakeLists.txt 文件中:
      catkin_install_python(
        PROGRAMS scripts/myscript
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

    安装头文件:
      install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
        PATTERN ".svn" EXCLUDE
      )
      或者:
      install(DIRECTORY include/
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
        PATTERN ".svn" EXCLUDE
      )

    安装 roslaunch 文件,其它资源:
      install(DIRECTORY launch/
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
        PATTERN ".svn" EXCLUDE)

