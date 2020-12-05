# Embedding Lua in CPP

<!-- TOC Marked -->

* [Headers needed](#headers-needed)
* [Tell Lua to execute a command and get the result in cpp](#tell-lua-to-execute-a-command-and-get-the-result-in-cpp)
* [Use Lua libs](#use-lua-libs)
* [Execute Lua file](#execute-lua-file)
* [Use Lua script to provide datas](#use-lua-script-to-provide-datas)
* [Use Lua table](#use-lua-table)
* [Get Lua function in CPP program](#get-lua-function-in-cpp-program)
* [Define Lua function in CPP program and call it in Lua](#define-lua-function-in-cpp-program-and-call-it-in-lua)

<!-- /TOC -->

## Headers needed

```cpp
extern "C"
{
#include <lauxlib.h>
#include <lualib.h>
#include <lua.h>
}
```

**Notice** the spelling of `lauxlib.h`.

## Tell Lua to execute a command and get the result in cpp

```cpp
// g++ lua_cpp.cpp -o lua_cpp.exe -llua
#include <iostream>

extern "C"
{
#include <lauxlib.h>
#include <lualib.h>
#include <lua.h>
}

int main()
{
    std::string cmd = "a = 7 + 11";

    lua_State* L = luaL_newstate();

    int r = luaL_dostring(L, cmd.c_str());

    if (r == LUA_OK)
    {
        lua_getglobal(L, "a");
        if (lua_isnumber(L, -1))
        {
            float a_in_cpp = (float)lua_tonumber(L, -1);
            std::cout << "a in cpp: " << a_in_cpp << std::endl;
        }
    }
    else
    {
        std::string errormsg = lua_tostring(L, -1);
        std::cerr << errormsg << std::endl;
    }

    lua_close(L);

    return 0;
}
```
- `lua_dostring(lua_State*, char*)` execute a cmd conveyed via a c-string.
    - `LUA_OK` if succeed.
    - Push error message as a string on the top of the virtual stack if failed.

## Use Lua libs

For example, if I want Lua to execute `a = 7 + 11 + math.sin(23.6)`, thus I
need lua to load `math`.

- `lua_openlibs(lua_State*)` open lua standard libraries for specific lua
  virtual machine.

## Execute Lua file

In lua_cpp.lua:
```lua
a = 7 + 11 + math.sin(123)
a = a + 100
```

In lua_cpp.cpp:
```cpp
// ...
bool CheckLua(lua_State* L, int r)
{
    if (r == LUA_OK)
        return true;
    else
    {
        std::string errormsg = lua_tostring(L, -1);
        std::cerr << errormsg << std::endl;
        return false;
    }
}

int main()
{
    lua_State* L = luaL_newstate();

    luaL_openlibs(L);
    if (CheckLua(L, luaL_dofile(L, "./lua_cpp.lua")))
    {
        lua_getglobal(L, "a");
        if (lua_isnumber(L, -1))
        {
            float a_in_cpp = (float)lua_tonumber(L, -1);
            std::cout << "a in cpp: " << a_in_cpp << std::endl;
        }
    }

    lua_close(L);

    return 0;
}
```

## Use Lua script to provide datas

In settingd.lua
```lua
PlayerTitle = "Saber"
PlayerName = "Aloha"
PlayerLevel = 70
```

In lua_cpp.cpp:
```cpp
if (CheckLua(L, luaL_dofile(L, "./settings.lua")))
{
    lua_getglobal(L, "PlayerName");
    if (lua_isstring(L, -1))
    {
        player me;
        me.name = lua_tostring(L, -1);
        std::cout << me.name << std::endl;
    }
    else
    {
        std::cout << "None!" << std::endl;
    }
}
```

## Use Lua table

In setting.lua:
```lua
player =
{
    title = 'Saber',
    name = 'Aloha',
    level = 70
}
```

In lua_cpp.cpp:
```cpp
if (CheckLua(L, luaL_dofile(L, "./settings.lua")))
{
    lua_getglobal(L, "player");
    if (lua_istable(L, -1))
    {
        player me;
        lua_pushstring(L, "name");
        lua_gettable(L, -2);
        me.name =lua_tostring(L, -1);
        std::cout << me.name << std::endl;
        lua_pop(L, 1);
    }
    else
    {
        std::cout << "None!" << std::endl;
    }
}
```

1. `lua_getglobal` push the table `player` on the top of virtual stack;
2. `lua_istable` verify on the top(-1) if it's a table;
3. `lua_pushstring` push a string on the top;
4. `lua_gettable`
    1. pop the value of the top of the stack;
    2. use it to **value** in the table which is now under the top value;
    3. push the **value** on the top of the stack;
5. (optional) `lua_pop` pop the value on the top.

## Get Lua function in CPP program

In settings.lua:
```lua
players = {}

players[1] =
{
    title = 'Saber',
    name = 'Aloha',
    level = 70
}

players[2] =
{
    title = 'Archer',
    name = 'Melody',
    level = 65
}

function GetPlayer(n)
   pa * brint("[Lua] GetPlayer("..n..") called.")
   return players[n]
end
```

```cpp
if (CheckLua(L, lua_pcall(L, 1, 1, 0)))
{
    if lua_istable(L, -1)
    {
        player p2;

        lua_pushstring(L, "name");
        lua_gettable(L, -2);
        p2.name = lua_tostring(L, -1);
        lua_pop(L, 1);

        lua_pushstring(L, "title");
        lua_gettable(L, -2);
        p2.title = lua_tostring(L, -1);
        lua_pop(L, 1);

        lua_pushstring(L, "level");
        lua_gettable(L, -2);
        p2.level = lua_tonumber(L, -1);
        lua_pop(L, 1);


        std::cout << "[C++] Called Lua function GetPlayer and get: " << std::endl;
        std::cout << "\tPlayer " << p2.name << ": " << std::endl;
        std::cout << "\t\tLevel: \t" << p2.level << std::endl;
        std::cout << "\t\tTitle: \t" << p2.title << std::endl;
    }
    else
    {
        std::cout << "It's NOT a table." << std::endl;
    }
}
```

Result:
```
[Lua] GetPlayer(2.0) called.
[C++] Called Lua function GetPlayer and get:
        Player Melody:
                Level:  65
                Title:  Archer
```

## Define Lua function in CPP program and call it in Lua

In settings.lua:
```lua
function DoAThing(a, b)
    print("[Lua:Dothing] Entered function DoAThing("..a..", "..b..").")
    print("[Lua:DoAThing] Before calling HostFunction("..a.." + 10 ,"..b.." * 3)")
    c = HostFunction(a + 10, b * 3)
    print("[Lua:DoAThing] DoAThing("..a.." ,"..b..") called, got "..c)
    print("[Lua:DoAThing] "..c.." is returned by HostFunction.")
    return c
end
```

In lua_cpp.cpp:
```cpp
int lua_HostFunction(lua_State* L)
{
    // Arguments are pushed on the bottom of the stack when the function in Lua is called.
    float a = float(lua_tonumber(L, 1));
    float b = float(lua_tonumber(L, 2));
    float c = a * b;
    lua_pushnumber(L, c);
    std::cout << "[C++:lua_HostFunction] lua_HostFunction(" << a << ", " << b << ") called, pushed " << c << "." << std::endl;
    // Return the number of values been pushed on the top of the stack.
    return 1;
}
```
```cpp
lua_register(L, "HostFunction", lua_HostFunction);
std::cout << "[C++:main] C++ function lua_HostFunction is registered as Lua function HostFunction." << std::endl;
if (CheckLua(L, luaL_dofile(L, "./settings.lua")))
{
    lua_getglobal(L, "DoAThing");
    if (lua_isfunction(L, -1))
    {
        lua_pushnumber(L, 5);
        lua_pushnumber(L, 6);

        if (CheckLua(L, lua_pcall(L, 2, 1, 0)))
        {
            float result = lua_tonumber(L, -1);
            std::cout << "[C++:main] " << result << " on virtual machine is also available in Host Program." << std::endl;
        }
    }
}

```
Result:
```
[C++:main] C++ function lua_HostFunction is registered as Lua function HostFunction.
[Lua:Dothing] Entered function DoAThing(5.0, 6.0).
[Lua:DoAThing] Before calling HostFunction(5.0 + 10 ,6.0 * 3)
[C++:lua_HostFunction] lua_HostFunction(15, 18) called, pushed 270.
[Lua:DoAThing] DoAThing(5.0 ,6.0) called, got 270.0
[Lua:DoAThing] 270.0 is returned by HostFunction.
[C++:main] 270 on virtual machine is also available in Host Program.
```
