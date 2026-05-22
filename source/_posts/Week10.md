title: Week10
date: 2026-5-21
category: Weekly
cover: /source/passagecover/11.jpg
---

## FridaLab-0x1
首先jadx，发现代码被严重混淆了，查阅博客后发现可以按照下图的方式进行排除

![](/blog_essay_picture/Week10/1.png)

勾选后混乱代码的可读性大大增强。

然后就有两个逆向点(后面看了官方solution后发现有三个)，一个是静态逆向

```java
 void check(int i, int i2) {
        if ((i * 2) + 4 == i2) {
            Toast.makeText(getApplicationContext(), "Yey you guessed it right", 1).show();
            StringBuilder sb = new StringBuilder();
            for (int i3 = 0; i3 < 20; i3++) {
                char cCharAt = "AMDYV{WVWT_CJJF_0s1}".charAt(i3);
                if (cCharAt >= 'a' && cCharAt <= 'z') {
                    cCharAt = (char) (cCharAt - 21);
                    if (cCharAt < 'a') {
                    }
                } else if (cCharAt >= 'A' && cCharAt <= 'Z' && (cCharAt = (char) (cCharAt - 21)) < 'A') {
                    cCharAt = (char) (cCharAt + 26);
                }
                sb.append(cCharAt);
            }
            this.t1.setText(sb.toString());
            return;
        }
        Toast.makeText(getApplicationContext(), "Try again", 1).show();
    }

```

由于这里给了目标字符串和逻辑，我们直接抄就能抄过来，不过其实这就是个位移密码(凯撒)，左移21位，可以在线工具一把梭
![](/blog_essay_picture/Week10/2.png)

但是这样就少了很多趣味，并且学不到什么frida的知识，所以我们重点看fridahook

进入mainactivity，发现如下

```java
  final int i = get_random()
 int get_random() {
        return new Random().nextInt(100);
    }
   if ((i * 2) + 4 == i2) {
            Toast.makeText(getApplicationContext(), "Yey you guessed it right", 1).show();
```

调用了random函数，意味着我们可以hook这个点。官方solution中的解法在我frida17.6.2中并不适用，没办法用spawn来hook，需要attach，脚本如下

```java
Java.perform(function() {
    var MainActivity = Java.use("com.ad2001.frida0x1.MainActivity");
    MainActivity.get_random.implementation = function() {
        console.log("attached");
        return 5;
    };
});
```

在powershell中输入。这里有一个细节<font style="color:#DF2A3F;">我们使用的是implementation，这个语句的使用范围是当我们需要替换掉某个函数，将某个函数的参数强行赋值，或者说更改某个函数的行为时我们才会用这个语句。</font>

```powershell
frida -U -n "Frida 0x1" -l "hook.js"
```

由于我们是attach上的，所以我们需要让程序刷新一次，经搜索得知，当我们旋转虚拟机的屏幕后，虚拟机会销毁当前activity并重启，这样我们就能强行attach上了。又由源码中的计算公式得知，当我们return 5时，我们需要输入14，输入后得到flag

![](/blog_essay_picture/Week10/3.png)

## FridaLab-0x2
逻辑清晰的题，主逻辑如下

```java
public class MainActivity extends AppCompatActivity {
    static TextView t1;

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        t1 = (TextView) findViewById(R.id.textview);
    }

    public static void get_flag(int a) throws BadPaddingException, NoSuchPaddingException, IllegalBlockSizeException, NoSuchAlgorithmException, InvalidKeyException, InvalidAlgorithmParameterException {
        if (a == 4919) {
            try {
                SecretKeySpec secretKeySpec = new SecretKeySpec("HILLBILLWILLBINN".getBytes(), "AES");
                Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
                IvParameterSpec iv = new IvParameterSpec(new byte[16]);
                cipher.init(2, secretKeySpec, iv);
                byte[] decryptedBytes = cipher.doFinal(Base64.decode("q7mBQegjhpfIAr0OgfLvH0t/D0Xi0ieG0vd+8ZVW+b4=", 0));
                String decryptedText = new String(decryptedBytes);
                t1.setText(decryptedText);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

可以看到当a=4919时app就会实现自解密。运行app回显如下
![](/blog_essay_picture/Week10/4.png)

可以看到app并没有给出一个可以输入的地方，那么我们就只能通过frida强制赋值然后让程序输出flag。hook脚本如下

```java
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x2.MainActivity");
    a.get_flag(4919);
})

```

输入后可以看到app回显如下

![](/blog_essay_picture/Week10/5.png)

便可以得到flag了

## FridaLab-0x3
这道题跟0x2的考察内容差不多，main逻辑如下

```java
/* loaded from: classes3.dex */
public class MainActivity extends AppCompatActivity {
    TextView t1;

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn = (Button) findViewById(R.id.button);
        this.t1 = (TextView) findViewById(R.id.textView);
        btn.setOnClickListener(new View.OnClickListener() { // from class: com.ad2001.frida0x3.MainActivity.1
            @Override // android.view.View.OnClickListener
            public void onClick(View v) throws BadPaddingException, NoSuchPaddingException, IllegalBlockSizeException, NoSuchAlgorithmException, InvalidKeyException {
                if (Checker.code == 512) {
                    byte[] bArr = new byte[0];
                    Toast.makeText(MainActivity.this.getApplicationContext(), "YOU WON!!!", 1).show();
                    byte[] KeyData = "glass123".getBytes();
                    SecretKeySpec KS = new SecretKeySpec(KeyData, "Blowfish");
                    byte[] ecryptedtexttobytes = Base64.getDecoder().decode("MKxsZsY9Usw3ozXKKzTF0ymIaC8rs0AY74GnaKqkUrk=");
                    try {
                        Cipher cipher = Cipher.getInstance("Blowfish");
                        cipher.init(2, KS);
                        byte[] decrypted = cipher.doFinal(ecryptedtexttobytes);
                        String decryptedString = new String(decrypted, Charset.forName("UTF-8"));
                        MainActivity.this.t1.setText(decryptedString);
                        return;
                    } catch (InvalidKeyException e) {
                        throw new RuntimeException(e);
                    } catch (NoSuchAlgorithmException e2) {
                        throw new RuntimeException(e2);
                    } catch (BadPaddingException e3) {
                        throw new RuntimeException(e3);
                    } catch (IllegalBlockSizeException e4) {
                        throw new RuntimeException(e4);
                    } catch (NoSuchPaddingException e5) {
                        throw new RuntimeException(e5);
                    }
                }
                Toast.makeText(MainActivity.this.getApplicationContext(), "TRY AGAIN", 1).show();
            }
        });
    }
}
```

可以看到当Checker.code=512时app就会进行自解密回显flag，追踪Checker类，其逻辑如下

```java
package com.ad2001.frida0x3;

/* loaded from: classes3.dex */
public class Checker {
    static int code = 0;

    public static void increase() {
        code += 2;
    }
}
```

可以看到这里主要是对code变量进行自加2，相当于如果我们点击的话要点击256次才能唤出flag，显然不可信。这里有两种解法，一种是hookcode这个变量，强行赋值为512；第二种是直接调用checker函数256次。

第一种的hook脚本如下

```java
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x3.Checker");
    a.code.value=512;
})
```

这里有对比0x2有一个小细节，以防忘记我把脚本再拉出来一次，以下是0x2的hook脚本

```java
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x2.MainActivity");
    a.get_flag(4919);
})
```

这里的细节在于赋值，0x3的赋值是直接利用等于号，而0x2是括号括起来的，为什么呢？

看0x3中的checker逻辑，对code的定义是int code，那么这就是一个变量，我们对变量赋值自然只能用=；但是在0x2中也有用int来定义值(以防忘了截取部分如下)

```java
    public static void get_flag(int a) throws BadPaddingException, NoSuchPaddingException, IllegalBlockSizeException, NoSuchAlgorithmException, InvalidKeyException, InvalidAlgorithmParameterException {
        if (a == 4919) {
            try {
                SecretKeySpec secretKeySpec = new SecretKeySpec("HILLBILLWILLBINN".getBytes(), "AES");
                Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
                IvParameterSpec iv = new IvParameterSpec(new byte[16]);
                cipher.init(2, secretKeySpec, iv);
                byte[] decryptedBytes = cipher.doFinal(Base64.decode("q7mBQegjhpfIAr0OgfLvH0t/D0Xi0ieG0vd+8ZVW+b4=", 0));
                String decryptedText = new String(decryptedBytes);
                t1.setText(decryptedText);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
```

这里也是int a，但是区别在于get_flag函数包括了a，所以a是get_flag的一个参数，当我们强行赋值的时候相当于调用函数并为函数内的参数赋值为4919所以才需要括号包裹。

解出flag如下

![](/blog_essay_picture/Week10/6.png)

第二种解法的hook脚本如下

```java
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x3.Checker");
    for(var i=0;i<256;i++)
    {
        a.increase();
    }
})
```

就是直接调用increase函数256次，然后点一下按钮就能出flag了，不多赘述。

## FridaLab-0x4
学习了前面几道题这题就显得相对简单main逻辑没什么好看的，就是在打印，来看看check逻辑

```java
package com.ad2001.frida0x4;

/* loaded from: classes3.dex */
public class Check {
    public String get_flag(int a) {
        if (a == 1337) {
            byte[] decoded = new byte["I]FKNtW@]JKPFA\\[NALJr".getBytes().length];
            for (int i = 0; i < "I]FKNtW@]JKPFA\\[NALJr".getBytes().length; i++) {
                decoded[i] = (byte) ("I]FKNtW@]JKPFA\\[NALJr".getBytes()[i] ^ 15);
            }
            return new String(decoded);
        }
        return "";
    }
}
```

虽然静态也能做但还是要学学fridahook。可以看到这里就是要去hook一个函数的参数让系统进行异或自解密，但是值得注意的是这里的定义是public String get_flag(int a)，而不是public static get_flag(int a)，所以我们hook的时候要创造一个实例，那么hook脚本如下

```java
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x4.Check");
    var obj=a.$new();
    obj.get_flag(1337);
    console.log("Hacked");
    console.log(obj.get_flag(1337));
});
```

运行之后可以在shell中看到flag的输出
![](/blog_essay_picture/Week10/7.png)

## FridaLab-0x5
相比前面几道题这道题考法更新颖，考察了直接在mainactivity中更改appUI让flag显示在屏幕上

main逻辑如下

```java
public class MainActivity extends AppCompatActivity {
    TextView t1;

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        this.t1 = (TextView) findViewById(R.id.textview);
    }

    public void flag(int code) throws BadPaddingException, NoSuchPaddingException, IllegalBlockSizeException, NoSuchAlgorithmException, InvalidKeyException, InvalidAlgorithmParameterException {
        if (code == 1337) {
            try {
                SecretKeySpec secretKeySpec = new SecretKeySpec("WILLIWOMNKESAWEL".getBytes(), "AES");
                Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
                IvParameterSpec iv = new IvParameterSpec(new byte[16]);
                cipher.init(2, secretKeySpec, iv);
                byte[] decodedEnc = Base64.getDecoder().decode("2Y2YINP9PtJCS/7oq189VzFynmpG8swQDmH4IC9wKAY=");
                byte[] decryptedBytes = cipher.doFinal(decodedEnc);
                String decryptedText = new String(decryptedBytes);
                this.t1.setText(decryptedText);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

可以看到这里也是非static的函数赋值hook，按照之前的思路去hook发现frida报错无法正常hook，查询solution后得知由于我们是在mainactivity中进行hook，所以我们不能像之前一样创造一个示例然后去修改实例，所以我们利用以下代码进行mainactivity的hook

```java
Java.perform(function () {
    var MainActivity = Java.use("com.ad2001.frida0x5.MainActivity");
    Java.choose("com.ad2001.frida0x5.MainActivity", {
        onMatch: function(a) {
            a.flag(1337);
        },
        onComplete: function() {
            console.log("Hacked");
        }
    });
});
```

逐一解释：

由于mainactivity有自己的生命周期，所以我们没办法进行new的操作，否则就只是new了一个空壳没有任何意义，所以我们直接利用Java.choose来扫描目标进程中的实例，当我们找到目标实例时就启动onMatch内的代码将目标进程中的代码替换成我们的代码，由于main逻辑中已经有setText了，所以我们不需要再console.log来输出。

其中Java.choose的工作逻辑就是遍历JVM堆找到所有mainactiviy的类，每找到一个就调用一次onMatch逻辑替换成其内的逻辑。onComplete就是当choose全部完成之后才会调用的，显示已完成的一个函数，完全可以省略。

hook后appUI回显如下

![](/blog_essay_picture/Week10/8.png)

