# Library related Compiler Directives

## #elsewhere

When declaring procedures, #elsewhere and #foreign no longer need to be followed by a library identifier. (This used to only work on global data declarations, but not on procedure declarations.) That means the compiler will make no assumptions about where these symbols will come from (and you can’t call them at compile time, of course). This is useful for declaring symbols that only exist at run-time (eg. WASM) or in libraries that are not available at compile-time (because you’re cross-compiling).

Made #elsewhere work on declarations, which lets you import symbols that are external to the program. Previously you could do this only on procedures with #elsewhere/#foreign, but now you can do it with arbitrary data members.

It's unclear to me how this differs from #foreign.

## #foreign

#foreign without a library identifier is mainly just for the case of 'function pointers'. You should not use it if you are really linking a known lib, because if you do that, you are re-encouraging the shitburger C behavior of silently getting a random/wrong declaration for your symbol if there is more than 1. We are trying to root out all that kind of unsoundness and not assume any of it in this system.

You can't use #foreign without a library name if the declaration is constant. The only time you see that is when something is a variable that is set at runtime by looking up a function, for example in the GL bindings. In this case #foreign just means #c_call because the #elsewhere part doesn't do anything. We should probably replace all those in the modules with just #c_call.

Those are variables, not constants. They aren't trying to link into any library, so in this case #foreign really just means #c_call and we should probably change that.

## #library

Link (either statically or dynamically?) to the library file specified e.g. #library "windows/freetype";
It's unclear how this chooses which thing to link if there are both static and dynamic libraries in the seearch path, it seems to prioritize static libraries?

## #library,link_always

The compiler now culls references to #libraries or #system_libraries when they are only referenced by #elsewhere/#c_call declarations but those declarations are not actually used. In such cases, we omit those libraries from the link line, removing dependencies and improving link speed. This fixes the annoying situation where, for example, compiling a minimal program in Windows was pulling in all kinds of Windows libraries and OpenGL.

If you don't want this behavior, you can add the suffix ",link_always" to your #library directive, as we do in modules/ImGui/module.jai (Dear Imgui needs to call clipboard functions, and those are compiled into its source code in C where the compiler has no access, so we tell it always to load user32.dll on Windows).

## #library,no_dll

#library now allows you to specify whether a DLL is unavailable: #library,no_dll "library_name".

## #library,no_static_library

If you use #elsewhere globals from a #library that does not include a static library, then you need declare the library as "#library,no_static_library" or you will get "unresolved external symbol" linker errors on Windows. (This change fixes linker warning "locally defined symbol imported" on Windows if you use #elsewhere globals coming from a static library.)

## #library,system

Removed the ,no_dll from #library,system. Moved around the Windows DLLs and static libs so they can be referenced without different names.

When searching for libaries declared as #library,system on Linux, the compiler now exclusively uses system library paths loaded from "/etc/ld.so.conf" and falls back to a default list of paths if can’t read that configuration. Previously, it always tried its builtin list of paths before trying paths from "/etc/ld.so.conf", which caused problems on some Linux variants.

On Linux, reduce installation pain (and widen the base of what we run on) by auto-searching for .so.* suffixes, when #library,system uses those suffixes. (Allows the compiler to run on Ubuntu 22.04 otherwise unmodified).

On Linux, #library,system can now specify the full filename to use, as a provisional way to deal with versioning, as in: #library,system "libatomic.so.1";

Added option "generated_library_path_prefix" to add an optional prefix to all "#library" declarations. (To correct the path to libraries if you want to put the generated bindings in a different directory than the program that generates them.)

## [deprecated] #foreign_library

Use #library instead.

## [deprecated] #foreign_system_library

Was renamed to #system_library, which is also now deprecated. Use #library,system instead.

## [deprecated] #system_library

Use #library,system instead

