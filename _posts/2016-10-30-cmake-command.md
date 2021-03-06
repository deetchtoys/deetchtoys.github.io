---
layout: post
title: "cmake学习"
subtitle: "CMakeLists.txt 的编写"
date: 2016-10-30 16:35:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - cmake
---

> "学会编写CMakeLists.txt,参考[^1],最好参考原PDF，写的比较详尽，这里只是做一些记录，方便自己查阅。"

### PROJECT(projectname [CXX] [C] [Java])
这个指令隐式的定义了两个cmake变量：\<projectname\>_BINARY_DIR 以及 \<projectname\>_SOURCE_DIR  
在内部编译下，这两个变量都是指向工程所在目录；  
在外部编译下，\<projectname\>_BINARY_DIR指向编译目录  
同时，cmake系统也帮助我们预定义可PROJECT_BINARY_DIR, PROJECT_SOURCE_DIR  

### SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
SET(SRC_LIST main.c t1.c t2,c)  
不论是SUBDIRS还是ADD_SUBDIRECTORY指令(不论是否指定编译输出目录)，我们都可以通过通过SET指令重新定义  
EXECUTABLE_OUTPUT_PATH和LIBRARY_OUTPUT_PATH变量来指定最终的目标二进制的位置(指最终生成的hello或者  
最终的共享库，不包含编译生成的中间文件)：  
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)  
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)  
在哪里ADD_EXECUTABLE或ADD_LIBRARY，如果需要改变目标存放路径，就在哪里加入上述的定义。

### MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display" ...)
这个指令用于向终端输出用户定义的信息，包含了三种类型：  
SEND_ERROR，产生错误，生成过程被跳过  
STATUS, 输出前缀为 - 的信息  
FATAL_ERROR, 立即终止所有cmake过程  

### ADD_EXECUTABLE(hello ${SRC_LIST})

### ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
这个指令用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。  
EXCLUDE_FORM_ALL参数的含义是将这个目录从编译过程中排除。

### CMAKE_INSTALL_PREFIX
CMAKE_INSTALL_PREFIX变量类似于configure脚本的 -prefix，常见的使用方法看起来是这个样子：  
cmake -DCMAKE_INSTALL_PREFIX=/usr .

### INSTALL指令
1.目标文件的安装:
  
~~~
  INSTALL(TARGETS targets ...
      [[ARCHIVE|LIBRARY|RUNTIME]
       [DESTINATION <dir>]
       [PERMISSIONS permissions ...]
       [CONFIGURATIONS
          [Debug|Release|...]]
       [COMPONENT <component>]
       [OPTIONAL]
      ] [...])
~~~
参数中的TARGETS后面跟的就是我们通过ADD_EXECUTABLE或者ADD_LIBRARY定义的目标文件，可能是可执行二进制、动态库、静态库。  
目标类型也就相对应的有三种，ARCHIVE特指静态库，LIBRARY特指动态库，RUNTIME特指可执行目标二进制。  
DESTINATION定义了安装的路径，如果路径以“/”开头，那么指绝对路径，这时候CMAKE_INSTALL_PREFIX其实就无效了。  
如果你希望使用CMAKE_INSTALL_PREFIX来定义安装路径，就要写成相对路径，即不要以"/"开头，那么安装后的路径就是：  
${CMAKE_INSTALL_PREFIX}/<DESTINATION定义的路径>  
简单举个例子：
  
~~~
  INSTALL(TARGETS myrun mylib mystaticlib
      RUNTIME DESTINATION bin
      LIBRARY DESTINATION lib
      ARCHIVE DESTINATION libstatic
      )
~~~
上面的例子将会：  
可执行二进制myrun安装到${CMAKE_INSTALL_PREFIX}/bin目录  
动态库libmylib安装到${CMAKE_INSTALL_PREFIX}/lib目录  
静态库libmystaticlib安装到${CMAKE_INSTALL_PREFIX}/libstatic目录  
特别注意的是你不需要关心TARGETS具体生成的路径，只需要写上TARGETS名称就可以了  

2.普通文件的安装：
  
~~~
  INSTALL(FILES files... DESTINATION <dir>
      [PERMISSIONS permissions...]
      [CONFIGURATIONS [Debug|Release|...]]
      [COMPONENT <component>]
      [RENAME <name>] [OPTIONAL])
~~~
可用于安装一般文件,并可以指定访问权限,文件名是此指令所在路径下的相对路径。如果  
默认不定义权限 PERMISSIONS,安装后的权限为:  
OWNER_WRITE, OWNER_READ,  
GROUP_READ,和 WORLD_READ,即 644 权限。  

3.非目标文件的可执行程序安装(比如脚本之类):
  
~~~
  INSTALL(PROGRAMS files... DESTINATION <dir>
      [PERMISSIONS permissions...]
      [CONFIGURATIONS [Debug|Release|...]]
      [COMPONENT <component>]
      [RENAME <name>] [OPTIONAL])
~~~
跟上面的 FILES 指令使用方法一样,唯一的不同是安装后权限为:  
OWNER_EXECUTE, GROUP_EXECUTE, 和 WORLD_EXECUTE,即 755 权限
  
4.目录的安装:
  
~~~
  INSTALL(DIRECTORY dirs... DESTINATION <dir>
      [FILE_PERMISSIONS permissions...]
      [DIRECTORY_PERMISSIONS permissions...]
      [USE_SOURCE_PERMISSIONS]
      [CONFIGURATIONS [Debug|Release|...]]
      [COMPONENT <component>]
      [[PATTERN <pattern> | REGEX <regex>]
      [EXCLUDE] [PERMISSIONS permissions...]] [...])
~~~
这里主要介绍其中的 DIRECTORY、PATTERN 以及 PERMISSIONS 参数。  
DIRECTORY 后面连接的是所在 Source 目录的相对路径,但务必注意:  
abc 和 abc/有很大的区别。  
如果目录名不以/结尾,那么这个目录将被安装为目标路径下的 abc,如果目录名以/结尾,  
代表将这个目录中的内容安装到目标路径,但不包括这个目录本身。  
PATTERN 用于使用正则表达式进行过滤,PERMISSIONS 用于指定 PATTERN 过滤后的文件权限。

我们来看一个例子:
  
~~~
  INSTALL(DIRECTORY icons scripts/ DESTINATION share/myprojPATTERN "CVS" EXCLUDE
      PATTERN "scripts/*"
      PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
      GROUP_EXECUTE GROUP_READ)
~~~
这条指令的执行结果是:  
将 icons 目录安装到 \<prefix\>/share/myproj,将 scripts/中的内容安装到\<prefix\>/share/myproj  
不包含目录名为 CVS 的目录,对于 scripts/* 文件指定权限为 OWNER_EXECUTE  
OWNER_WRITE OWNER_READ GROUP_EXECUTE GROUP_READ.  

5.安装时 CMAKE 脚本的执行:  
  
~~~
  INSTALL([[SCRIPT <file>] [CODE <code>]] [...])
~~~
SCRIPT 参数用于在安装时调用 cmake 脚本文件(也就是<abc>.cmake 文件)  
CODE 参数用于执行 CMAKE 指令,必须以双引号括起来。比如:  
INSTALL(CODE "MESSAGE(\"Sample install message.\")")  
安装还有几个被标记为过时的指令,比如 INSTALL_FILES 等,这些指令已经不再推荐使用,所以,这里就不再赘述了。  

### 编译共享库
  
~~~
  ADD_LIBRARY(libname [SHARED|STATIC|MODULE] [EXCLUDE_FROM_ALL] source1 source2 ... sourceN)
~~~
类型有三种：  
SHARED,动态库  
STATIC，静态库  
MODULE，在使用dyld的系统有效，如果不支持dyld，则被当作SHARED对待  
  
EXCLUDE_FROM_ALL参数的意思是这个库不会被默认构建，除非有其他的组件依赖或者手工构建。  
  
~~~
  SET(LIBHELLO_SRC hello.c)
  ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
~~~
  
~~~
  SET(LIBHELLO_SRC hello.c)
  ADD_LIBRARY(hello STATIC ${LIBHELLO_SRC})
~~~
  以上方法的缺点是动态库和静态库不能同名，因为TARGET是不能重名的
  
~~~
  SET_TARGET_PROPERTIES(target1 target2 ... PROPERTIES prop1 value1 prop2 value2 ...)
~~~
这条指令可以用来设置输出的名称，对于动态库，还可以用来指定动态库版本和API版本。如下：
  
~~~
  SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")
~~~
  
~~~
  GET_TARGET_PROPERTY(VAR target property)
  具体用法：
  GET_TARGET_PROPERTY(OUTPUT_VALUE hello_static OUTPUT_NAME)
  MESSAGE(STATUS “This is the hello_static OUTPUT_NAME:”${OUTPUT_VALUE})
~~~
  
### INCLUDE_DIRECTORIES
  
~~~
  INCLUDE_DIRECTORIES([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)
~~~
  这条指令可以用来向工程添加多个特定的头文件搜索路径,路径之间用空格分割,如果路径  
  中包含了空格,可以使用双引号将它括起来,默认的行为是追加到当前的头文件搜索路径的  
  后面,你可以通过两种方式来进行控制搜索路径添加的方式:  
  1,CMAKE_INCLUDE_DIRECTORIES_BEFORE,通过 SET 这个 cmake 变量为 on,可以
  将添加的头文件搜索路径放在已有路径的前面。  
  2,通过 AFTER 或者 BEFORE 参数,也可以控制是追加还是置前。  
  
~~~
  INCLUDE_DIRECTORIES(/usr/include/hello)
~~~

### LINK_DIRECTORIES 和 TARGET_LINK_LIBRARIES
~~~
  LINK_DIRECTORIES(directory1 directory2 ...)
~~~
  这个指令添加非标准的共享库搜索路径。比如,在工程内部同时存在共享库和可执行二进制,在编译时就需要指定一下这些共享库的路径  
  
~~~
  TARGET_LINK_LIBRARIES(target library1 <debug | optimized> library2 ...)
~~~
  这个指令可以用来为 target 添加需要链接的共享库,本例中是一个可执行文件,但是同样可以用于为自己编写的共享库添加共享库链接。  
  
~~~
  TARGET_LINK_LIBRARIES(main hello)
  也可以写成
  TARGET_LINK_LIBRARIES(main libhello.so)
  链接静态库：
  TARGET_LINK_LIBRARIES(main libhello.a)
  或者
  TARGET_LINK_LIBRARIES(main hello_static)
~~~
  
### 特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_PATH
  务必注意,这两个是环境变量而不是 cmake 变量。
  使用方法是要在 bash 中用 export 或者在 csh 中使用 set 命令设置或者
  CMAKE_INCLUDE_PATH=/home/include cmake ..等方式。
  
  前面我们直接使用了绝对路径 INCLUDE_DIRECTORIES(/usr/include/hello)告诉工程这个头文件目录。
  为了将程序更智能一点,我们可以使用 CMAKE_INCLUDE_PATH 来进行,使用 bash 的方法
  如下:
  
~~~
  export CMAKE_INCLUDE_PATH=/usr/include/hello
  然后在头文件中将 INCLUDE_DIRECTORIES(/usr/include/hello)替换为:
  FIND_PATH(myHeader hello.h)
  IF(myHeader)
  INCLUDE_DIRECTORIES(${myHeader})
  ENDIF(myHeader)
~~~
  FIND_PATH(myHeader NAMES hello.h PATHS /usr/include /usr/include/hello)  
  
  如果你不使用 FIND_PATH,CMAKE_INCLUDE_PATH 变量的设置是没有作用的.  
  
  以此为例,CMAKE_LIBRARY_PATH 可以用在 FIND_LIBRARY 中。  
  
### ADD_DEFINITIONS
  向 C/C++编译器添加-D 定义,比如:  
  ADD_DEFINITIONS(-DENABLE_DEBUG -DABC),参数之间用空格分割。  
  如果你的代码中定义了#ifdef ENABLE_DEBUG #endif,这个代码块就会生效。  
  如果要添加其他的编译器开关,可以通过 CMAKE_C_FLAGS 变量和 CMAKE_CXX_FLAGS 变量设置。  
  
### ADD_DEPENDENCIES
  定义 target 依赖的其他 target,确保在编译本 target 之前,其他的 target 已经被构建。  
  ADD_DEPENDENCIES(target-name depend-target1 depend-target2 ...)  
  
### ADD_TEST 与 ENABLE_TESTING 指令
  ENABLE_TESTING 指令用来控制 Makefile 是否构建 test 目标,涉及工程所有目录。语  
  法很简单,没有任何参数,ENABLE_TESTING(),一般情况这个指令放在工程的主CMakeLists.txt 中.
  ADD_TEST 指令的语法是:  
  ADD_TEST(testname Exename arg1 arg2 ...)  
  testname 是自定义的 test 名称,Exename 可以是构建的目标文件也可以是外部脚本等  
  等。后面连接传递给可执行文件的参数。如果没有在同一个 CMakeLists.txt 中打开  
  ENABLE_TESTING()指令,任何 ADD_TEST 都是无效的。比如我们前面的 Helloworld 例子,可以在工程主 CMakeLists.txt 中添加  
  ADD_TEST(mytest ${PROJECT_BINARY_DIR}/bin/main) ENABLE_TESTING()  
  生成 Makefile 后,就可以运行 make test 来执行测试了。  
  
### AUX_SOURCE_DIRECTORY
  基本语法是:  
  AUX_SOURCE_DIRECTORY(dir VARIABLE)  
  作用是发现一个目录下所有的源代码文件并将列表存储在一个变量中,这个指令临时被用来  
  自动构建源文件列表。因为目前 cmake 还不能自动发现新添加的源文件。  
  比如:  
  AUX_SOURCE_DIRECTORY(. SRC_LIST)  
  ADD_EXECUTABLE(main ${SRC_LIST})  
  你也可以通过后面提到的 FOREACH 指令来处理这个 LIST  
  
### EXEC_PROGRAM
  在 CMakeLists.txt 处理过程中执行命令,并不会在生成的 Makefile 中执行。具体语法为:
  
~~~
  EXEC_PROGRAM(Executable [directory in which to run]
      [ARGS <arguments to executable>]
      [OUTPUT_VARIABLE <var>]
      [RETURN_VALUE <var>])
~~~
  用于在指定的目录运行某个程序,通过 ARGS 添加参数,如果要获取输出和返回值,可通过  
  OUTPUT_VARIABLE 和 RETURN_VALUE 分别定义两个变量.  
  这个指令可以帮助你在 CMakeLists.txt 处理过程中支持任何命令,比如根据系统情况去  
  修改代码文件等等。  
  举个简单的例子,我们要在 src 目录执行 ls 命令,并把结果和返回值存下来。可以直接在 src/CMakeLists.txt 中添加:  
  
~~~
  EXEC_PROGRAM(ls ARGS "*.c" OUTPUT_VARIABLE LS_OUTPUT RETURN_VALUE
  LS_RVALUE)
  IF(not LS_RVALUE)
  MESSAGE(STATUS "ls result: " ${LS_OUTPUT})
  ENDIF(not LS_RVALUE)
~~~
  在 cmake 生成 Makefile 的过程中,就会执行 ls 命令,如果返回 0,则说明成功执行,  
  那么就输出 ls *.c 的结果。关于 IF 语句,后面的控制指令会提到。  
  
### FILE 指令
  文件操作指令,基本语法为:
  
~~~
  FILE(WRITE filename "message to write"... )
  FILE(APPEND filename "message to write"... )
  FILE(READ filename variable)
  FILE(GLOB
  expressions]...)
  variable [RELATIVE path] [globbing
  FILE(GLOB_RECURSE variable [RELATIVE path]
  [globbing expressions]...)
  FILE(REMOVE [directory]...)
  FILE(REMOVE_RECURSE [directory]...)
  FILE(MAKE_DIRECTORY [directory]...)
  FILE(RELATIVE_PATH variable directory file)
  FILE(TO_CMAKE_PATH path result)
  FILE(TO_NATIVE_PATH path result)
~~~
  这里的语法都比较简单,不在展开介绍了。
  
### INCLUDE 指令
  用来载入 CMakeLists.txt 文件,也用于载入预定义的 cmake 模块.  
  INCLUDE(file1 [OPTIONAL])  
  INCLUDE(module [OPTIONAL])  
  OPTIONAL 参数的作用是文件不存在也不会产生错误。  
  你可以指定载入一个文件,如果定义的是一个模块,那么将在 CMAKE_MODULE_PATH 中搜索这个模块并载入。  
  载入的内容将在处理到 INCLUDE 语句是直接执行。  
  
### FIND 指令
  FIND_系列指令主要包含一下指令:  
  FIND_FILE(<VAR> name1 path1 path2 ...)  
  VAR 变量代表找到的文件全路径,包含文件名  
  FIND_LIBRARY(<VAR> name1 path1 path2 ...)  
  VAR 变量表示找到的库全路径,包含库文件名  
  FIND_PATH(<VAR> name1 path1 path2 ...)  
  VAR 变量代表包含这个文件的路径。  
  FIND_PROGRAM(<VAR> name1 path1 path2 ...)  
  VAR 变量代表包含这个程序的全路径。  
  FIND_PACKAGE(<name> [major.minor] [QUIET] [NO_MODULE]  
  [[REQUIRED|COMPONENTS] [componets...]])  
  用来调用预定义在 CMAKE_MODULE_PATH 下的 Find<name>.cmake 模块,你也可以自己  
  定义 Find<name>模块,通过 SET(CMAKE_MODULE_PATH dir)将其放入工程的某个目录  
  中供工程使用,我们在后面的章节会详细介绍 FIND_PACKAGE 的使用方法和 Find 模块的编写。  
  FIND_LIBRARY 示例:  
  FIND_LIBRARY(libX X11 /usr/lib) 
  IF(NOT libX)  
  MESSAGE(FATAL_ERROR “libX not found”)  
  ENDIF(NOT libX)  
  
### 控制命令
  1.IF 指令,基本语法为:  
  
~~~
      IF(expression)
      # THEN section.
      COMMAND1(ARGS ...)COMMAND2(ARGS ...)
      ...
      ELSE(expression)
      # ELSE section.
      COMMAND1(ARGS ...)
      COMMAND2(ARGS ...)
      ...
      ENDIF(expression)
~~~
  另外一个指令是 ELSEIF,总体把握一个原则,凡是出现 IF 的地方一定要有对应的  
  ENDIF.出现 ELSEIF 的地方,ENDIF 是可选的。  
  表达式的使用方法如下:  
  IF(var),如果变量不是:空,0,N, NO, OFF, FALSE, NOTFOUND 或  
  <var>_NOTFOUND 时,表达式为真。  
  IF(NOT var ),与上述条件相反。  
  IF(var1 AND var2),当两个变量都为真是为真。  
  IF(var1 OR var2),当两个变量其中一个为真时为真。  
  IF(COMMAND cmd),当给定的 cmd 确实是命令并可以调用是为真。  
  IF(EXISTS dir)或者 IF(EXISTS file),当目录名或者文件名存在时为真。  
  IF(file1 IS_NEWER_THAN file2),当 file1 比 file2 新,或者 file1/file2 其  
  中有一个不存在时为真,文件名请使用完整路径。  
  IF(IS_DIRECTORY dirname),当 dirname 是目录时,为真。  
  IF(variable MATCHES regex)  
  IF(string MATCHES regex)  
  当给定的变量或者字符串能够匹配正则表达式 regex 时为真。比如:  
  IF("hello" MATCHES "ell")  
  MESSAGE("true")  
  ENDIF("hello" MATCHES "ell")IF(variable LESS number)  
  IF(string LESS number)  
  IF(variable GREATER number)  
  IF(string GREATER number)  
  IF(variable EQUAL number)  
  IF(string EQUAL number)  
  数字比较表达式  
  IF(variable STRLESS string)  
  IF(string STRLESS string)  
  IF(variable STRGREATER string)  
  IF(string STRGREATER string)  
  IF(variable STREQUAL string)  
  IF(string STREQUAL string)  
  按照字母序的排列进行比较.  
  IF(DEFINED variable),如果变量被定义,为真。  
  一个小例子,用来判断平台差异:  
  IF(WIN32)  
  MESSAGE(STATUS “This is windows.”)  
  #作一些 Windows 相关的操作  
  ELSE(WIN32)  
  MESSAGE(STATUS “This is not windows”)  
  #作一些非 Windows 相关的操作  
  ENDIF(WIN32)  
  上述代码用来控制在不同的平台进行不同的控制,但是,阅读起来却并不是那么舒服,  
  ELSE(WIN32)之类的语句很容易引起歧义。  
  这就用到了我们在 “ 常用变量 ” 一节提到的 CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS 开关。
  可以 SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)  
  这时候就可以写成:  
  IF(WIN32)  
  ELSE()  
  ENDIF()如果配合 ELSEIF 使用,可能的写法是这样:  
  IF(WIN32)  
  #do something related to WIN32  
  ELSEIF(UNIX)  
  #do something related to UNIX  
  ELSEIF(APPLE)  
  #do something related to APPLE  
  ENDIF(WIN32)  
  2.WHILE  
  
~~~
  WHILE 指令的语法是:
  WHILE(condition)
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
  ENDWHILE(condition)
~~~
  其真假判断条件可以参考 IF 指令。  
  3,FOREACH  
  FOREACH 指令的使用方法有三种形式:  
  
  a.列表  
  
~~~
  FOREACH(loop_var arg1 arg2 ...)
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
  ENDFOREACH(loop_var)
~~~
  像我们前面使用的 AUX_SOURCE_DIRECTORY 的例子  
  AUX_SOURCE_DIRECTORY(. SRC_LIST)  
  FOREACH(F ${SRC_LIST})  
  MESSAGE(${F})  
  ENDFOREACH(F)  
  
  b.范围  
  
~~~
  FOREACH(loop_var RANGE total)
  ENDFOREACH(loop_var)
~~~
  从 0 到 total 以1为步进举例如下:  
  FOREACH(VAR RANGE 10)  
  MESSAGE(${VAR})  
  ENDFOREACH(VAR)  
  最终得到的输出是:  
  0  
  1  
  2  
  3  
  4  
  5  
  6  
  7  
  8  
  9  
  10  
  
  c.范围和步进  
  
~~~
  FOREACH(loop_var RANGE start stop [step])
  ENDFOREACH(loop_var)
~~~
  从 start 开始到 stop 结束,以 step 为步进,  
  举例如下  
  FOREACH(A RANGE 5 15 3)  
  MESSAGE(${A})  
  ENDFOREACH(A)  
  最终得到的结果是:  
  5  
  8  
  11  
  14  
  这个指令需要注意的是,知道遇到 ENDFOREACH 指令,整个语句块才会得到真正的执行。  


### tips
make clean 即可对构建结果进行清理  
make distclean 是清理构建过程中产生的中间文件的，cmake不支持  



### 附录

[^1]: cmake实践.pdf
