---
title: "Symbolic Execution"
date: 2018-03-25 21:00 -05:00
category: C 
tags: [c, python, angr, symbolic_execution, fuzzing, crack]
---

Symbolic execution, bir programın girdi seçeneklerini belirlemek için kullanılan bir analiz tekniğidir. Temel amaç, programın normal çalışmasını sağlayan somut (concrete) girdiler yerine sembolik değerler ile soyut bir yorumlama yolunu izlemektir. Girilen her sembolik değer ile birlikte programda bulunan her koşul biriminde mümkün olan her bir dallanma için bir sembolik ifade (veya denklem) oluşur. En sonunda programa ait bir akış grafiği (flow graph) meydana gelir. Bu aşamaları otomatik olarak symbolic execution motoru gerçekleştirir.<!--more-->


![_config.yml]({{ site.baseurl }}/images/se.jpg)
Kaynak: https://www.youtube.com/watch?v=QrtGOrSrVPQ 

Symbolic execution, aslında formül olarak olası yolları temsil etme sürecidir. Burada bahsettiğimiz symbolic değer (örn:'*'), bir koşulun işletilmesi anında "herhangi bir değeri" alabilmektedir, yani temsili olarak orada bulunmakta. Bu sayede her yol (path) keşfedilebilir oluyor. 
SMT çözücüleri kullanılarak uygunluk doğrulaması ve bu ifadelere çözüm sunmak suretiyle, herhangi bir ulaşılabilir uygulama noktasına ulaşmak için gerekli olan somut değerler üretilir.
Süreç kısaca şöyle işler; symbolic execution motoru önce tüm olası yolları sıralar. Her bir yol için motor, yolların bağlı olduğu koşul tarafından uygulanan kısıtlamaları kaydeder. Oluşan birçok ifade içerisinden tercih ettiğimiz yolu seçip formülün çözülmesini sağlayıp gerekli somut input’a ulaşıyoruz.
Bu noktada işin asıl can alıcı noktası formülün çözüm aşamasıdır. Bu kısmı daha iyi anlamak ve biraz daha derinlere inmek için [SAT](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem) ve [SMT](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories) terimleri incelenebilir.
Şimdi basit bir uygulama ile teoriyi uygulamaya geçirelim…

```c
#include <stdio.h>
#include <string.h>

 char *mcz = "MUCOZE";

 int main(int argc, char *argv[]){
 char password[20];
 scanf("%s", &password);
 if(strcmp(mcz, password) != 0){
 printf("wrong!");
 return 1;
 }
 printf("Win!\n");
 system("/bin/sh");
 return 0;

```

![_config.yml]({{ site.baseurl }}/images/login.png)

Yukarıdaki C kodu anlaşılacağı üzere kullanıcıdan alınan password girdisi ile "MUCOZE" dizisini karşılaştırıyor. İstenilen değerin girilmesi durumunda ise "Win!" mesajı ile birlikte bize bir kabuk açıyor. Bizden istenilen bu girdiyi bulmak için [Angr](http://angr.io/) adli python kütüphanesini kullanacağız. Angr, symbolic execution dahil birçok binary analiz tekniğini uygulayabilmemizi sağlıyor.

```python
#!/usr/bin/env python
import angr
p = angr.Project('./login')  #proje dosyasını belirttik
state = p.factory.entry_state()  #programın canlı verileri kaydedildi

sm = p.factory.simulation_manager(state)  #yeni bir simule ortam oluşturuldu.
sm.step(until=lambda lpg: len(lpg.active) > 1) 
#kaydedilen durumlar ile symbolic execution başlatıldı. 
#Bu komut ile birlikte birden fazla aktif yol oluşana kadar adim adım çalıştırma sürdü.
#Ta ki amaç yol bulunana kadar.

input_0 = sm.active[0].posix.dumps(0) #Sonuç kaydedildi

print input_0

```

![_config.yml]({{ site.baseurl }}/images/solv.png)

Yazdığımız bir kaç satirlik python kodu ile kısa surede sonuca ulaştık...

![_config.yml]({{ site.baseurl }}/images/win.png)

Symbolic execution, tekniğini gerçekleştirebileceğimiz birçok araç ve kütüphane olmasına karşın, angr`nin başlangıç için uygun olduğunu düşünüp bu kütüphane ile çalışmaya başladım. Çok ayrıntılı ve soyut bir konu olduğundan kafamda hala birleşmeyen noktalar var. Buraya kadar anlattıklarım birilerine bir şeyler katar, fikir sağlarsa ne mutlu 


Kaynak
http://www.vantagepoint.sg/blog/81-solving-an-android-crackme-with-a-little-symbolic-execution

http://www.usrsb.in/symbolic-execution-intuition-and-implementation.html 

https://github.com/ksluckow/awesome-symbolic-execution 

http://web.cs.iastate.edu/~weile/cs641/9.SymbolicExecution.pdf


