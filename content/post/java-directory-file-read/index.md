+++

author = "H. Wang"
title = "Java目录及子目录文件读取"
date = "2023-05-05"
description = "Java目录及子目录文件读取"
tags = [
    "Java", "Files"
]
categories = [
    "Java"
]

image = ""

+++

## Java目录及子目录文件读取

在 Java 中，File 类提供了多个方法来获取目录下的文件，包括 `list()`、`listFiles()` 和 `walk()` 方法。这些方法分别有不同的用途和特点。

1. `list()`
    `list()` 方法返回一个**字符串数组**，包含该目录下所有文件和目录的名称（不包括子目录中的内容）。例如，下面的代码获取当前目录下所有文件和目录的名称：

  ```java
  File directory = new File(".");
  String[] fileList = directory.list();
  for (String fileName : fileList) {
      System.out.println(fileName);
  }
  ```

2. `listFiles()`
    `listFiles()` 方法返回一个 **File 对象数组**，包含该目录下所有文件和目录的文件对象（不包括子目录中的内容）。例如，下面的代码获取当前目录下所有文件和目录的文件对象：

  ```java
  File directory = new File(".");
  File[] fileList = directory.listFiles();
  for (File file : fileList) {
      System.out.println(file.getName());
  }
  ```

3. `walk()`
    `walk()` 方法可以递归地遍历一个目录下的所有文件和子目录，并返回一个流（Stream<File>），该流包含所有文件和目录的 File 对象。例如，下面的代码获取当前目录及其所有子目录下的所有文件和目录的文件对象：

  ```java
  Path start = Paths.get(".");
  try (Stream<Path> stream = Files.walk(start)) {
      stream.filter(Files::isRegularFile)
        .filter(path -> path.toString().endsWith(".sql"))
    		.forEach(System.out::println);
  }
  ```

方法的选择需要根据实际需求来决定。例如，如果只需要获取文件和目录的名称，可以使用 `list()` 方法；如果需要获取文件和目录的 File 对象，则可以使用 `listFiles()` 方法；如果需要递归地遍历目录下的所有文件和子目录，则可以使用 `walk()` 方法。

如下是各个方法的优缺点：

1. `list()`

   优点：

   - `list()` 方法获取目录下所有文件和目录的名称，返回结果简单，使用方便。
   - 对于大型目录，`list()` 方法通常比 `listFiles()` 方法更快，因为它只返回文件名而不是文件对象，所以不需要加载整个文件对象到内存中。

   缺点：

   - `list()` 方法无法获取文件对象，只能获取文件名称，如果需要使用文件对象，就需要进一步转换。
   - `list()` 方法无法递归遍历目录下的子目录，只能获取当前目录下的所有文件和目录的名称。

2. `listFiles()`

   优点：

   - `listFiles()` 方法获取目录下所有文件和目录的文件对象，方便进行进一步的操作。
   - 可以通过 `isDirectory()` 和 `isFile()` 方法判断文件是否是目录或文件。
   - 可以通过传入 `FilenameFilter` 或 `FileFilter` 对象进行过滤，只获取满足条件的文件对象。

   缺点：

   - 对于大型目录，`listFiles()` 方法可能会耗费大量内存，因为它返回的是文件对象而不是文件名。
   - `listFiles()` 方法无法递归遍历目录下的子目录，只能获取当前目录下的所有文件和目录的文件对象。

3. `walk()`

   优点：

   - `walk()` 方法可以递归地遍历目录下的所有文件和子目录，非常方便。
   - 返回的是一个 Stream 对象，可以轻松地对文件进行过滤、排序、分组等操作。

   缺点：

   - `walk()` 方法可能会遍历到非常深的目录层级，如果目录结构比较复杂，可能会导致遍历时间很长。
   - 返回的是 File 对象流，可能会占用大量内存。
   - `walk()` 方法只在 Java 8 及以上版本中可用，如果需要在较早版本中使用，需要使用其他方法或者自己编写递归方法。

对于目录层级适当，且需要递归读取的情况`walk()`较为合适。可使用 `Files.walk()` 方法来递归地遍历目录下的所有文件和子目录，并以文件输入流的方式读取文件内容。但需要注意的是，在处理文件内容时，需要根据实际情况进行相应的处理。例如，如果文件内容是文本文件，可以使用 `BufferedReader` 来读取文件内容，如果是二进制文件，可以使用 `InputStream` 来读取文件内容，并根据实际情况进行解析和处理。

```java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;

public class ReadFiles {
    public static void main(String[] args) throws IOException {
        String dirPath = "path/to/directory";
        try (Stream<Path> stream = Files.walk(Paths.get(dirPath))) {
            stream.filter(Files::isRegularFile)
                    .forEach(path -> {
                        try (InputStream in = Files.newInputStream(path)) {
                            // 处理文件内容
                            // 例如解析文件，或将文件内容存入数据库等操作
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    });
        }
    }
}
```

------

```java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;

public class ReadFiles {
    public static void main(String[] args) throws IOException {
        String dirPath = "path/to/directory";
        try (Stream<Path> stream = Files.walk(Paths.get(dirPath))) {
            stream.filter(Files::isRegularFile)
                    .forEach(path -> {
                        try (BufferedReader br = Files.newBufferedReader(filePath, StandardCharsets.UTF_8)) {
                            // 处理文件内容
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    });
        }
    }
}
```

