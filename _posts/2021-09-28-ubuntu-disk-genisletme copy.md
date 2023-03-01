---
title: 'Ubuntu 20.04 Sunucu Üzerinde Disk Genişletme (Disk Extend)'
date: 2021-09-28
author: Furkan Bilgin
layout: post
permalink: /ubuntu-disk-genisletme/
categories: ubuntu
tags: [ubuntu,centos, linux, disk, fdisk]
img: /assets/img/posts/ubuntu_extend_disk/cover.png
description: 'Bu öğreticide sizlerle Ubuntu işletim sistemine sahip bir sunucuda terminal üzerinden disk genişletme(disk extend) işleminin nasıl yapıldığını göreceğiz.'
---

Bu öğreticide sizlerle Ubuntu işletim sistemine sahip bir sunucuda terminal üzerinden disk genişletme(disk extend) işleminin nasıl yapıldığını göreceğiz.

Windows tarafında user interface (UI) alışkanlığı olan kişiler bu işlemi ilk yaptıklarında zorlanabilirler ancak tekrar ettikçe ve mantığını kavradıkça konu kolaylaşacaktır.

İşlemlere başlamadan önce aşağıdaki aşağıdaki komutu çalıştırarak hali hazırda disklerin son durumu görelim. 
Dilerseniz çıkan sonucun fotoğrafını çekerek, işlem sonuyla kıyaslama yapabilirsiniz.

{% highlight c %}
df -h
{% endhighlight %}

![Picture description](/assets/img/posts/ubuntu_extend_disk/df-h.png){: .center-image }

Yukarıdaki görselde kırmızı kutu içine aldığım alanı /dev/sda2 diskini genişleteceğim.

Dilerseniz çıkan sonucun fotoğrafını çekerek, işlem sonuyla kıyaslama yapabilirsiniz.

Şimdi sunucuyu kapatarak diski dilediğiniz kadar genişeletebilirsiniz. Ben 70 gb olan diskime 30 gb daha ekleme yapacağım.
Bu adımdan sonra mutlaka sunucuyu yeniden başlatınız. Aksi taktirde işlem bittiğinde tekrar en başa dönmeniz gerekecektir.
Ben sunucumu yeniden başlattım ve işlemlere devam ediyorum.

Şimdi parted fonksiyonunu kullanarak diski genişleteceğiz.

{% highlight c %}
parted /dev/sda
{% endhighlight %}

![Picture description](/assets/img/posts/ubuntu_extend_disk/parted.png){: .center-image }

Çıkan ekran "help" yazarak seçenekleri görebiliriz. 
Sırasıyla aşağıdaki komutları giriyoruz. 
	
Hangi partition büyütmek istediğimizi soruyor. 
List yazarak partition listesi görebiliriz.
Biz dev/sda2 yani 2 numaralı partition exten edeceğiz.

{% highlight c %}
(parted) resizepart 2                                                  
{% endhighlight %}

2 numaralı partition kullanıldığı için bizden onay istiyor 
{% highlight c %}
Warning: Partition /dev/sda2 is being used. Are you sure you want to continue?
Yes/No? Yes                                                            
{% endhighlight %}

Şimdi bize yeni eklenen alanın ne kadarını dahil edeceğimizi soruyor.
-0 dersek tamamını eklemiş olacağız.

{% highlight c %}
End?  [70GB]? -0                                                      
{% endhighlight %}

{% highlight c %}                                                   
(parted) quit                                                          
Information: You may need to update /etc/fstab.
{% endhighlight %}

Evet neredeyse işlemlerin sonuna geldik. Artık "parted /dev/sda" ile işimiz bitti.
Diski şimdi resize etmemiz gerekiyor. Bunun için ilgili diskin adı ile aşağıdaki komutu çalıştırıyorum. 

{% highlight c %}
resize2fs /dev/sda2
{% endhighlight %}

İşlemler bu kadar şuanda diskiniz genişledi. Görmek için tekrar df -h komutunu çalıştırıyorum.

{% highlight c %}
df -h
{% endhighlight %}

![Picture description](/assets/img/posts/ubuntu_extend_disk/new-df-h.png){: .center-image }


<b>Not: Bazı kelime ve kavramları yanlış kullanmış olabilirim. Düzeltme için lütfen e-mail atınız.<b>