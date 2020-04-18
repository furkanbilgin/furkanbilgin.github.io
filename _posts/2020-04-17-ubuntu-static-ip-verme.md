---
title: 'Ubuntu Tabanlı İşletim Sistemlerinde Static Ip Tanımlama'
date: 2020-04-17T00:00:00+00:00
author: Furkan
layout: post
permalink: /ubuntu-tabanli-isletim-sistemlerinde-static-ip-tanimlama/
categories: Ubuntu
tags: [ubuntu, staticip, network, ip]
img: posts/staticip/ip-addr.png
---

Bugünki yazımızda ubuntu tabanlı işletim sistemlerinde terminal üzerinde nasıl static ip tanımlanacağını göreceğiz.

Öncelikle düzenleyeceğimiz network kartının ismini öğrenmemiz gerekiyor. Aşağıdaki komutu çalıştırarak network kartlarının isimlerini görebilirsiniz. Ben "ens160" isimli kart için işlem yapacağım.
{% highlight c %}
ip addr show
{% endhighlight %}

![Picture description](/assets/img/posts/staticip/ip-addr.png){: .center-image }

Network kartının ismini öğrendikten sonra /etc/netplan dizinine gelmemiz gerekmektedir, bunun için aşağıdaki kodu çalıştırıyoruz.
{% highlight c %}
cd /etc/netplan
{% endhighlight %}

![Picture description](/assets/img/posts/staticip/cd-etc.png){: .center-image }

netplan klasörüne geldiğimizde ls komutu ile dizin içerisindeki dosyaları görebiliriz.
bu dosyalardan 01-netcfg.yaml dosyasını düzenlememiz gerekiyor, ben nano ile düzenleyeceğim. sizler vi vb editörleri tercih edebilirsiniz

{% highlight c %} 
sudo nano 01-netcfg.yaml
{% endhighlight %}

Bu dosyayı açtıktan sonra aşağıdaki görseldeki gibi bir ip tanımlaması yapabilirsiniz.

![Picture description](/assets/img/posts/staticip/01-netcfg.png){: .center-image }

Editörden çıktıtan sonra netplan apply yapmamız gerekiyor.

{% highlight c %} 
sudo nano netplan apply
{% endhighlight %}

![Picture description](/assets/img/posts/staticip/netcfg-apply.png){: .center-image }