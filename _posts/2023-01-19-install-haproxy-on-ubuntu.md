---
title: 'Ubunut 20.04 Sunucu Üzerine HAProxy Kurulumu'
date: 2023-01-19
author: Furkan
layout: post
permalink: /install-haproxy-on-ubuntu/
categories: HAProxy, loadbalance, ubuntu, centos, rhel
tags: [HAProxy,loadbalance,ubuntu,centos ,rhel , linux]
img: posts/2023-01-19-ubuntu-HAProxy-kurulumu/cover.png
---

Bu öğreticide sizlerle Ubuntu 20.04 işletim sistemine sahip bir sunucuda terminal üzerinden HAProxy yükleyerek configurasyonları hakkında bilgi vereceğim.

İşlemlere başlamadan önce sunucunuzun yedeğini almanızı tavsiye ederim. (snapshot, full backup vs.)

**HAProxy Nedir?**

HAProxy, yüksek trafik alan veya yedeklilik sağlamak istediğiniz TCP ve HTTP protokolünü kullanan uygulamalarınız için yük dengeler ve proxy hizmeti sunar. 

HAProxy open source bir üründür. Ürünün enterprise versionu ise ücretlidir. Bu lisans kapsamında kurumsal destek almakla birlikte farklı özelliklerindende yararlanabilirsiniz.

Her zaman olduğu gibi  ürünün kendi resmi sitesine gidelim ve kurulum için gerekli komutları alalım
https://haproxy.debian.net/

Sunucuda ubuntunun hangi sürümünün kurulu olduğunu anlamak için aşağıdaki komutu çalıştırabilirsiniz.
![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/hostnamectl.png){: .center-image }

Ben ubuntu 20.04.5 üzerine bu kurulumu yaptığım tarihteki son HAProxy sürümünün kurulumunu yapacağım için seçimlerimi bu yönde yaptım.

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/command.png){: .center-image }

1- Adım: HAProxy yüklemek için sırasıyla aşağıdaki komutlarımızı çalıştıralım.

{% highlight c %}
apt-get install --no-install-recommends software-properties-common
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-noins-rec.png){: .center-image }

{% highlight c %}
add-apt-repository ppa:vbernat/haproxy-2.7
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-add-repo.png){: .center-image }

Aşağıdaki komutun sonuna -y parametresini eklerseniz benim gibi kurulum sırasında Y tuşuna basarak onay vermek zorunda kalmazsınız.

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

3- Adım: Kurulum bittiğine göre şimdi sunucu restart olduktan sonra HAProxy servisinin otomatik olarak yeniden başlaması için aşağıdaki komutu çalıştırıyoruz.

{% highlight c %}
systemctl enable haproxy
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-enable.png){: .center-image }

4- Adım: Haproxy ayarlamalarına geçmeden önce son olarak HAProxy durumunu sorgulamamız gerekiyor. Status çekerek servisin aktif olup olmadığını göreceğiz.

{% highlight c %}
systemctl status haproxy
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-status.png){: .center-image }

Şimdi haproxy ile proxy işleminin nasıl yapılacağını göreceğiz.
Buna başlamadan önce bazı kavramları açıklamak istiyorum.

**Global:**

Yapılandırma(haproxy.conf) dosyanızın en üstünde görünür. Kabul edilecek maksimum bağlantı sayısı, günlüklerin(logların) nereye iletileceği ve HAProxy Enterprise modüllerinin yükleneceği yol gibi yönergeleri tanımladığımız alandır.

**Default:**

Fronted, backend veya listen bölümünde ayarlayabileceğiniz parametlerin çoğunu buradan varsayılan olarak belirleyebilirsiniz. Daha sonra frontend veya backend kısmında bu ayarları tekrar belirtirseniz frontend veya backend altındaki belirttiğiniz parametreler aktif olmuş olur.

**Listen:**

Bu bölümde HAProxy istatisklerini görmek için tanımlama yapabiliriz.
Örnek görsel aşağıdaki şekildedir.

**Frontend:**

Bu alanda bir frontend oluştururuz ve gideceği backend serverını belirtiriz. Yine bu alanda hangi protokol ve port üzerinden iletişim kurulacağı belirtilir.

**Backend:**

Front end alanında oluşturulan frontendler için isteklerin yönlendirileceği serverlerı belirtiriz. Yine bu alanda hangi protokol ve port üzerinden iletişim kurulacağı belirtilir.

Önemli olduğunu düşündüğüm bu başlıkları açıkladıktan sonra şimdi örnek bir **conf file** düzenleyebiliriz.
Bunun için haproxy.cfg dosyasının olduğu yere gidelim.

{% highlight c %}
cd /etc/haproxy/
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/conf-file-location.png){: .center-image }

Şimdi conf dosyasını düzenlemek için nano ile içine giriyoruz.
Nano ile dosyayı düzeledikten sonra **CTRL + X** yapıp çıkarken **Y** tuşuna ve ardından **enter** tuşuna basıyoruz.

{% highlight c %}
sudo nano haproxy.cfg
{% endhighlight %}

Aşağıdaki örnekte # simgesinden sonra altındaki satırın hangi amaçlı kullanıldığını belirtmeye çalıştım.
Conf kopyala yapıştır ile alıp kendinize göre düzenleyebilirsiniz.

{% highlight c %}
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin

        # gelen isteklere 30s içinde server'dan cevap dönmezse istekler timeout alacaktır.
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull

        # load balance türü roundrobin olarak belirlendi.
        balance roundrobin
        timeout connect 5000
        timeout client  50000
        timeout server  50000

        # http status codelarına göre gösterilebilecek özelleştirebileceğiniz error sayfalarının lokasyonu. 
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen stats
        # istatisk ekranını için erişilecek adres tanımı
        bind 192.168.2.81:8080
        mode http

        # istatistik ekranını açıp kapatmak için düzenleme yapabilirsiniz.
        stats enable
        stats realm FURKAN

        # haproxy istatisklerini görebileceğiniz ekran url suffixi
        # ben bu şekilde bu ekrana erişeceğim >>> http://192.168.2.81:8080/stats 
        stats uri /stats
        # istatistik ekranına giriş için kullanıcıadı:parola bilgisi
        stats auth furkan:1234

frontend furkanbilgin_frontend        
        # hangi port üzerinden gelen isteklerin dinleneceğini söylüyoruz
        bind *:80

        # www.furkanbilgin.com için kayıt ekliyoruz.
        acl furkanbilgin_frontend  hdr(host) -i www.furkanbilgin.com

        # furkanbilgin.com için kayıt ekliyoruz.
        acl furkanbilgin_frontend  hdr(host) -i furkanbilgin.com

        # furkanbilgin_frontend için hangi backend configuration kullanacağını belirtiyoruz.
        use_backend furkanbilgin_backend if furkanbilgin_frontend

backend furkanbilgin_backend
        # f1 ibaresi istatistik ekranında ilgili servera isim vermek içindir.
        server f1 192.168.2.141:80

        # f2 ibaresi istatistik ekranında ilgili servera isim vermek içindir.
        server f2 192.168.2.96:80
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/edit-conf.png){: .center-image }

Conf dosyasını düzenledikten sonra aktif olması için haproxy servisini restart ediyoruz.
Eğer confda hata varsa restart işlemi sonrası hata alacaksınız.

{% highlight c %}
systemctl restart haproxy
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-restart.png){: .center-image }

Conf dosyası başarılı bir şekilde yapılandırıldı ve servis restart olduysa aşağıdaki komut ile servisin durumuna bakıp **running** olduğunu görmemiz gerekmektedir.

{% highlight c %}
systemctl status haproxy
{% endhighlight %}

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-status-conf.png){: .center-image }

İstatistik ekran görüntüsü

İlgili görselde backend sunuculara giden istekleri ve istekle ilgili detayları görebilirsiniz.

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-statistic.png){: .center-image }

HAProxy gelen isteklerin loglarına aşağıdaki yerden ulaşabilirsiniz.

{% highlight c %}
cd /var/log
{% endhighlight %}

Log dosyasındaki son 100 satırı görmek için aşağıdaki komutu çalıştırabilirsiniz.

{% highlight c %}
tail -n 100 /var/log/haproxy.log
{% endhighlight %}

Aşağıdaki görselde hangi isteğin hangi servera iletildiğini kırmızı ile daire içine aldım.

f1 ve f2 isimleri haproxy.cfg dosyası içinde backend serverlara verdiğimiz takma isimlerdir.

![Picture description](/assets/img/posts/2023-01-19-ubuntu-haproxy-kurulumu/ha-log.png){: .center-image }

**LAB Ortamı hakkında açıklama**

Bu anlatımı yapabilmek adına virtual box üzerinde 3 tane ubuntu 20.04 sanal sunucu kurdum.

192.168.2.81  : HAProxy server 

192.168.2.141 : furkanbilgin.com web sunucusu - 1

192.168.2.96  : furkanbilgin.com web sunucusu - 2

İnternet tarayıcısı üzerinden yaptığım isteklerin HAProxy gitmesi için bilgisayarımda C:\Windows\System32\drivers\etc\hosts dosyasına aşağıdaki bilgileri ekledim.

{% highlight c %}
192.168.2.81 furkanbilgin.com
192.168.2.81 www.furkanbilgin.com
{% endhighlight %}

Bu sayede istekler HAProxy üzerinden web sunucularına yönlendirilmiş oldu.

**Not: Bazı kelime ve kavramları yanlış kullanmış olabilirim. Düzeltme için lütfen <a href=" furkanbilgin@windowslive.com">e-mail atınız</a>.**
