---
title: How the module system works
icon: extension
layout: default
---

## How it's loaded
How the module system works is very simple, all it does is load a compiled dexed jar using `DexClassLoader` and invoke functions on it.

https://github.com/Blokkok/blokkok-modsys/blob/main/module-system/src/main/java/com/blokkok/modsys/ModuleLoader.kt#L70-L85

## Module principle
A module is a part of the app, it can do **whatever** the app can do. This allowance can make modules to be able to extend literally anything, do anything they want, and extend features without knowing much how the module bridge API works. Though, this comes with a security risk, since modules can do whatever they wanted, they can also do malicious stuff, that's why we recommend only using open-source modules, because we can inspect and audit their code to check if there is anything malicious in it.

## Host App
> "The Blokkok host app is empty, why?"

It's because we wanted to make every features of it to be implemented by entirely modules. Not just a module, but a bunch of modules working together to make a fully functional IDE. This way, the user can mix, mash and fit their own modules into the module chain to implement their own feature!

## Libraries used
Every modules should have two libraries included in their code, `module-bridge-stub` and `module-interface`.

`module-bridge-stub` Contains empty stub APIs where the implementation lies on the `module-system`. Modules would need to implement this library as `compileOnly`, since we do not want the module to access stub API entry points.

`module-interface` Contains the `Module` abstract class where the modules need to extend to be able to be recognized and controlled correctly by the module system. This library should be implemented in the module system (I'm currently thinking if we could merge this library with `module-bridge-stub` since `module-system` can just have this implemented there rather than having two implementations of `module-interface`)

## Communications API
Since the target of this module system is so modules can work together to create something within the host app, we need a way to be able to communicate with other modules, that's where this Communication API comes in.

Communication API is an API used so modules can communicate to each other through so-called "communications". These communications are structured inside a tree called the "namespace tree". Every communications lies on a namespace where a module can define by overriding it's `namespace` property, or use the `namespace` function provided by `CommunicationContext`.

There are two communication types:
 - **Functions**

   It works as you think it would. It's a function that can be accessed by any modules or the host. There are rules for the function communication:
    - Any modules or the host can create functions assigned to a name, with the exception of having the same or different communication with same name declared before in the current namespace in any modules or host
    - Any functions' name **should be alphanumeric plus some other characters (-_+)**
    - Any modules or the host can call functions defined by any other modules or the host
    - Functions can have a key-value pair as parameters, but their types can't be defined, it's basically a map. Make sure to check their types first
    - Modules **cannot delete a function**
 - **Broadcast**

   Same as functions, it works as you think it would. Any modules or host can create and subscribe / listen to a broadcast declared by other entity. This communication method also has a few rules:
   - Any modules or host can create broadcasts assigned to a name, with the exception of having the same or different communication with same name declared before in the current namespace in Any modules or host
   - Any broadcasts' name **should be alphanumeric plus some other characters (-_+)**
   - Any modules or host can subscribe / listen to a broadcast declared by other entities
   - Any modules or host that have subscribed to a broadcast can unsubscribe it's subscription
   - Broadcasts can have a key-value pair as parameters, but their types, it's basically a map. Make sure to check their types first
   - Modules **cannot delete a broadcast**

> Important: Modules CANNOT define communications outside their own namespace. And also, the global namespace can only be defined by the app using `ModuleManager.executeCommunications`

## Flags
Flags are basically a list of strings that you can apply to your module. This flag technique is usually used to show that a module has implemented an "interface" where the modules that needed that interface can use the module to, well, do stuff.

Flags needed to be claimed using `ComContext.claimFlag(flagName: String)` by a module so it can be able to list every modules (using `ComContext.getFlagNamespaces(): List<String>`) that has that flag in their flags list.