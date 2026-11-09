title: FridaLab
date: 2026-11-9
category: Weekly
cover: /passagecover/35b418097280c37e567d680b2232fb51.jpg
---

## FridaLab-0x1
逐渐把安卓的学习提上日程，先一步步来吧。

首先jadx，发现代码被严重混淆了，查阅博客后发现可以按照下图的方式进行排除

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1778065978451-dbb78180-33d6-4c16-8236-ac038132ee3d.png)

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

由于这里给了目标字符串和逻辑，我们直接抄就能抄过来，不过其实这就是个位移密码(凯撒)，左移21位，可以在线工具一把梭<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1778071532057-0ce558f7-7171-43d3-bae9-5231a9062fa8.png)

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

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1778116533495-39af6c7f-fd20-4a8a-af31-65e22d9477ed.png)

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

可以看到当a=4919时app就会实现自解密。运行app回显如下<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779028627708-e3ffc898-1072-4b27-a19e-bfea83774c81.png)

可以看到app并没有给出一个可以输入的地方，那么我们就只能通过frida强制赋值然后让程序输出flag。hook脚本如下

```java
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x2.MainActivity");
    a.get_flag(4919);
})

```

输入后可以看到app回显如下

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779028808188-edf0ca48-0eb9-4eb6-b1d3-c014e2011412.png)

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

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779107947003-14b5a5d0-3992-4c55-a7b0-1e6ce0c0a488.png)

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

运行之后可以在shell中看到flag的输出<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779111475002-1167d63b-3fa1-4c4b-848c-8cf279447263.png)

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

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779269070458-853bae16-2525-4ace-9726-62b06efb2fa1.png)

## FridaLab-0x6
mainactivity如下

```java
package com.ad2001.frida0x6;

import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.SecretKeySpec;

/* loaded from: classes3.dex */
public class MainActivity extends AppCompatActivity {
    TextView t1;

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        this.t1 = (TextView) findViewById(R.id.textview);
    }

    public void get_flag(Checker A) throws BadPaddingException, NoSuchPaddingException, IllegalBlockSizeException, NoSuchAlgorithmException, InvalidKeyException {
        if (1234 == A.num1 && 4321 == A.num2) {
            Cipher cipher = Cipher.getInstance("AES");
            SecretKeySpec secretKeySpec = new SecretKeySpec("MySecureKey12345".getBytes(), "AES");
            cipher.init(2, secretKeySpec);
            byte[] decryptedBytes = Base64.getDecoder().decode("QQzMj/JNaTblEHnIzgJAQkvWJV2oK9G2/UmrCs85fog=");
            String decrypted = new String(cipher.doFinal(decryptedBytes));
            this.t1.setText(decrypted);
        }
    }
}
```

这里仍然是对mainactivity非static的函数进行hook，所以需要进行choose，hook脚本如下

```javascript
Java.perform(function () {
  Java.choose('com.ad2001.frida0x6.MainActivity',{
    onMatch: function(a){
      var b=Java.use('com.ad2001.frida0x6.Checker');
      var obj=b.$new();
      obj.num1.value=1234;
      obj.num2.value=4321;
      a.get_flag(obj);
    },
    onComplete: function() {
      console.log('Hacked');
    }
  })
});
```

可以得到flag如下

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779868403946-a757fbac-e69f-4895-a4e5-2bc8c38e54b2.png)

但需要注意的是脚本的启动时机，当我们在使用-U -f xxx -l xxx.js时，frida会默认在app启动的一瞬间就注入脚本，但那个时候mainactivity可能还没有初始化，还没被创建，所以这时我们使用Java.choose会无法选中导致直接进入onComplete并输出hacked，所以最好的注入脚本的时候就是使用-U -f xxx 当app正常启动之后再将hook脚本注入进去，这样就能正常得到flag。

## FridaLab-0x7
这题跟0x6稍有不同但总体相似，mainactivity如下

```java
package com.ad2001.frida0x7;

import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.SecretKeySpec;

/* loaded from: classes3.dex */
public class MainActivity extends AppCompatActivity {
    TextView t1;

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        this.t1 = (TextView) findViewById(R.id.textview);
        Checker ch = new Checker(123, 321);
        try {
            flag(ch);
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

    public void flag(Checker A) throws BadPaddingException, NoSuchPaddingException, IllegalBlockSizeException, NoSuchAlgorithmException, InvalidKeyException {
        if (A.num1 > 512 && 512 < A.num2) {
            Cipher cipher = Cipher.getInstance("AES");
            SecretKeySpec secretKeySpec = new SecretKeySpec("MySecureKey12345".getBytes(), "AES");
            cipher.init(2, secretKeySpec);
            byte[] decryptedBytes = Base64.getDecoder().decode("cL/bBqDmfO0IXXJCVFwYLeHp1k3mQr+SP6rlQGUPZTY=");
            String decrypted = new String(cipher.doFinal(decryptedBytes));
            this.t1.setText(decrypted);
        }
    }
}
```

try逻辑的作用为出现错误时处理错误而上面对checker赋值为123和321的主要作用就是误导，核心逻辑还是下面的flag。这里有一个点需要注意，checker逻辑如下

```java
package com.ad2001.frida0x7;

/* loaded from: classes3.dex */
public class Checker {
    int num1;
    int num2;

    Checker(int a, int b) {
        this.num1 = a;
        this.num2 = b;
    }
}
```

可以看到这里的逻辑出现了int a, int b，和0x6的checker

```java
package com.ad2001.frida0x6;

/* loaded from: classes3.dex */
public class Checker {
    int num1;
    int num2;
}
```

不一样所以我们在写脚本时也需要改变，hook脚本如下

```java
Java.perform(function () {
    Java.choose('com.ad2001.frida0x7.MainActivity',{
        onMatch: function(a){
            var b=Java.use('com.ad2001.frida0x7.Checker');
            var obj=b.$new(600,600);
            a.flag(obj);
        },
        onComplete: function() {
            console.log('Hacked');
        }
    })
});
```

可解出flag为

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779875030975-3ac08267-f592-4573-af9b-0b74bdcc5600.png)

## FridaLab-0x8
开始接触安卓nativehook。首先打开JADX

```java
package com.ad2001.frida0x8;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import com.ad2001.frida0x8.databinding.ActivityMainBinding;

/* loaded from: classes4.dex */
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;
    Button btn;
    EditText edt;

    public native int cmpstr(String str);

    static {
        System.loadLibrary("frida0x8");
    }

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding activityMainBindingInflate = ActivityMainBinding.inflate(getLayoutInflater());
        this.binding = activityMainBindingInflate;
        setContentView(activityMainBindingInflate.getRoot());
        this.edt = (EditText) findViewById(R.id.editTextText);
        Button button = (Button) findViewById(R.id.button);
        this.btn = button;
        button.setOnClickListener(new View.OnClickListener() { // from class: com.ad2001.frida0x8.MainActivity.1
            @Override // android.view.View.OnClickListener
            public void onClick(View v) {
                String ip = MainActivity.this.edt.getText().toString();
                int res = MainActivity.this.cmpstr(ip);
                if (res == 1) {
                    Toast.makeText(MainActivity.this, "YEY YOU GOT THE FLAG " + ip, 1).show();
                } else {
                    Toast.makeText(MainActivity.this, "TRY AGAIN", 1).show();
                }
            }
        });
    }
}
```

可以看到加载了frida0x8.so，解压开IDA，逻辑如下

```c
_BOOL8 __fastcall Java_com_ad2001_frida0x8_MainActivity_cmpstr(__int64 a1, __int64 a2, __int64 a3)
{
    int v4; // [rsp+30h] [rbp-C0h]
    int i; // [rsp+34h] [rbp-BCh]
    char *s1; // [rsp+40h] [rbp-B0h]
    char s2[104]; // [rsp+80h] [rbp-70h] BYREF
    unsigned __int64 v9; // [rsp+E8h] [rbp-8h]

    v9 = __readfsqword(0x28u);
    s1 = _JNIEnv::GetStringUTFChars(a1, a3, 0);
    for ( i = 0; i < __strlen_chk("GSJEB|OBUJWF`MBOE~", -1); ++i )
        s2[i] = aGsjebObujwfMbo[i] - 1;
    s2[__strlen_chk("GSJEB|OBUJWF`MBOE~", -1)] = 0;
    v4 = strcmp(s1, s2);
    __android_log_print(3, "input ", &unk_6B0, s1);
    __android_log_print(3, "Password", &unk_6B0, s2);
    _JNIEnv::ReleaseStringUTFChars(a1, a3, s1);
    return v4 == 0;
}
```

可以看到就是一个很简单的ASCII加减，但是我们要学习fridanativehook所以就不做静态的方法。

可以看到在IDA中有strcmp(s1, s2)逻辑出现，由JNI的特性可知，s1是我们在app中的输入，s2就是自解码后的flag，意味着我们要hook这个strcmp逻辑，让程序在执行strcmp前后执行我们的hook代码以达到我们的目的，我们的nativehook脚本如下

```javascript
var strcmp_addr =Module.findGlobalExportByName("strcmp");

Interceptor.attach(strcmp_addr,{

  onEnter(args)
  {
    var str1=args[0].readCString();
    var str2=args[1].readCString();

    if(str1=="Hello")
    {
      console.log("[+] Input:",str1);
      console.log("\n");
      console.log("[+] Secret:",str2);
    }
  }
});
```

hook后我们在程序中输入Hello并提交虽然程序会会先错误但是我们可以在shell中看到flag的输出

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1786353687263-b8064489-a164-454f-b9b5-49d5547e8dee.png)

详细内容查看文章FridaNativeHook

## FridaLab-0x9
一道nativehook题，java层唯一特别的地方在于告诉我们判断条件为1337，具体逻辑如下

```javascript
@Override // android.view.View.OnClickListener
            public void onClick(View v) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException {
                if (MainActivity.this.check_flag() == 1337) {
```

同时还加载了一个so文件，so文件内部逻辑很简单，就只有一个return 1

可知此时我们需要让so文件的JNI函数return 1337程序才能给我们回显flag，根据0x8所用的onEnter和onLeave，此题可用onLeave

启动fridaserver后用-n挂载上app，具体hook脚本为

```javascript
var a=Process.getModuleByName("liba0x9.so");
var addr=a.getExportByName("Java_com_ad2001_a0x9_MainActivity_check_1flag");
Interceptor.attach(addr,{
    onLeave(retval)
    {
        retval.replace(1337);
    }
});
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1793272089850-7a6c4683-4015-48d2-ae23-124640e54620.png)

点击即可获得flag

## FridaLab-0xa
至今不知道为什么我的frida在做这一类nativehook时无法用-f来spawn进行hook，只能用-n来attach进行hook。

进入java层后发现调用了so文件。JNI中就只有打印hellohackers，发现调用了一个get_flag函数

```c
unsigned __int64 __fastcall get_flag(int a1, int a2)
{
    int i; // [rsp+Ch] [rbp-44h]
    char v4[24]; // [rsp+30h] [rbp-20h] BYREF
    unsigned __int64 v5; // [rsp+48h] [rbp-8h]

    v5 = __readfsqword(0x28u);
    if ( a2 + a1 == 3 )
    {
        for ( i = 0; i < __strlen_chk("FPE>9q8A>BK-)20A-#Y", -1); ++i )
            v4[i] = 2 * i + aFpe9q8aBk20aY[i];
        v4[19] = 0;
        __android_log_print(3, "FLAG", "Decrypted Flag: %s", v4);
    }
    return __readfsqword(0x28u);
}
```

逻辑链条很简单，需要注意有两个点，第一flag是在日志中打印，第二函数的真实名称为_Z8get_flagii在写脚本时不能直接套用get_flag

可以在导出表中查询到此函数的地址，在面临此类nativehook调用函数时我们需要知道so的基地址然后通过偏移来确定函数位置从而实现调用，hook脚本如下

```javascript
var baseaddr=Process.getModuleByName("libfrida0xa.so");
var addr=baseaddr.base.add(0x206B0);
var get_flag= new NativeFunction(addr,'void',['int' , 'int']);
get_flag(1,2);
```

注入完成后可在日志中查询到flag<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1793699638966-ccb560b7-c1ae-42be-accf-72d781377d95.png)

## FridaLab-0xB
两种做法，一种可以直接在so上nop掉指令，然后UP回来，跟花指令基本一个做法，另一种就是nativehook

java层调用了so文件，进入native层分析

进入JNI发现反编译什么都没有出来，大概能判断是死代码或花指令，来看汇编

```c
mov     [rbp+var_24], 0DEADBEEFh
 cmp     [rbp+var_24], 539h
 jnz     loc_171A6
```

由于这个cmp始终不成立，所以jnz始终会执行，我们可以把jnz指令nop掉UP后就可以正常反编译伪代码进行静态逆向。

动态nativehook脚本为

```javascript
Process.attachModuleObserver({
  onAdded:function(module){
    var a=Process.getModuleByName("libfrida0xb.so");
    var addr=a.getExportByName("Java_com_ad2001_frida0xb_MainActivity_getFlag");
    var nopaddr=addr.add(0x1E);
    var write=new X86Writer(nopaddr);
    Memory.protect(nopaddr, 0x1000, 'rwx');
    try{
      write.putNop();
      write.putNop();
      write.putNop();
      write.putNop();
      write.putNop();
      write.putNop();
      write.flush();
    }
    finally{
      write.dispose()
    }
  }
})
```

点击程序中的按钮即可在日志中查看到flag<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1793701975732-58ce4804-3976-49db-bf90-53573b6d8bdb.png)

