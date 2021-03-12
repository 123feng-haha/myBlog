### maven项目打成jar包，并在无java环境下运行jar

##### 1、maven将依赖打进jar包
（1）maven先引入打包插件

```java
<build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.autoai.Main</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

（2）idea工具打jar包

File->project structure->Artifacts

![img](https://img-blog.csdnimg.cn/2019040317430951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmd5YWZlaTM2MjM5,size_16,color_FFFFFF,t_70)



![img](https://img-blog.csdnimg.cn/20190403174408524.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmd5YWZlaTM2MjM5,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190403174345973.png)



最后可以在project structure中查看jar位置

##### 2、bat文件运行jar包

首先把你的jar文件和所需文件（图片，文档，音乐等）放到一个文件夹中如：C:\Users\admin\Desktop\测试\java.jar，
然后找到一台已安装过java环境的电脑，找到jdk安装目录下的jre文件夹，整个复制到C:\Users\admin\Desktop\测试\下，
最后创建一个bat文件，内容为：start jre\bin\javaw -jar java.jar，其中，jre\bin\javaw为javaw对应路径，按实际上的写，一般都是这个路径，-jar为命令，java.jar为要运行的jar文件。


3.带入参数

项目中main方法接受外部参数

System.getProperty("参数")
在bat文件中

start jre\bin\javaw -D参数1=参数1 -D参数2=参数2 -jar test.jar
