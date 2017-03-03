+++
tags = ["lua", "luarocks", "zerobrane"]
date = "2016-09-23T17:00:46+05:30"
description = "with Lua Rocks and ZeroBrane Studio"
title = "Setup Lua on Windows"

+++

# Ingredients

* CMake
* MinGW compiler (one of MinGW, TDM, MinGW-W64, …)
* Lua binaries
* LuaRocks setup binaries
* ZeroBrane Studio

# Steps Involved

0. Make sure the commands `cmake` and `mingw32-gcc` are accessible from a general _Command Prompt_
    + Install CMake; straight-forward
    + Install MinGW64; choose `i686-6.2.0-posix-dwarf-rt_v5-rev0`.  Both archive and installer works.
    + mingw-w64 project's MinGW only has `gcc.exe` while luarocks expects `mingw32-gcc.exe`; create a symbolic link (`mklink`)
1. Download `lua-5.3.3_Win32_bin.zip` from [LuaForge Lua Binaries project][]
    + Make sure Lua and MinGW are of the same arch
    + This contains the binaries (`.exe`s and `.dll`)
    + Put them in the `/bin` directory e.g. `F:\Apps\Lua\bin`
    + This package does not have the `lua` and `luac` executables; make links if necessary to the versioned ones
    + LuaBinaries is preferred over [Joe DF's Builds][] as it depends on `libgcc_s_dw2-1.dll`; distributed as a separate download
2. Download `lua-5.3.3_Win32_dllw4_lib.zip` from [LuaForge Lua Binaries project][]
    + This has the includes and libraries (except the `.dll` everything else is needed)
    + Put them in the right places: `F:\Apps\Lua\lib` and `F:\Apps\Lua\include` -- although `inc` would be a better name, _LuaRocks_, by default, looks for `include`
3. Put the `bin` directory in `$PATH`; Lua 5.3 should work now from Command Prompt.
4. Extract `luarocks-2.3.0-win32.zip` somewhere (this can be removed post installation); `cd` to it from an elevated prompt.
5. Run `install.bat /LV 5.3 /LUA F:\Apps\Lua /P F:\Apps\LuaRocks /MW`
    + This should successfully install LuaRocks. Save the logs for the information on `ENV` variables.
    + The LuaRocks installer directory that you extracted may be removed now.
6. Go to the installed directory, search and install a rock.
    + `luarocks install luafilesystem`
7. `l = require lfs` should now evaluate to something non-nil.
8. Configure *ZeroBrane Studio* to use this installation of Lua 5.3 interpreter by adding this line to the user configuration file `user.lua`:
{{< highlight lua >}}
path.lua53 = 'F:/Apps/Lua/bin/lua.exe'
{{< /highlight >}}
9. After successful installation, adding the environment variables mandated by `install.bat` wasn't needed when making ZBS use this Lua.
    + Earlier setting `PATH`, `LUA_PATH` and `LUA_CPATH` for LuaRocks and System rocktree were needed
    + If adding, *do not* change `/` into `\` and keep the `?`s

[LuaForge Lua Binaries project]: http://luabinaries.luaforge.net/
[Joe DF's Builds]: http://joedf.users.sourceforge.net/luabuilds/
