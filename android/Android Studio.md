# Android Studio Problem

## Problem

* Gradle构建项目失败

```tex
> Could not resolve all dependencies for configuration ':classpath'.
   > Using insecure protocols with repositories, without explicit opt-in, is     unsupported. Switch Maven repository 
'maven3(http://oss.sonatype.org/content/repositories/snapshots)' to redirect to a secure protocol (like HTTPS) or allow insecure protocols.
 See https://docs.gradle.org/7.0.2/dsl/org.gradle.api.artifacts.repositories.UrlArtifactRepository.html#org.gradle.api.artifacts.repositories.UrlArtifactRepository:allowInsecureProtocol for more details.
```

**Answer**：

For insecure HTTP connections in Gradle 7+ versions, we need to specify a boolean [allowInsecureProtocol](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.repositories.MavenArtifactRepository.html#org.gradle.api.artifacts.repositories.MavenArtifactRepository:allowInsecureProtocol) as true to `MavenArtifactRepository` closure.
Since you have received this error for `sonatype` repository, you need to set the repositories as below:

```ruby
repositories {
    //  maven { url "https://maven.fabric.io/public" }
    maven {
        url "https://jitpack.io"
    }
    maven {
        url "https://raw.github.com/Raizlabs/maven-releases/master/releases"
    }
    maven {
        url "http://oss.sonatype.org/content/repositories/snapshots"
        allowInsecureProtocol = true
    }
    maven {
        url "https://plugins.gradle.org/m2/"
    }
    maven {
        url "https://maven.google.com"
    }
    google()
    mavenCentral()
    jcenter()
}
```

> https://stackoverflow.com/questions/68585885/allow-insecure-protocols-android-gradle



* 模拟器启动失败，提示：`Error while waiting for device: The emulator process for XXX has terminated.`

查看`idea.log`可发现以下信息，但没有具体的报错信息：

> idea.log路径位于*C:\Users\Administrator\AppData\Local\Google\AndroidStudio2020.3\log\idea.log*

```tex
2022-01-12 15:01:21,105 [3649068]   INFO - manager.EmulatorProcessHandler - Emulator: G:\Android\SDK\emulator-2\emulator.exe -netdelay none -netspeed full -avd Pixel_XL_API_30 
2022-01-12 15:01:21,713 [3649676]   INFO - manager.EmulatorProcessHandler - Emulator: INFO    | Android emulator version 31.1.4.0 (build_id 7920983) (CL:N/A) 
2022-01-12 15:01:21,713 [3649676]   INFO - manager.EmulatorProcessHandler - Emulator: Process finished with exit code -1073741819 (0xC0000005) 
2022-01-12 15:01:21,713 [3649676]   WARN - manager.EmulatorProcessHandler - Emulator terminated with exit code -1073741819 
2022-01-12 15:13:43,987 [4391950]   INFO - il.indexing.FileBasedIndexImpl - START INDEX SHUTDOWN 
```

**Answer：**

复制上述报错中的启动模拟器命令，此处为：`G:\Android\SDK\emulator-2\emulator.exe -netdelay none -netspeed full -avd Pixel_XL_API_30`，然后把他放到命令行中去执行，此时会得到报错信息。我得到的报错：`cannot add library vulkan-1.dll: failed`。

> ps：如果在命令行中执行该命令提示：已经定义了HOME变量，但是在xxxx路径下找不到xxxx(翻译后的大概意思，原报错懒得重现了)。你需要定义一个环境变量（像是java_home这种），名称为：`ANDROID_AVD_HOME`，值就是你放avd文件的位置，例如： *G:\Android\User\avd*。定义好了后重启。

因此去下载相应的dll文件放入对应的位置，然后重启电脑。

注意：

1. **请注意务必备份原文件！**

2. 我是同时下载了32位和64位的dll文件并且分别放入对应位置，因此我不知道是否只需要下载某一种就能解决问题；

3. 32位dll对应位置为： *C:\Windows\SysWOW64\\*；64位：*C:\Windows\System32\\*；

   > 文件下载和位置可参见下方第三条链接

4. 将下载下来的dll文件放入上述位置即可，同时，如果文件名称不是报错提示的dll名称，比如提示上述提示`vulkan-1.dll`文件错误，但我下载下来的文件名和这个文件名不同，请改为提示的文件名。

> https://stackoverflow.com/questions/36841461/error-android-emulator-gets-killed/66337580#66337580 
>
> https://stackoverflow.com/questions/65696048/android-studio-emulator-cannot-add-library-vulkan-1-dll-failed
>
> https://www.dll-files.com/download/dd77ed1733723e46d8b4ad541c086c6d/vulkan-1.dll.html?c=bkdxR2xCc2pMcVFtQnpCT09EM1N6UT09
