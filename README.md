VTM Code Guide BOOK
==============================
Tips
*updata1：*生成了Partitioner的ClassDiagram.cd，可以产看Partitioner类的UML类图
*argues: *ENABLE_TRACE :0→1

前缀、后缀、缩写
------------

---p        :pointer    ;指针
---c        :const,class    ;静态，类
---i        :int    ;int型

-PLT- PaLetTe 调色板模式(SCC)
-SBT- Sub-Block Transform
-ft- feature

常见数据结构
---------------
**CS**

*data:*

1.有cus,pus,tus,三个vector<* ~>，应该是链接到向其内部的CU，PU，TU

2.CS（应该）对应于某个区域UnitArea

3.包含(区域内的)特征如cost,dist,fracBits, ETM_type, ETM_opts等

*function:*
```C++
cs.initSubStructure(_cs, chType, subCUArea, isTuEnc)
```
根据param2,param3,param4初始化

```c++
cs.getPredBuf(UnitArea)
cs.getRecoBuf(...)
cs.getResiBuf(...)
cs.getOrgResiBuf(...)
```

获取UnitArea区对应的预测块
```c++
cs.useSubStructure(subCS, chType, subArea(_UnitArea), cpyPrea, cpyReco, cpyOrgResi, cpyResi, updateCost)
```
将subCS的Pred,Reco,Resi,OrgResi等复制到picture中，甚至是parentcs(根据flag)

**Partitioner**

*UnitArea*

包含当前Partitioner对应的UnitArea

*.m_partStack，partitioningStack*

栈，应该是存储每个CS对应的partitioner

*partLevel*
应该对应于一个partitioner中的一种encTestMode的东西


每个UnitArea应该都对应有CS、Partitioner、PartLevel
（INTRA）在check split类型的时候会push N个PartLevel入（m_partStack）栈

**ComprCUCtx**
存储一个CU在compress时各种数据

**m_ComprCUCtxList**
存储CTU内所有CU在compress时各种数据

**EncModeCtrl**
模式控制类


常见（可能会用到）函数
------


**Slice::getCtuAddrInSlice(ctuIdx);**

**CS::getCURestricted(offsetPOS, currPOS, currSlice, currTile, chType)**
example:   
```c++
const CodingUnit* cuLeft   = cs.getCURestricted( pos.offset( -1,  0 ), pos, curSliceIdx, curTileIdx, chType );
const CodingUnit* cuBelowLeft   = cs.getCURestricted( pos.offset( -1, currArea().blocks[chType].height), pos, curSliceIdx, curTileIdx, chType );
  ```

**CS::getCU(pos, partitioner)**

**EncCU::CompressCtu(CodingStructure, UnitArea, ctuRsAddr,prevQP[], currQP[])**

传入CTU对应的CS，UnitArea以及S=CTU绝对地址和两个QP数组
此函数中，创建一个Partitioner；创建一个相同大小的tempCS和bestCU，传入xCompressCU并修改值，将tempCS和bestCS设为传入参数CS的sub-CS
调用xCompressCU, xCompressCU形成递归。并将最终的bestCS划分一级一级传入其父CS
（*注意，VVC的结构是把划分与预测当中一种类型，在这些类型中switch，选择CU需要进行的操作，如帧内预测、帧间预测、SKIP、划分决策等等*）

**EncCU::xCompressCU(tempCS, bestCS, Partitioner)**
传入当前CS的sub tempCS和bestCS，以及当前CS的Partitioner

**adaptiveDepthPartitioner::setMaxMinDepth(min, max, cs)**
限制划分深度,传入min，max并更新

**inline PartSplit getPartSplit( encTestmode )**
根据encTestMode.type确定划分类型

*我要死死死死死死死死死了的函数*

**CABACWriter::split_cu_mode（）**
What the fuck is：

**sub.resize( 4, cuArea );**
???


























This software package is the reference software for Versatile Video Coding (VVC). The reference software includes both encoder and decoder functionality.

Reference software is useful in aiding users of a video coding standard to establish and test conformance and interoperability, and to educate users and demonstrate the capabilities of the standard. For these purposes, this software is provided as an aid for the study and implementation of Versatile Video Coding.

The software has been jointly developed by the ITU-T Video Coding Experts Group (VCEG, Question 6 of ITU-T Study Group 16) and the ISO/IEC Moving Picture Experts Group (MPEG, Working Group 11 of Subcommittee 29 of ISO/IEC Joint Technical Committee 1).

A software manual, which contains usage instructions, can be found in the "doc" subdirectory of this software package.

Build instructions
==================

The CMake tool is used to create platform-specific build files.

Although CMake may be able to generate 32-bit binaries, **it is generally suggested to build 64-bit binaries**. 32-bit binaries are not able to access more than 2GB of RAM, which will not be sufficient for coding larger image formats. Building in 32-bit environments is not tested and will not be supported.


Build instructions for plain CMake (suggested)
--------------------------------------------------

**Note:** A working CMake installation is required for building the software.

CMake generates configuration files for the compiler environment/development environment on each platform.
The following is a list of examples for Windows (MS Visual Studio), macOS (Xcode) and Linux (make).

Open a command prompt on your system and change into the root directory of this project.

Create a build directory in the root directory:
```bash
mkdir build
```

Use one of the following CMake commands, based on your platform. Feel free to change the commands to satisfy
your needs.

**Windows Visual Studio 2015/17/19 64 Bit:**

Use the proper generator string for generating Visual Studio files, e.g. for VS 2015:

```bash
cd build
cmake .. -G "Visual Studio 14 2015 Win64"
```

Then open the generated solution file in MS Visual Studio.

For VS 2017 use "Visual Studio 15 2017 Win64", for VS 2019 use "Visual Studio 16 2019".

Visual Studio 2019 also allows you to open the CMake directory directly. Choose "File->Open->CMake" for this option.

**macOS Xcode:**

For generating an Xcode workspace type:
```bash
cd build
cmake .. -G "Xcode"
```
Then open the generated work space in Xcode.

For generating Makefiles with optional non-default compilers, use the following commands:

```bash
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=gcc-9 -DCMAKE_CXX_COMPILER=g++-9
```
In this example the brew installed GCC 9 is used for a release build.

**Linux**

For generating Linux Release Makefile:
```bash
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
```
For generating Linux Debug Makefile:
```bash
cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug
```

Then type
```bash
make -j
```

For more details, refer to the CMake documentation: https://cmake.org/cmake/help/latest/

Build instructions for make
---------------------------

**Note:** The build instructions in this section require the make tool and Python to be installed, which are
part of usual Linux and macOS environments. See below for installation instruction for Python and GnuWin32
on Windows.

Open a command prompt on your system and change into the root directory of this project.

To use the default system compiler simply call:
```bash
make all
```


**MSYS2 and MinGW (Windows)**

**Note:** Build files for MSYS MinGW were added on request. The build platform is not regularily tested and can't be supported.

Open an MSYS MinGW 64-Bit terminal and change into the root directory of this project.

Call:
```bash
make all toolset=gcc
```

The following tools need to be installed for MSYS2 and MinGW:

Download CMake: http://www.cmake.org/ and install it.

Python and GnuWin32 are not mandatory, but they simplify the build process for the user.

python:    https://www.python.org/downloads/release/python-371/

gnuwin32:  https://sourceforge.net/projects/getgnuwin32/files/getgnuwin32/0.6.30/GetGnuWin32-0.6.3.exe/download

To use MinGW, install MSYS2: http://repo.msys2.org/distrib/msys2-x86_64-latest.exe

Installation instructions: https://www.msys2.org/

Install the needed toolchains:
```bash
pacman -S --needed base-devel mingw-w64-i686-toolchain mingw-w64-x86_64-toolchain git subversion mingw-w64-i686-cmake mingw-w64-x86_64-cmake
```
