# NIO - Channel 基础使用

[toc]

## 代码

```java
package com.xzy.phase0.channel;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * @author xzy.xiao
 * @date 2022/9/21  20:39
 */
public class NioFileChannel {
    public static void main(String[] args) throws IOException {
//        writeFile();
//        readFile();
//        copyFile();
        copyFile2();
    }

    /**
     * Data --> [Buffer] --> Channel --> File
     *
     * @throws IOException -
     */
    private static void writeFile() throws IOException {
        // 创建通道
        FileOutputStream fileOutputStream = new FileOutputStream("./NioFileChannel.txt");
        FileChannel fileChannel = fileOutputStream.getChannel();

        // 创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        // 向缓冲区写入数据
        String data = "hello world!";
        byteBuffer.put(data.getBytes());

        // 从缓冲区读取数据至通道
        byteBuffer.flip();
        fileChannel.write(byteBuffer);

    }

    /**
     * File --> Channel --> [Buffer] --> Data
     *
     * @throws IOException -
     */
    private static void readFile() throws IOException {
        // 创建通道
        FileInputStream fileInputStream = new FileInputStream("./NioFileChannel.txt");
        FileChannel fileChannel = fileInputStream.getChannel();

        // 创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        // 从通道读取数据至缓冲区
        fileChannel.read(byteBuffer);

        // 从缓冲区读出数据
        String data = new String(byteBuffer.array());
        System.out.println(data);
    }

    /**
     * File1 --> Channel1 --> [Buffer] --> Channel2 --> File2
     *
     * @throws IOException -
     */
    private static void copyFile() throws IOException {
        // 创建读通道
        FileInputStream fileInputStream = new FileInputStream("./test01.txt");
        FileChannel fileReadChannel = fileInputStream.getChannel();

        // 创建写通道
        FileOutputStream fileOutputStream = new FileOutputStream("./test02.txt");
        FileChannel fileOutChannel = fileOutputStream.getChannel();

        // 创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(5); // NOTE：为了演示效果，特意设置的小一点

        // 从File1读取数据，写入缓冲区：File1 --> Channel1 --> [Buffer]
        int length;
        while ((length = fileReadChannel.read(byteBuffer)) != -1) {

            // 将buffer切换到“读”模式
            byteBuffer.flip();

            // 从缓冲区读取数据，写入File2：[Buffer] --> Channel2 --> File2
            fileOutChannel.write(byteBuffer);
            System.out.println("拷贝数据量：" + length);

            // 将buffer切换到“写”模式
            byteBuffer.clear();
        }
    }

    /**
     * File1 --> Channel1 --> Channel2 --> File2
     *
     * @throws IOException -
     */
    private static void copyFile2() throws IOException {
        // 创建读通道
        FileInputStream fileInputStream = new FileInputStream("./test01.txt");
        FileChannel fileReadChannel = fileInputStream.getChannel();

        // 创建写通道
        FileOutputStream fileOutputStream = new FileOutputStream("./test02.txt");
        FileChannel fileOutChannel = fileOutputStream.getChannel();

        // 读通道 --> 写通道
        fileOutChannel.transferFrom(fileReadChannel, 0, fileReadChannel.size());
    }
}
```

## 效果

