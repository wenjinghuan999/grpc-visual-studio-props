# grpc-visual-studio-props
GRPC for windows - Visual Studio .props file

Pre-requisites
--
Git for windows
CMake



Installing grpc-windows
--

Clone grpc from https://github.com/plasticbox/grpc-windows

```bat
git clone https://github.com/plasticbox/grpc-windows
```

Run grpc_clone.bat
Run grpc_build.bat

Note: setting up proxy before running grpc_clone.bat *(Chinese users: This might be the solution you are finding)*
```bat
git config --global http.proxy 127.0.0.1:1080
```


#### Legacy, ignore these

> 
> Clone grpc from https://github.com/grpc/grpc.git   
> ```bat
> git clone https://github.com/grpc/grpc.git
> ```
> 
> Update submodules (IMPORTANT)
> 
> ```bat
> git submodule update 
> ```
> 
> Copy grpc folder to `grpc-windows` (so that `grpc_clone.bat` won't download again.)
> Run `grpc_build.bat`
> 
> #####Installing protobuf
>
>Download `protobuf` from: https://github.com/google/protobuf/releases
>(Note: if you want to use it together with a pre-installed protoc, make sure to match the versions)
>
>```bat
>mkdir .build
>cd .build
>cmake -G "Visual Studio 14 2015 Win64" ../cmake/
>```
>
>Open Visual Studio solution and build. Targets are in `.build/[Debug|Release]/`

Setting up projects
--
Try to use predefined `.props` files and `packages.config` file~
**It seems you have to manually add NuGet packages by right click project - `Manage NuGet packages`. Simply adding `packages.config` to project does not seem to work.** (because package dependencies are defined in `.vcxproj` file)

In summary:
- Add `grpc_debug.props` and `grpc_release.props` via `Property Manager`
- Change the macro `$(GRPC_HOME)` in those `.props` if needed
- Add `grpc.dependencies.*` packages in NuGet package manager (Actually adding `grpc.dependencies.openssl` is enough, the others are dependencies of openssl)
- Build and enjoy~

Or manually
--
```
$(GRPC_HOME) = dir\to\grpc-windows\grpc
```
Add it to `User Macros` under `Property Manager` is a good idea.

INCLUDE PATH: 
```
$(GRPC_HOME)\third_party\protobuf\src
$(GRPC_HOME)\include
``` 

LIBRARY PATH: 
```
$(GRPC_HOME)\bin\protobuf\release
$(GRPC_HOME)\bin\grpc\release
``` 

LINKED LIBRARIES:
```
libprotobuf.lib
gpr.lib
grpc.lib
grpc++.lib
ws2_32.lib
```

NuGet Packages:
```
grpc.dependencies.openssl
grpc.dependencies.openssl.redist
grpc.dependencies.zlib
grpc.dependencies.zlib.redist

//grpc.dependencies.openssl.symbols
//grpc.dependencies.zlib.symbols
```

NOTE:
- Set `C/C++` - `Code Generation` - `Runtime Library` to `/MT`
- Add `_WIN32_WINNT=0x600`(or higher) to `C/C++` - `Preprocessor`-`Preprocessor Definitions`
- Add to `Build events` - `Pre-build events` - `Command` something like
```bat
for %%f in (*.proto) do (
$(GRPC_HOME)\bin\protobuf\release\protoc.exe -I . --grpc_out=. --plugin=protoc-gen-grpc=$GRPC_HOME$\bin\grpc_protoc_plugins\grpc_cpp_plugin.exe %%f
$(GRPC_HOME)\bin\protobuf\release\protoc.exe -I . --cpp_out=. %%f
)
```
