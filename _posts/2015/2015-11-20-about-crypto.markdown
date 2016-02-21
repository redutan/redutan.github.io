---
layout: post
title: 상호 간 암호화 스팩 공유에 관하여
date: '2015-11-20 01:20'
tags:
  - blahblah
---

최근에 타 업체와 같이 암호화에 관해서 협의한 적이 있었습니다.

다들 어떻게 하는지는 잘 모르겠지만, 일반적으로 아래와 같이 의견교환을 할 것입니다.

**암호화 스팩**

- 알고리즘 : **AES-256**
- 암호화키 : `abcdefghijklmnopqrstuvwxyz123457890`
- 인코딩 : `UTF-8`

알고리즘, 암호화키 만 보내는 경우가 일반적이며, 인코딩을 보내지 않는 경우도 심심찮게 확인할 수 있습니다.
그리고 **샘플코드** 라도 첨부파일로 보내주면 다행인데, 그렇지 않은 경우도 허다합니다.
*일부 경우에는 검증되지 않은 블로그 포스트나 링크를 보내주고 그대로 해달라고 하기도 합니다.*

참으로 답답한 현실이 아닐수가 없습니다. 저도 과거 주니어 시절에는 잘 모르고 위 처럼 의사소통을 하기도 하였는데,
지금 생각하면 얼굴이 붉어지는 일입니다 ㅠ

많은 개발자 들이 암호화에 대한 정확한 이해가 없이 의사소통을 하고 이런 방식이 어느 순간부터 대중화(?) 된 것 같습니다.
여기에 관련해서 해당 부분에 대한 간략한 이론적인 내용을 알아보고, 이제부터는 어떻게 암호화 스팩을 상호 간 공유해야 하는지 알아보겠습니다.

> 짚고 넘어가야할 부분은 여기서는 대중적으로 상호 간 통신 시 가장 많이 쓰이는 **대칭키 암호화** 방식으로 이야기 하겠습니다.
> 코드는 Java 기준으로 설명하겠습니다.

# 암호화 인터페이스

{% highlight java %}
public class Crypto {
  // 암호화
  String encrypt(String plain);
  // 복호화
  String decrypt(String cipher);
}
{% endhighlight %}

일반적으로 대부분의 개발자들은 개발 시 암/복호화를 위해서 위와 같은 인터페이스가 필요할 것입니다.

*반론으로 왜 `byte[]` 가 지원되지 않는가?, 왜 암호화 키를 입력 받지 않는가도 있지만, 그것은 생각하지 않겠습니다.*

위와 같은 간단한 인터페이스를 통해서 간단하게 암호화가 가능하게 할려면 최소 아래와 같은 내용을 확인해야합니다.

## 대칭키 암호화 시 스팩

- 암호화 알고리즘
  - key size
  - 필요에 따라서 block size
- 암호화 모드
- Padding 방식
- 암호화 키
  - 일부 암호화 모드에서는 초기화벡터 도 필요함
- 암호문자열 바이트인코딩 + 문자인코딩
- 키문자열(암호화키, 초기화벡터) - 바이트 간 인코딩 방식 (문자인코딩, 바이트인코딩 둘 다 가능)
  - *암호 문자열과 인코딩 방식과 키문자열 인코딩 방식이 다르면 필요*

말로 풀어서 쓰니 어려우니 예를 들어서 위 내용을 풀어보겠습니다.

- 암호화 알고리즘 : **AES**
  - key size : **256 bit**
  - block size : **128 bit** (AES의 블록사이즈는 128로 고정입니다. 고로 block size는 skip해도 됩니다.)
- 암호화 모드 : **CBC**
- Padding 방식 : **PKCS5**
- 암호화키 : `12345678901234567890123456789012` (32자)
  - 초기화벡터 : `1234567890123456` (16자)
- 암호문 인코딩 방식 : **Base64 + UTF-8**
- 키 인코딩 방식 : **ASCII** (문자인코딩)

이렇게 보아도 이해가 잘 안되는 거 같습니다.
그래서 위에서 선언한 인터페이스를 구현하면서 어떻게 해당 내용이 구현되는지 보면서 알아보겠습니다.

## 암호화 인터페이스 구현

#### 암호화 알고리즘, 모드, 패딩
{% highlight java %}
public class AES256Crypto implements Crypto {

  // 알고리즘/모드/패딩
  private static final String algorithm = "AES/CBC/PKCS5Padding";

  // 암호화
  public String encrypt(String plain) {
    Cipher c = Cipher.getInstance(algorithm);
    // TODO
    return null;
  }
  ...
}
{% endhighlight %}

기본적으로 알고리즘, 모드, 패딩이 필요합니다.

- 알고리즘은 암호화 알고리즘이며 예제의 경우 `AES` 방식을 사용합니다. 블록암호화 방식이기 때문에 Byte Padding이 필요합니다.
- 모드는 암호화 방식입니다. 예제의 경우 `CBC`를 사용하는데 이럴 경우 초기화 벡터가 필요합니다.
- 블록 암호화 방식이기 때문에 padding 방식이 필요합니다.

예제의 경우 우선 암호화(`encrypt`) 부터 구현하겠습니다.

> 참고 : [블럭암호 AES][2b86750a]
> [Padding와 암호화 모드][a55c5334]  

#### 암호화 키, key size
{% highlight java %}
public class AES256Crypto implements Crypto {
  // 알고리즘/모드/패딩
  private static final String algorithm = "AES/CBC/PKCS5Padding";
  // 암호화 키
  private final String secretKey;

  public AESCrypto(String secretKey) {
    if (secretKey.length != 256 / 8) {
      throw new IllegalArgumentException("'secretKey' must be 256 bit");
    }
    this.secretKey = secretKey;
  }

  // 암호화
  public String encrypt(String plain) {
    Cipher c = Cipher.getInstance(algorithm);
    // TODO
    return null;
  }
  ...
}
{% endhighlight %}

256 bit 암호화 방식이기 때문에 키 입력에 대한 유효성을 추가하였습니다. - 아직은 논란이 있는 코드입니다.
key size가 암호화 알고리즘의 bit 수를 가르치는 것과 동일하게 됩니다.
**즉 AES256 이라는 것은 암호화 키 사이즈가 256 bit 라는 말과 동일합니다.**



#### 초기화 벡터, block size
{% highlight java %}
public class AES256Crypto implements Crypto {
  // 알고리즘/모드/패딩
  private static final String algorithm = "AES/CBC/PKCS5Padding";
  // 암호화 키
  private final String secretKey;
  // 초기화 벡터
  private final String iv;

  public AESCrypto(String secretKey, String iv) {
    if (secretKey.length != 256 / 8) {
      throw new IllegalArgumentException("'secretKey' must be 256 bit");
    }
    if (iv.length != 128 / 8) {
      throw new IllegalArgumentException("'iv' must be 128 bit");
    }
    this.secretKey = secretKey;
    this.iv = iv;
  }

  // 암호화
  public String encrypt(String plain) {
    Cipher c = Cipher.getInstance(algorithm);
    // TODO
    return null;
  }
  ...
}
{% endhighlight %}

`AES`의 block size는 128 bit 고정이기 때문에 별 다른 변화가 없습니다.

`AES`의 모태가 되는 `Rijendael`알고리즘 경우에는 128 192 256 bit이기 때문에 달라질 수 있습니다.

단, 초기화벡터(iv)의 경우 block size와 같아야 하기 때문에 128 bit 여야합니다.
iv의 경우 더 강력한 암호화를 위해서는 암호화 요청 시 마다 달라지는 것이 보안에 더 좋으나,
예제이므로 우선은 초기 세팅으로 표현하겠습니다.


#### 암호문 인코딩
{% highlight java %}
public interface Encoder {
  // string -> bytes
  byte[] encode(String str);
  // bytes -> string
  String decode(byte[] bytes);
}
{% endhighlight %}

갑자기 새로운 인터페이스가 추가되었습니다.
암호문을 인코딩 하기 위해서는 위와 같은 인터페이스가 필요하기 때문입니다.

간단하게 설명하면 string -> bytes 가 `encode`이며, bytes -> string 는 `decode`입니다.
기본적으로 인코딩은 charset와 연관관계가 깊은 문자인코딩으로 생각하기 쉬운데, 암호화의 경우에는 암호화로 인해
문자인코딩과는 별개의 규칙이 없는 bytes가 반환되므로 문자로 표현할 수 없게 됩니다.

고로 **암호문을 위한 인코딩** 은 1차적으로 문자(charset)와 연관없는 인코딩이 가능해야합니다.
즉 문자열을 기준으로 해서 byte화 시키는 문자인코딩과는 다르게 **bytes를 기준으로 문자열화 시키는
인코딩 방식이 필요** 하게 됩니다.

가장 간단하게는 bytes를 16진수 기반으로 표현하는 `Hex` 방식이나 `Base64` 방식을 사용하는 것이 좋습니다.

예제는 `Base64` 방식으로 진행하겠습니다.

#### 암호문 Base 64 인코딩 구현
{% highlight java %}
public class Base64Encoder implements Encoder {
  // base64는 아스키 코드 내로 표현가능한 인코딩 방식
  private static final String ASCII = "US-ASCII";

  // string -> bytes
  public byte[] encode(String str) {
    return Base64.encodeBase64(str.getBytes(ASCII));
  }
  // bytes -> string
  public String decode(byte[] bytes) {
    return new String(Base64.decodeBase64(bytes), ASCII);
  }
}
{% endhighlight %}

편의상 `Exception` 핸들링은 제외하였습니다.

위 구현체를 바탕으로 `AES256Crypto` 를 계속 구현해 보겠습니다.
아까 예시에서 정의한 "암호문 인코딩 방식 : **Base64 + UTF-8**" 에서 **UTF-8** 은 이후에 나옵니다.
`Base64Encoder`에 정의된 것과 혼동하면 안됩니다. : base64 스팩 자체가 ASCII에 의존하는 방식입니다.

> 참고 : [위키백과-base64][a1942bdd]

#### 암호문 구현체에 암호문 인코더 주입
{% highlight java %}
public class AES256Crypto implements Crypto {
  // 알고리즘/모드/패딩
  private static final String algorithm = "AES/CBC/PKCS5Padding";
  // 암호화 키
  private final String secretKey;
  // 초기화 벡터
  private final String iv;
  // 문자인코딩 방식
  private final String charset = "UTF-8";
  // 암호문 바이트 인코더
  private Encoder encoder = new Base64Encoder();
  ...

  // 암호화
  public String encrypt(String plain) {
    // 암호화 키 생성
    byte[] keyData = secretKey.getBytes("US-ASCII");
    SecretKey secureKey = new SecretKeySpec(keyData, "AES");

    Cipher c = Cipher.getInstance(algorithm);
    // 암호화 키 주입, iv 생성 주입, 초기화 - 이상하지만 우선 무시
    c.init(Cipher.ENCRYPT_MODE, secureKey, new IvParameterSpec(iv.getBytes("US-ASCII")));
    // 문자인코딩 방식을 통한 string -> byte 변환 후 암호화
    byte[] encrypted = c.doFinal(plain.getBytes(charset));
    // encoder#encode를 통한 byte -> string 변환
    return encoder.encode(encrypted);
  }
  ...
}
{% endhighlight %}

객체 변수로 `charset` 항목이 `UTF-8`로 추가되었습니다.
그리고 `encoder` 객체도 default로 생성된 상태입니다.

**입력받은 plain은 문자열 이기 때문에 문자인코딩 방식을 통해서 byte로 변환하겠습니다.**

{% highlight java %}
// 문자인코딩 방식을 통한 string -> byte 변환
plain.getBytes(charset)
{% endhighlight %}
Java에서는 기본적으로 `String#getBytes(String charset)` 메소드가 있기 때문에 이것을 이용해서 bytes로 변환하였습니다.

**암호화 한 후 bytes 로 반환된 값을 문자열로 출력해야하는데 이럴 경우에는 바이트 인코딩 방식이 필요합니다.**
위에서 미리 선언해 둔 Encoder인터페이스를 통해서 변환하겠습니다.

{% highlight java %}
// encoder.encode를 통한 byte -> string 변환
encoder.encode(encrypted);
{% endhighlight %}

**이렇게 해서 1차적으로 암호화 부분 구현이 완료되었습니다.**

*키 관련된 부분은 우선 무시하겠습니다. 그냥 봐도 좀 이상하지만 다음에 수정하겠습니다.*

#### 키 인코더 구현
{% highlight java %}
public class StringEncoder implements Encoder {
  // 문자인코딩
  private final String charset;

  public StringEncoder(String charset) {
    this.charset = charset;
  }
  // string -> bytes
  public byte[] encode(String str) {
    return str.getByte(charset);
  }
  // bytes -> string
  public String decode(byte[] bytes) {
    return new String(bytes, charset);
  }
}
{% endhighlight %}

갑자기 매우 단순한 키 인코더 구현체가 나와서 당황스러울 듯 합니다.
하지만 키 인코딩의 경우 문자인코딩, 바이트인코딩 방식 둘 다 가능하기 때문에 위와 같이
`Encoder` 인터페이스를 통한 구현체를 제공하는 것이 다형성이 도움이 되기 때문에 이렇게 구성해보았습니다.

지금의 예제는 `ASCII` 방식으로 암호화 키와 초기화 벡터를 제공하지만 이런 경우에는 키 bytes 구성이 아스키 기반으로 인해서 단순해 집니다.
고로 가능하면 다양한 **바이트 구성이 가능한 `base64`나 `hex`를 이용해서 키 bytes를 제공할 수 있게 하는 것이 보안에 유리합니다.**

우선 예제를 위해서 위 `StringEncoder`를 사용하는 것으로 진행하겠습니다.

*중요한 점은 위 인코더는 문자인코딩 방식이기 때문에 암호문 인코더로 사용하면 정상적인 암/복호화가 불가능 하게 됩니다.**

#### 암호문 구현체에 키 인코더 주입 후 변경
{% highlight java %}
public class AES256Crypto implements Crypto {
  // 알고리즘/모드/패딩
  private static final String algorithm = "AES/CBC/PKCS5Padding";
  // 암호화 키
  private final String secretKey;
  // 초기화 벡터
  private final String iv;
  // 문자인코딩 방식
  private final String charset = "UTF-8";
  // 암호문 바이트 인코더
  private Encoder encoder = new Base64Encoder();
  // 키 인코더
  private Encoder keyEncoder = new StringEncoder("US-ASCII");
  ...

  // 암호화
  public String encrypt(String plain) {
    // 암호화 키 생성 - keyEncoder를 이용
    byte[] keyData = keyEncoer(secretKey);
    SecretKey secureKey = new SecretKeySpec(keyData, "AES");

    Cipher c = Cipher.getInstance(algorithm);
    // 암호화 키 주입, iv 생성 주입, 초기화 - iv의 경우 keyEncodr를 이용
    c.init(Cipher.ENCRYPT_MODE, secureKey, new IvParameterSpec(keyEncoer(iv));
    byte[] encrypted = c.doFinal(plain.getBytes(charset));
    return encoder.encode(encrypted);
  }
  ...
}
{% endhighlight %}

`keyEncoder`를 이용해서 암호화 키와 초기화 벡터를 bytes로 성공적으로 변환하였습니다.
여기에서 `keyEncoder`의 경우 `ASCII` charset을 기반으로 생성하였는데, 대부분의 경우 암호화 키를
예제(`12345678901234567890123456789012`)와 같이 영어와 숫자 또는 특수문자 기반으로 문자열을 제공하기 때문에 `ASCII` 만으로도 충분합니다.

만약 한국어 등을 이용한다면 해당 문자를 표현할 수 있는 charset(ex:`UTF-8`, `EUC-KR`)로 변환해야 합니다만 그런 케이스는 거의 없을 것입니다.

**위에도 설명했지만 가능하면 문자열 키도 base64 과 같은 바이트 인코딩 방식으로 제공하는 것이 보안에 좋습니다.**

#### 복호화 메소드 구현

{% highlight java %}
public class AES256Crypto implements Crypto {
  // 알고리즘/모드/패딩
  private static final String algorithm = "AES/CBC/PKCS5Padding";
  // 암호화 키
  private final String secretKey;
  // 초기화 벡터
  private final String iv;
  // 문자인코딩 방식
  private final String charset = "UTF-8";
  // 암호문 인코더
  private Encoder encoder = new Base64Encoder();
  // 키 인코더
  private Encoder keyEncoder = new StringEncoder("US-ASCII");
  ...

  // 복호화
  public String decrypt(String cipher) {
    // 암호화 키 생성 - keyEncoder를 이용
    byte[] keyData = keyEncoer(secretKey);
    SecretKey secureKey = new SecretKeySpec(keyData, "AES");

    Cipher c = Cipher.getInstance(algorithm);
    c.init(Cipher.DECRYPT_MODE, secureKey, new IvParameterSpec(keyEncoer(iv));
    // encoder.decode 를 통해서 string -> bytes 변환
    byte[] encrypted = encoder.decode(cipher);
    // 복호화 후 문자인코딩 방식을 통한 byte -> string 변환 후 반환
    return new String(c.doFinal(encrypted), charset);
  }
}
{% endhighlight %}

암호화와는 반대로 진행하는 것을 확인할 수 있습니다.


**암호화**

1. `String#getBytes(String)`(문자 인코딩)를 이용해서 bytes로 변환
1. 암호화
1. `encoder.encode`(바이트 인코딩)를 이용해서 String으로 변환

**복호화**

1. `encoder.decode`(바이트 디코딩)를 통해서 bytes로 변환
1. 복호화
1. `new String(byte[], String)`(문자 디코딩)을 이용해서 String으로 변환

![암/복호화 흐름](/images/2015/11/crypto-flow.png)

*구현체에 중복코드가 많아서 약간의 리펙토링을 진행하고 전체코드를 보겠습니다.*

#### AES256Crypt 리펙토링
{% highlight java %}
public class AES256Crypto implements Crypto {
  public static final int KEY_SIZE = 256;
  public static final int BLOCK_SIZE = 128;
  private static final String AES = "AES";
  // 알고리즘/모드/패딩
  private static final String algorithm = AES + "/CBC/PKCS5Padding";
  // 암호화 키
  private final SecretKey secretKey;
  // 초기화 벡터
  private final IvParameterSpec iv;
  // 문자인코딩 방식
  private final String charset = "UTF-8";
  // 암호문 인코더
  private Encoder encoder = new Base64Encoder();
  // 키 인코더
  private Encoder keyEncoder = new StringEncoder("US-ASCII");

  public AESCrypto(String secretKey, String iv) {
    this.setSecretKey(secretKey);
    this.setIv(iv);
  }

  private void setSecretKey(String secretKey) {
    byte[] keyBytes = keyEncoder.encode(secretKey);
    if (keyBytes.length != KEY_SIZE / 8) {
      throw new IllegalArgumentException("'secretKey' must be "+ KEY_SIZE +" bit");
    }
    this.secretKey = new SecretKeySpec(keyBytes, AES);
  }

  private void setIv(String iv) {
    byte[] ivBytes = keyEncoder.encode(iv);
    if (ivBytes.length != BLOCK_SIZE / 8) {
      throw new IllegalArgumentException("'iv' must be "+ BLOCK_SIZE +" bit");
    }
    this.iv = new IvParameterSpec(keyEncoer(iv)
  }
  // 암호화
  public String encrypt(String plain) {
    Cipher c = Cipher.getInstance(algorithm);
    c.init(Cipher.ENCRYPT_MODE, this.secretKey, this.iv);
    byte[] encrypted = c.doFinal(plain.getBytes(charset));
    return encoder.encode(encrypted);
  }
  // 복호화
  public String decrypt(String cipher) {
    Cipher c = Cipher.getInstance(algorithm);
    c.init(Cipher.DECRYPT_MODE, this.secureKey, this.iv);
    byte[] encrypted = encoder.decode(cipher);
    return new String(c.doFinal(encrypted), charset);
  }
}
{% endhighlight %}

#### 사용 예시
{% highlight java %}
public static Example {
  // 암호화 키
  static final String key = "12345678901234567890123456789012";
  // 초기화 벡터
  static final String iv = "1234567890123456";

  public static void main(String[] args) {
    // 암호화 객체 생성
    Crypto aes256 = new AES256Crypto(key, iv);

    String plain = "1234 가나다라 !@#$"
    // 암호화
    String cipher = aes256.encode(plain);

    // 출력
    System.out.println("plain = " + plain);
    System.out.println("cipher = " + cipher);

    // 복호화
    String plain2 = aes256.decode(cipher);
    System.out.println("plain2 = " + plain2);

    // 검증
    assert plain.equals(plain2);
  }
}
{% endhighlight %}

# 결론

**다음과 같은 스팩을 바탕으로 암호화 방식을 공유하자**

- 암호화 알고리즘 : **AES**
  - key size : **256 bit**
  - block size : **128 bit** (AES의 블록사이즈는 128로 고정입니다. 고로 block size는 skip해도 됩니다.)
- 암호화 모드 : **CBC**
- Padding 방식 : **PKCS5**
- 암호화키 : `12345678901234567890123456789012` (32자)
  - 초기화벡터 - CBC모드일 경우 : `1234567890123456` (16자)
- 암호문 인코딩 방식 : **Base64(바이트인코딩) + UTF-8(문자인코딩)**
- 키 인코딩 방식 - 암호문 인코딩과 다른경우 : **ASCII** (문자인코딩, 바이트인코딩 둘 다 가능)


  [2b86750a]: http://dxdy2x.comze.com/content/secret_aes.htm "블럭암호 AES"
  [a55c5334]: http://dxdy2x.comze.com/content/util_pad_mode.htm "Padding와 암호화 모드"
  [a1942bdd]: https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%9D%B4%EC%8A%A464 "위키백과-base64"
