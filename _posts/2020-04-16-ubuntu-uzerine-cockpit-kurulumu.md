---
title: 'Ubuntu Tabanlı Sistemlere Cockpit Kurulumu'
date: 2020-04-12T00:00:00+00:00
author: Furkan
layout: post
permalink: /ubuntu-uzerine-cockpit-kurulumu/
categories: Ubuntu
tags: [ubuntu, cockpit, web admin]
img: cockpit.png
---

Cockpit LINUX tabanlı sunucularımızı bir web tarayıcısı aracılığıyla kontrol etmemize yarayan basit ve kullanışlı bir yönetim panelidir.

Cockpit ile yapabileceğimiz temel bazı konular aşağıdaki şekildedir.

1- Sistem loglarını takip edebilirsiniz

2- Kullanıcı hesaplarını yönetebilirsiniz

3- Ağ, Disk, Cpu, Ram gibi kaynakları takip edebilirsiniz.

4- Kubernetes , Docker vb uygulamarı desteklemektedir.

5- Web arayüzünde terminal erişimi sunmaktadır.

Uygulamanın kendi web sitesinde detayları inceleye bilirsiniz.
 https://cockpit-project.org/


Kuruluma gelecek olursak.
Öncelikle sisteminizi update etmenizi öneririm.

Sisteminizi update etmek için aşağıdaki kodları çalıştırınız

Upgrade sonrası gerekliyse sunucunun yeniden başlaması için -y parametresini ekliyorum.

{% highlight c %}
sudo apt-get update

sudo apt-get upgrade -y
{% endhighlight %}

cockpit

Sunucumuzu güncelledikten sonra cockpit uygulamasını kurmak için aşağıdaki kodu çalıştırıyoruz.

Kurulum sırasında bizden y/n şeklinde onay isteyecektir.

y tuşuna basarak onay veriyoruz.
 
apt-get install cockpit
 

Kurulum başarılı bir şekilde tamamlandıktan sonra bir tarayıcı ile erişmek istiyorsanız cockpit socketini enable etmeniz gerekmektedir.
 
sudo systemctl enable --now cockpit.socket
 
Artık bir tarayıcı ile paneli keşfe çıkabilirsiniz.

Not: Google Chrome tarayısında sorun yaşayabilirsiniz, Firefox denemenizi öneririm.

 
