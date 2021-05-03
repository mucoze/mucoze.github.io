---
title: "Linked List"
date: 2017-09-10 23:58 -05:00
category: C 
tags: [c, LinkedList]
---

Linked List’ler yani bağlı listeler, C Programlama Dilinde kullanılan kendine dönüşlü bir yapıdır(struct). “Kendine dönüşlü nasıl oluyo ? ” diyenler için azıcık daha açarsak : Bağlı listeler, içerisinde kendi türünden bir gösterici (pointer) içeren yapılardır diyebiliriz. Yine de kafanızda tam olarak şekillenmediyse endişelenmeyin konunun devamında tam olarak anlaşılacaktır. Gelecek konulara yabancı kalmamak için ‘pointer’ ve ‘struct’ terimlerinin anlamlarının ve işlevlerinin iyi bir şekilde anlaşılması gerekiyor. Eksiği olanlar BURADAKİ yazıyı inceleyebilir. <!--more-->

Bağlı listeler, veri yapıları konusudur ve bir biriyle ilişkili veriler üzerinde işlemler yapılırken kullanılırlar. Birçok kişinin bildiği diziler (arrays) ile benzer özelliklere sahiptir. İkisininde avantaj ve dezavantajları bulunmasına karşın, bağlı listelerin daha üstün özelliklere sahip olduğu açıktır. Bunlardan en önemlisi bağlı listelerin dinamik bir özelliğe sahip olmasıdır. Buradaki dinamikten kastımız; bağlı listeler üzerinde, çalışma sırasında aktif olarak işlemler (ekleme,silme gibi) gerçekleştirebiliriz. Bu işlemleri dizilerde de gerçekleştirmek mümkün ama diziler oluşturulurken en başta belirteceğimiz maksimum dizi boyutu hafızada gereksiz alan işgal etmemize neden olabilir. 
Kısaca bağlı listelerin ne olduğundan bahsettikten sonra, kendine dönüşlü yapıyı pratikte inceleyelim:

```c
struct node{
	int data;
	node *next;
};
```
Görüldüğü gibi ‘node’ adında bir yapımız var ve içerisinde, ‘data’ adında int türünden bir değişken ile kendi türünden bir yapı gösteren ‘next’ adında işaretçi bulunuyor. Bu yapının içerisine ihtiyaca göre farklı veri tiplerinde değişkenler de tanımlayabiliriz. Biz örneklemede basit olması açısından sadece bir adet değişken kullandık. 

Bağlı listeler yukarıda örnek olarak gösterdiğimiz yapılardan meydana gelir. Bu yapılara konu dahilinde ‘node’ yani düğüm adı verilir. Düğümler birbirlerine işaretçiler sayesinde bağlanırlar. Düşünülenin aksine, bağlı listelerde düğümler art arda gelmiyorlar. Hafızada her düğüm farklı yerde bulunur. Her düğüm kendinden sonraki düğümün adresini tutuyor. Yani her düğümde bir veri alanı ve diğer düğümü işaret eden bir işaretçi bulunuyor. Bildiğimiz üzere işaretçi burada adres tutmak amacı ile kullanılıyor. 

Bağlı listeler konusu başlarda karmaşık ve anlaşılması zor gözükse de çok basit ve zevkli bir konudur. Hafızada gerçekleştirdiğimiz işlemler sırasında aynı zamanda kafamızda da kurgulama yapmamız gerektiği için düşünme kabiliyetimizin gelişmesine katkı sağlıyor. 
Yukarıda bahsettiğimiz düğüm yapısını iyi bir şekildi kavrayabilirsek ilerleyen aşamaları anlamamız daha da kolaylaşacaktır.
İsterseniz konunun görselleştirilmesi ve daha kolay anlaşılması için düğümleri vagonlara ve ilk düğüm yani başlangıç düğümünü lokomotife benzetelim.

![_config.yml]({{ site.baseurl }}/images/L1.png)

Her bağlı listenin root veya head denilen bir kök göstericisi vardır.( Şekildeki lokomotif root’u temsil ediyor) Herhangi bir vagona (düğüm) erişmek istediğimizde önce lokomotife erişmemiz gerekiyor. Nasıl bir treninin yönetimi lokomotifte ise, bağlı listelerde de root göstericisindedir. Lokomotif yoksa tren de yok yani root yoksa bağlı liste de yok  Her vagonda bulunan işaretçi kendinden sonra gelen vagonu işaret eder ve böylelikle vagonlar bir birine BAĞLANMIŞ olur. Listenin en sonundaki düğümün işaretçisi her zaman NULL değerini gösterir. Bu listenin sonunu belirtmek için yapılır.

Şekil üzerinde anlatmak kolay, şimdi bunları koda dökelim…
İlk önce, bağlı listeler oluşturulurken kullanılan malloc() ve sizeof() fonksiyonlarından bahsedelim. Kısaca, sizeof ile elimizdeki yapının boyutunu alırız. Malloc ile de hafızada o kadar alanlık bir yer ayırırız. 


```c
#include <stdio.h>
#include <stdlib.h>

struct n{
        int data;
        n * next;
};
typedef n node; 

int main(){
        node * root; // ilk önce root u oluşturduk.
        root = (node *)malloc(sizeof(node)); //Root un gösterdiği yere, Birtane düğümün hafızada kapladığı yer kadar hafızada bana yer ayır.
        root -> data = 3;  // root un gösterdiği kutunun data kısmına 3 yaz. 
        root -> next = (node *)malloc(sizeof(node));
        root -> next -> data = 4;
	   root -> next -> next = NULL;
        printf("%d,%d", root -> data, root -> next -> data);
}
```
Yukarıdaki kod ile bir root oluşturduk ve root’un işaret ettiği yere ilk vagonumuzu koyduk. Burada (node *) neden var merak edenler  [şu](http://www.tutorialspoint.com/cprogramming/c_type_casting.htm) konuya bakıp gelebilir. Node struct türündeki Next işaretçisi ile yine root’un gösterdiği vagonun devamına yeni bir vagon oluşturup data kısmına 4 değerini atadık. Son kutunun nextine de NULL değerini eşitleyip bağlantıyı sonlandırdık. Son kısımda ise ilk vagonların data kısmlarını yazdırdık. Çok basit tekniklerle bağlı liste bu şekilde oluşturuluyor. 
Listedeki düğümlerin çok olduğunu düşünürsek, bu yöntemle işi yokuşa süreriz. O yüzden itarotor (dolaşıcı) ve özel fonksiyonlar kullanacağız. Bu fonksiyonlar ekleme,slime,yazdırma. Bağlı listelerde ekleme 3 farklı yere yapılabilir : En başa, herhangi bir düğüm arasına ve en sona. 
Adından da anlaşılacağı üzere dolaşıcı, bağlı liste üzerinde hareket edebilen bir düğümdür ve veri aranması, ekleme veya silme gibi işlemler sırasında listenin ilgili elemanına kadar gidilmesini sağlar. Root’a eşittir, tek farklı hareket etmesidir. Bu hareketli arkadaşımız sayesinde yaptığımız işlemler daha kolaylaşacak ve listeyi etkilememiş olacağız.

Diğerlerine göre daha basit olduğu için ilk önce sona vagon ekleme işlemini yapalım.

```c
void bastir(node * r) {
	while(r!=NULL){
		printf("%d \n",r->data);
		r = r->next;	
	}
}
int main(){
        node * root;
        root = (node *)malloc(sizeof(node));
        root -> data = 3;
        root -> next = (node *)malloc(sizeof(node));
        root -> next -> data = 4;

	node * dolas;
	dolas = root;
	while(dolas->next != NULL){
		dolas = dolas->next;
	}
	dolas->next = (node *)malloc(sizeof(node));
	dolas=dolas->next;
	dolas->data = 5;
	dolas->next = NULL;
	bastir(root);
}
```
Main fonksiyonu inceleyecek olursak, daha önce 3 ve 4 verilerinin bulunduğu vagonları oluşturmuştuk. Şimdide dolaşıcı vasıtası ile listenin sonuna geldik. Bu noktada bir tane node struct türünden dolas adında bir vagon oluşturduk ve dolaşıcının root kök göstericisinin gösterdiği yeri onunda göstermesini sağladık. While döngüsü yardım ile de dolaşıcının next inin NULL değere sahip olana dek ilerlemesini sağladık. Dolaşıcının next ‘ i dedik çünkü ; ekleme işlemlerinde ekleme yapılacak yere varmadan bi düğüm öncesinde durulması gerekiyor. Aksi taktirde ekleme işlemi yapılsa dahi bağlantı yapılamaz ve kopma meydana gelebilir.
Listenin sonuna geldiğimizde ise son geldiğimiz noktaya bir vagon ekledik ve data kısmına 4 değerini atadık. Bastır fonksiyonu ile de tüm vagonların verilerini gördük.
Bastır fonksiyonu paremetre olarak aldığı kök gösterici NULL değere sahip olana kadar ilerliyor ve her nokta da verileri yazdırıyor.

Bir sonraki işlemde herhangi bir vagon arasına ekleme yaptıracağız. Eklemeye geçmeden önce şekil üzerinde inceleme yapalım.

![_config.yml]({{ site.baseurl }}/images/L2.png)

------------------------------------------------
![_config.yml]({{ site.baseurl }}/images/L3.png)

------------------------------------------------
![_config.yml]({{ site.baseurl }}/images/L4.png)

Vagon ekleme işlemi için aşağıdaki adımlar gerçekleşitirilir:
1- Ekleme işlemini yapılacağı aralıktan önceki vagona dolaşıcı tarafından gidilir.
2- Yeni vagon oluşturularak, sonrasına, dolaşıcının sonrası atanır.
3- Dolaşıcının sonrasına ise yeni vagon atanır.

Bu incelediğimiz tabiki bir çizgi film senaryosu değil animasyon da değil  sadece hayal gücümüzü kullanarak gerçekleşen adımları görselleştiriyoruz. Şimdi gerçeklerle yüzleşip C kodunu inceleyelim…

```c
void ekle(node * r, int data){  //dolaşıcıya gerek kalmadan, sona eklemeyi fonksiyonla yaptık.
	while(r->next!=NULL){
		r = r->next;
	}
	r->next = (node *)malloc(sizeof(node));
	r->next->data = data;
	r->next->next = NULL; //List in sonunda her zaman NULL olmalı ki sonradan yine ekleme yapabilelim.
}
int main(){
        node * root;
	root = (node *)malloc(sizeof(node));
	root -> next = NULL;  
	root -> data = 3;
	int i = 0;
	for(i=4;i<7;i++){
		ekle(root, i);
	}
	node * dolas;
	dolas = root;
	for(i=0;i<2;i++)
		dolas = dolas -> next;
	node * yeni = (node *)malloc(sizeof(node));  
	yeni -> next = dolas -> next;
	dolas -> next = yeni;
	yeni -> data = 100;		
	bastir(root);
}

```

Yukarıdaki sona ekleme örneğinde yaptığımız ekleme işlemi için fonksiyon oluşturduk. Fonksiyon parameter olarak bir kök gösterici ve oluşturulacak vagon için veri alıyor. Main fonksiyonda ise bu ekle fonksiyonu yardımı ile ard arda 4 vagon(3,4,5,6) ekledik. Bir sonraki aşamada tekrar bir dolaşıcı oluşturduk ve for döngüsü ile 3 adım ilerledik. Buradaki ilerlemeyi, o an ki ekleme yapmak istediğimiz noktaya göre ayarlıyoruz. Biz bu örnek ile 5 ve 6 verileri bulunduran vagonlar arasına yeni bir vagon oluşturduk ve data kısmına 100 değerini koyduk.

Ekleme işlemlerinde bahsetmediğimiz, en başa ekleme kaldı. Bu ekleme işleminde dikkat etmemiz gereken en önemli nokta. En baştaki düğüm, kök(root) göstericisinin işaret ettiği düğümdür. Dolayısıyla başa ekleme yaptığımızda kök göstericide değişklik olacaktır.

```c
node * basaEkle(node * r, int data){
	node * yeni2 = (node *)malloc(sizeof(node));
	yeni2 -> data = data;
	yeni2 -> next = r;
	return yeni2; //yeni root u döndür

}
int main(){
	node * root;
	root = (node *)malloc(sizeof(node));
	root -> next = NULL;  
	root -> data = 3;
	int i = 0;
	for(i=4;i<7;i++){
		ekle(root, i);
	}
	root = basaEkle(root, 111);
	bastir(root);
}

```
basaEkle() fonksiyonu fark ettiğiniz gibi diğerlerinde ki gibi void değil, geriye node * türünde bir vagon döndürüyor. Bu vagon yukarıda bahsettiğimiz gibi en başa eklediğimiz ve kök göstericisinin işaret ettiği vagondur. İlk önce yeni bir düğüm tanımlayıp bir node luk yer tahsis ettik ve parametre de belirtilen 111 değerini veri olarak yerleştirdik. Sonra da yeni2 adlı vagonumuzun next ini root un next i yaptık ve en can alıcı nokta olan bağlamayı gerçekleştirdik. Böylelikle lokomotifin işaret ettiği ilk kutu değişmiş oldu. Yeni sıralamamız  111 3 4 5 6 şeklinde.

Ekleme işlemleri bittikten sonra şimdi sırada slime işlemi var. C dilinde slime işlemleri için free() fonksiyonu kullanılır. Aslında silmek istediğimiz vagonun bağlantılarını listeden ayırdığımızda vagon kaybolmuş olur. Ama hafızada gereksiz alan işgal etmemesi açısından bağlantılar koparıldıktan sonar slime işlemi de yapılmalıdır. Silme işleminde, eğer silinmek istenen vagon kök göstericinin işaret ettiği vagonsa yani ilk vagonsa burada da root düğümü değişikliğe uğrayacaktır. Sonuçta root bir işaretçidir ve işaret ettiği ilk düğüm listenin başlangıcıdır. Bazı kaynaklarda buna kısaca root düğümüde denilebilir. 


```c
node * sil (node * r, int data){
	node * bos;
	node * dolas = r;
	if(r->data == data){  //eğer silinen root düğümü ise 
		bos = r;
		r = r->next;
		free(bos);
		return r;
	}
	while(dolas->next != NULL && dolas->next->data != data){ //
		dolas = dolas->next;
	}
	if(dolas->next == NULL){ //listenin sonuna geldik ve silinmek istenen bulunamadıysa
		printf("%d sayisi bulunamadi\n",data);
		return r;
	}
	bos = dolas->next;
	dolas->next = dolas->next->next;
	free(bos);
	return r;
	
}
int main(){
	node * root;
	root = (node *)malloc(sizeof(node));
	root -> next = NULL;  
	root -> data = 3;
	int i = 0;
	for(i=4;i<7;i++){
		ekle(root, i);
	}
	root = sil(root, 5);
	bastir(root);
}

```
İf yapısı ile, silinmek istenen vagonun ilk vagon olması durumunda olacakları kodladık. İlk önce bos adında bir dolaşıcı tanımlayıp root a eşitledik. Böylelikle bos dolaşıcısı root un gösterdiği yeri gösterecek. Yani silinmek istenileni. Bir sonraki adımda ise root un gösterdiği yeri bir adım sonraki vagon olarak ayarladık. Ve bos dolaşıcısının işaretettiği ilk vagonu free() fonksiyonu ile imha ettik. Son kısımda ise yeni root göstericimizi döndürdük.
Eğer silinmek istenilen ilk vagon değilse, while() döngüsü ile listenin sonuna gelene kadar ilerliyoruz. Bulduğumuz anda While() dögüsünden çıkıyor ve slime işlemine geçiyoruz. Bu noktaya ‘dolas’ adlı dolaşıcı ile geldik. Tabi geldiğimiz nokta silinmek istenilen vagondan bi önceki vagon. Silinmek istenilen vagonun üstüne geldiğimizde slime işlemini yapamayız. Sonuçta silme yapılırken işaretçiler ile kurulan bağlantılar koparılıyor. Dolayısıyla bir önceki vagondan bağlantıyı ayırmamız ve silinmek istenilen vagondan sonraki vagona bağlantıyı gerçekleştirmeliyiz.
Yukarıda yaptığımız gibi burda da bos dolaşıcısı ile silinmek istenilene bağlantı kurup slime işlemini tamamlıyoruz. Root düğümü değişmediği için yine root’u döndürüyoruz.
Eğer listenin sonuna geldik vesilinmek istenilen vagonu bulamadıysak ekrana ‘bulunamadı’ mesajı yazdırılıyor. Biz bu örnekte 5 değerine sahip vagonu şutladık.

Bağlı listelerde işlemler bu kadardı. İlk başlarda konuya çabuk hakim olunması için kağıda çizerek adım adım işlemler yapmak yararlı olacaktır diyip diğer bağlı liste türleri olan ‘dairesel bağlı liste’ ve ‘çift yönlü bağlı listeye’ye geçiyoruz. 

Dairesel Bağlı Liste

Dairesel bağlı listenin bildiğimiz bağlı listeden tek farkı sonunun olmamasıdır. Normalde listenin son düğümü NULL değerini gösteriyordu. Dairesel bağlı listed ise son düğümün ilk düğümü işaret eder. Herhangi bir düğüm aradığımızda eğer dolaşıcının bir turu tamamladığında durmasını sağlamazsak, işlem sonsuza gider.  Bunun için dolaşıcının işaret ettiği düğüm, root’un işaret ettiği düğüme eşit olduğunda durmalıyız.
Örneğin bastir() fonksiyonunda While döngsünde eşitlemeyi NULL değil kök gösterici yapmalıyız.


```c
void bastir(node * r) {
	node * dolas = r;
	printf(“%d \n”,dolaş->data); //ilk elemanı bastırmalıyız ki başlangıçta rootu atlamsın.
	dolas=dolas->next; 
	while(dolas!=r){
		printf("%d \n",dolas->data);
		dolas = dolas->next;	
	}
}
```
Geri kalan işlemlerde pek bir fark yok. Konunun çok uzamaması için ayrıntıya girmeyeceğim. 

Çift Yönlü Bağlı Liste

Bu tür bağlı listelerde esasında çok farklı değil, küçük değişiklikler mevcut. Bunlardan ilki ve listeyi çift yönlü yapan özellik ise normal listeye artı olarak bir ’geri’ işaretçisinin olmasıdır. Yani normal bağlı listede her düğümde iki alan varken çift yönlü bağlı listelerde üç alan bulunuyor (prev,data ve next). Nasıl next işaretçisi ileriyi işaret ediyorsa prev işaretçisi de geriyi yani bi önceki vagonu işaret ediyor. Bu sayede liste de ileri gittiğimiz gibi herhangi bir anda geri de gitmemiz mümkün oluyor. Listemizin aynı şekilde bir kök göstericisinin yani lokomotifinin olması gerekiyor ve ilk prev lokomotifi(root) göstermeli. 
Burada dikkat etmemiz gereken iki nokta bulunuyor. Birisi silme veya ekleme işlemlerinde prev işaretçisinin de bağlantılarını gerçekleştirmeyi unutmamalıyız. Diğer nokta da, NULL değerinin prev işaretçisinin bulunmadığı ve ilk düğümün prev işaretçisinin başka bir NULL değerini gösterdiği bilinmelidir.
Yeni struct yapımızda doğal olarak bir adet daha kendi türünden işaretçi olmalı.


```c
struct n{
        int data;
        n * next;
        n * prev;

};
typedef n node; 
```
![_config.yml]({{ site.baseurl }}/images/L5.png)

Tren yoldan çıkmış gibi gözükse de çift bağlı listeleri şekil üzerinde anlatmak istersek böyle bir yapı oluşuyor.
Konumuzun sonuna geldik, son olarak birkaç madde belirtip bitirelim…
Bir Linked List içerisinde başka türden struct’lar da olabilir.
Linked List’ler doğrusal erişimlidir, diziler random.
Herhangi bir ‘Segmantation fault’ hatası ile karşılaşırsınız bunun anlamı : Hafızada bazı çakışmalar meydana geldi demektir. Bunun için listenin sonunu NULL değere eşitlemeyi unutmayın.

