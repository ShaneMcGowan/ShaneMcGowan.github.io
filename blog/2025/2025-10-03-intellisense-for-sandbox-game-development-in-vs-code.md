---
title: How to set up intellisense in VS Code for a s&box project
published: true
description: How to set up intellisense in VS Code for a s&box project. 
tags: GAme Development, Sandbox, s&box,
cover_image: 
---

I've been having fun with the s&box game engine by Face Punch

While there have been many improvements things are still somewhat immature and there isn't much documentation on how to get things working as you want for your dev environment so I'm documenting it here.

- Install VS Code
- Install dotnet SDK 9.0
- C# Extension https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp 
This also installed the `.NET Install Tool` for me
- for me installing the `C# Dev Kit` extension was causing errors so disabling that fixed things on my end

Then in VSCode, specifically open the "code" folder, not the project folder
so for me my path is `~\repos\sandbox-farming-game\code` where `sandbox-farming-game` is my project folder

Hope this helps and it will probably break in future but good luck.