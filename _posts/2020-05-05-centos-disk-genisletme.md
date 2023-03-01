---
title: 'Centos 7 Sunucu Üzerinde Disk Genişletme (Disk Extend)'
date: 2020-05-06T02:51:00+00:00
author: Furkan
layout: post
permalink: /centos-disk-genisletme/
categories: Centos
tags: [centos, linux, disk, fdisk]
img: /assets/img/posts/extend_disk/lvm.png
description: 'Bu öğreticide sizlerle Centos işletim sistemine sahip bir sunucuda terminal üzerinden disk genişletme(disk extend) işleminin nasıl yapıldığını göreceğiz.'
---

Bu öğreticide sizlerle Centos işletim sistemine sahip bir sunucuda terminal üzerinden disk genişletme(disk extend) işleminin nasıl yapıldığını göreceğiz.
Maalesef bu yazıda sizlere "next>next>next" diyerek işlem yaptıramayacağım :) 
Windows tarafında user interface (UI) alışkanlığı olan kişiler bu işlemi ilk yaptıklarında zorlanabilirler ancak tekrar ettikçe ve mantığını kavradıkça konu kolaylaşacaktır.

İşlemlere başlamadan önce aşağıdaki aşağıdaki komutu çalıştırarak hali hazırda disklerin son durumu görelim. 
Dilerseniz çıkan sonucun fotoğrafını çekerek, işlem sonuyla kıyaslama yapabilirsiniz.

{% highlight c %}
df -khT
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/df-h.png){: .center-image }

Yukarıdaki görselde kırmızı kutu içine aldığım alanı genişleteceğim, yeşil kutu içerisinde olan ibare ise bu alanın tipi, en son işlem sırasında bize lazım olacak.

Daha sorna partionları listeleyerek durumu görelim. 
Dilerseniz çıkan sonucun fotoğrafını çekerek, işlem sonuyla kıyaslama yapabilirsiniz.

{% highlight c %}
fdisk -l
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/fdisk-l.png){: .center-image }

Şimdi sunucuyu kapatarak diski dilediğiniz kadar genişeletebilirsiniz. Ben 100 gb olan diskime 10 gb daha ekleme yapacağım.
Eğer diski sunucu açıkken genişletir ve daha sonra sunucuyu yeniden başlatmazsanız ilerleyen adımlarda aşağıdaki şekilde bir hata alacaksınız.

{% highlight c %}
No free sectors available
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/no-free.png){: .center-image }

Ben sunucumu yeniden başlattım ve işlemlere devam ediyorum.

Şimdi bir partition oluşturmamız gerekiyor. Bunun için aşağıdaki komutu çalıştırıyorum.

{% highlight c %}
fdisk /dev/sda
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/fdisk-dev-sda.png){: .center-image }

Çıkan ekran "m" tuşuna basarak seçenekleri görebiliriz. 
Sırasıyla aşağıdaki komutları giriyoruz. 
<B>İşleme başlmadan önce alttaki görseli lütfen inceleyiniz.</B>

{% highlight c %}
n
p
default seçenek için sadece enter diyerek geçiyoruz
default seçenek için sadece enter diyerek geçiyoruz
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/fdisk-n.png){: .center-image }

Şimdi oluşturdugumuz partion için tip belirteceğiz. 
Burada partion için tip belirtirken başta çalıştırdığımız "fdisk -l" komutunda gözüken disk tipini seçeceğim.
Benim disk tipim "Linux LVM"
Bunun için aşağıdaki komutları sırasıyla giriyorum. İşleme başlmadan önce alttaki görseli lütfen inceleyiniz.

{% highlight c %}
t
default seçenek için sadece enter diyerek geçiyoruz
L
8e
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/fdisk-t.png){: .center-image }

Şimdi oluşturdugumuz diski, disk tablosuna yazmamız gerekiyor.
Bunun için aşağıdaki komutu çalıştırıyorum. İşleme başlmadan önce alttaki görseli lütfen inceleyiniz.

{% highlight c %}
w
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/fdisk-w.png){: .center-image }

Evet neredeyse işlemlerin sonuna geldik. Artık "fdisk /dev/sda" ile işimiz bitti.
Disklerin durumunu güncellemek için aşağıdaki komutu çalıştırıyorum. 
Eğer aşağıdaki komutu çalıştırmadan devam ederseniz bu <B> "Device /dev/sda4 not found.</B> hatayı alacaksınız.

{% highlight c %}
partprobe
{% endhighlight %}

Fiziksel alanı oluşturmak için aşağıdaki komutu kullanıyorum.
Komut içerisinde bulunan /dev/sda4 kısmını partition oluşturduğumuzda, komut çıktısında bize gösterilen partition id yi yazıyoruz.
Bu sizde 2 veya 3 veya (n) olabilir ...
Benim partion ID 4.  yani ben /dev/sda4 için işlem yapacağım.
Eğer 5 olsaydı /dev/sda5 için işlem yapacaktım.
Görselde yeşil kutucuk içine aldıgım alanda benim partition id gözüküyor.
Bu görseli görüntülemek için <a href="https://furkanbilgin.com/assets/img/posts/extend_disk/fdisk-n.png" target="_blank">tıklayınız.</a>

{% highlight c %}
pvcreate /dev/sda4
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/pvcreate.png){: .center-image }

Fiziksel alanı oluşturduktan sonra bu alanı diğer alanlar ile aynı gruba alıyoruz. 
Benim fiziksel alan grup adım "centos" , bunu görüntülemek için en başta çalıştırdığımız ve ekran görüntüsü aldığınız "pvdisplay" komutuna bakabilirsiniz.

{% highlight c %}
vgextend centos /dev/sda4
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/vgextend.png){: .center-image }

Disk oluşturduktan sonra, bu diski istediğiniz diske dahil ederek diski genişletebilirsiniz.
Komut içerisindeki "+100%FREE" ibaresi tüm boş alanı hedef gösterilen alana genişlet manasına geliyor, dilerseniz o alana +10GB  veya +5GB yazarak dilediğiniz kadar genişleme sağlayabilirsiniz.
Ben tüm boş alanı genişletmek istiyorum, bunun için aşağıdaki komutu kullanıyoruz. 

{% highlight c %}
lvextend -l +100%FREE /dev/mapper/centos-root
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/lvextend.png){: .center-image }

Diskinizin tipini öğrenmemiz gerekiyor, bunun için aşağıdaki komutu çalıştırıyoruz.

{% highlight c %}
df -khT
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/df-khT.png){: .center-image }

ve son olarak diskimizin tipi xfs oldugu için aşağıdaki komutla diski resize ediyoruz.
Eğer diskinizin tipi farklı birşey ise bir alt satırdaki komutu deneyiniz.

{% highlight c %}
xfs_growfs /dev/mapper/centos-root
{% endhighlight %}

![Picture description](/assets/img/posts/extend_disk/xfs_growfs.png){: .center-image }

xfs_growfs komutunda hata alınırsa aşağıdaki komut denenebilir.
{% highlight c %}
resize2fs /dev/mapper/centos-root 
{% endhighlight %}
 
şimdi yeni diskinizi doldurana kadar kullanabilirsiniz.

Ve son olarak aşağıdaki görseli incelemeniz ve kavramları araştırmanızda fayda var.

![Picture description](/assets/img/posts/extend_disk/lvm.png){: .center-image }

<b>Not: Bazı kelime ve kavramları yanlış kullanmış olabilirim. Düzeltme için lütfen bana ulaşınız. <b>