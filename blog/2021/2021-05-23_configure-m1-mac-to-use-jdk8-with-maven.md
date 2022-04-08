---
title: Configure M1 Mac to use JDK8 with Maven
published: true
description: A guide on how to use Maven with JDK8 on the new ARM based M1 MacBook Pro
tags: Maven, JDK8, M1, ARM
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zk8dli502153rauzgf6i.png
---

Getting started with the new ARM based M1 Macs can be a bit awkward so here is how you get Maven working with JDK 8.

# Install Homebrew
Homebrew is what we will use to install `Maven`. You can instructions on downloading Brew here https://brew.sh/ or by simply running this script in your terminal
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

# Install Maven via Homebrew
Run the following command in your terminal
```
brew install maven
```

# Install ARM based JDK 8 from Azul
We need to get a version of JDK 8 that works on the ARM based M1 Mac, Azul provide just what we need. At the link below, enter the following if the link doesn't populate it for you.

Java Version - Java 8 (LTS)
Operating System - macOS
Architecture - ARM 64-bit
Java Package - JDK

Download the .dmg file here
https://www.azul.com/downloads/?version=java-8-lts&os=macos&architecture=arm-64-bit&package=jdk

Install the .dmg file

# Set JAVA_HOME environment variable
Now you need to set the `JAVA_HOME` environment variable to point at our new JDK 8 install. This is the environment variable that Maven looks for.
To add the environment variable, you need to add it to your zsh profile. This can be done by adding the following line to your `.zshrc` file.
The `.zshrc` file should be located in home directory.
You can navigate to it via 
```
# Navigate to directory
cd ~
# Editing the file
vim .zshrc
```

Add the following line to the file and then save
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home/jre
```

Now restart your terminal

# That's it!
When running Maven from your terminal then now it will use JDK8

Now follow me on twitter plz https://twitter.com/theshanemcgowan
