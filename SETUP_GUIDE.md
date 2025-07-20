# EternalClient Script Setup Guide

## Project Setup

### 1. Create New Project in IntelliJ IDEA
- Click "New Project" on welcome screen
- Configure:
  - **Name and Location**: Choose project name and location
  - **Language**: Java
  - **Build System**: Gradle
  - **SDK**: Java JDK (e.g. 11, 11.0.19)
  - **Gradle DSL**: Kotlin

### 2. Create Main Script File
Create `Main.java` in `src/main/java` directory:

```java
import net.eternalclient.api.script.AbstractScript;
import net.eternalclient.api.script.ScriptCategory;
import net.eternalclient.api.script.ScriptManifest;

@ScriptManifest(
    name = "EternalClientScript",
    description = "A script for EternalClient.",
    author = "EternalClient",
    category = ScriptCategory.OTHER,
    version = 1.0
)
public class Main extends AbstractScript
{
    @Override
    public void onStart(String[] strings)
    {
        // This method is called when the script is started with the script paramters.
    }

    @Override
    public void onExit()
    {
        // This method is called when the script is stopped.
    }

    @Override
    public void onPause()
    {
        // This method is called when the script is paused.
    }

    @Override
    public void onResume()
    {
        // This method is called when the script is resumed.
    }

    @Override
    public int onLoop()
    {
        // The returned value is the duration in milliseconds that it will sleep before calling onLoop again.
        return 0;
    }
}
```

### 3. Configure build.gradle.kts
Replace contents of `build.gradle.kts`:

```kotlin
val userHomeDir: String = System.getProperty("user.home");

plugins {
    java
}

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    // Adding the EternalClient API as a compileOnly dependency.
    compileOnly(files("${userHomeDir}/path/to/EternalClient/Data/EternalClient.jar"))
}

// Changing this will make it so that the jar file is named differently
val scriptFileName: String = "EternalClientScript"

tasks {
    jar {
        archiveFileName.set("${scriptFileName}.jar")

        // Destination directory for the jar file, EternalClients local script directory.
        destinationDirectory.set(file("$userHomeDir/path/to/EternalClient/Scripts"))
    }
}
```

### 4. Build Script
Run the `jar` task in Gradle tool window to build and place script in EternalClient script directory.

## Notes
- The `scriptFileName` variable can be changed to rename the output JAR file
- Script will be automatically placed in `~/path/to/EternalClient/Scripts/` when built
- Import statements are required: `net.eternalclient.api.script.*`