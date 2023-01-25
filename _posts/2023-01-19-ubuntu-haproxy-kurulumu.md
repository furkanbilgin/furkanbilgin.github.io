---
title: 'Ubunut 20.04 Sunucu Üzerine Haproxy Kurulumu'
date: 2023-01-19
author: Furkan
layout: post
permalink: /install-ubuntu-haproxy/
categories: haproxy, loadbalance, haproxy, ubuntu, centos, rhel
tags: [haproxy,loadbalance,ubuntu,centos ,rhel ,centos , linux]
img: posts/2023-01-19-ubuntu-haproxy-kurulumu/cover.png
---

Bu öğreticide sizlerle Ubuntu 20.04 işletim sistemine sahip bir sunucuda terminal üzerinden haproxy yükleyerek configurasyonları hakkında bilgi vereceğim.

İşlemlere başlamadan önce sunucunuzun yedeğini almanızı tavsiye ederim. (snapshot, full backup vs.)

Öncelikle ürünün kendi resmi sitesine gidelim ve kurulum için gerekli komutları alalım
https://haproxy.debian.net/

Sunucuda ubuntunun hangi sürümünün kurulu olduğunu anlamak için aşağıdaki komutu çalıştırabilirsiniz.
![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/hostnamectl.png){: .center-image }

Ben ubuntu 20.04.5 üzerine bu tarihteki son haproxy sürümünün kurulumunu yapacağım için seçimlerimi bu yönde yaptım.

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/command.png){: .center-image }

1- Adım: haproxy yüklemek için sırasıyla aşağıdaki komutlarımızı çalıştıralım.

{% highlight c %}
apt-get install --no-install-recommends software-properties-common
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-noins-rec.png){: .center-image }

{% highlight c %}
add-apt-repository ppa:vbernat/haproxy-2.7
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-add-repo.png){: .center-image }

Aşağıdaki komutun sonuna -y parametresini eklerseniz benim gibi kurulum sırasında tekrar enter ve sonrasında Y tuşuna basmanıza gerek kalmaz

{% highlight c %}
apt-get install haproxy=2.7.\*
{% endhighlight %}
veya
{% highlight c %}
apt-get install haproxy=2.7.\* -y
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-install.png){: .center-image }

2- Adım: Kurulumun tamamlandığına emin olmak için haproxy versionunu sorguluyoruz. Komutumuz aşağıdaki şekildedir.

{% highlight c %}
    haproxy -v
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/haproxy-v.png){: .center-image }

3- Adım: Kurulum bittiğine göre şimdi sunucu restart olduktan sonra haproxy servisinin otomatik olarak yeniden başlaması için aşağıdaki komutu çalıştırıyoruz.

{% highlight c %}
systemctl enable haproxy
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-enable.png){: .center-image }

4- Adım: Haproxy ayarlamalarına geçmeden önce son olarak haproxy durumunu sorgulamamız gerekiyor. Status çekerek servisin aktif olup olmadığını göreceğiz.

{% highlight c %}
systemctl status haproxy
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-status.png){: .center-image }

Şimdi haproxy ile proxy işleminin nasıl yapılacağını göreceğiz.
Buna başlamadan önce bazı kavramları açıklamak istiyorum.

**Global :**

Yapılandırma(haproxy.conf) dosyanızın en üstünde görünür. Kabul edilecek maksimum bağlantı sayısı, günlüklerin(logların) nereye iletileceği ve HAProxy Enterprise modüllerinin yükleneceği yol gibi yönergeleri tanımladığımız alandır.

**Default:**

Fronted, backend veya listen bölümünde ayarlayabileceğiniz parametlerin çoğunu buradan varsayılan olarak belirleyebilirsiniz. Daha sonra frontend veya backend kısmında bu ayarları tekrar belirtirseniz frontend veya backend altındaki belirttiğiniz parametreler aktif olmuş olur.

**Listen:**

Bu bölümde haproxy istatisklerini görmek için tanımlama yapabiliriz.
Örnek görsel aşağıdaki şekildedir.

**Frontend:**
Bu alanda bir frontend oluştururuz ve gideceği backend serverını belirtiriz. Yine bu alanda hangi protokol ve port üzerinden iletişim kurulacağı belirtilir.

**Backend:**
Front end alanında oluşturulan frontendler için isteklerin yönlendirileceği serverlerı belirtiriz. Yine bu alanda hangi protokol ve port üzerinden iletişim kurulacağı belirtilir.

Önemli olduğunu düşündüğüm bu başlıkları açıkladıktan sonra şimdi örnek bir conf düzenleyebiliriz.
