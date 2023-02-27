---
title: 'Ubuntu 20.04 Sunucu Üzerine Docker Compose ile Nginx Proxy Manager Kurulumu'
date: 2023-02-27
author: Furkan Bilgin
layout: post
permalink: /install-nginxproxymanager-withdockercompose/
categories: Nginx, Proxy, Manager, loadbalance, ubuntu, centos, rhel, docker, compose
tags: [Nginx, loadbalance, Proxy, Reverse Proxy, ubuntu, centos, rhel, linux]
img: /assets/img/posts/nginx-proxy-manager/cover.png
---

Bu öğreticide sizlerle Ubuntu 20.04 işletim sistemine sahip bir sunucuda docker compose ile Nginx Proxy Manager kurulumunu göreceğiz.

Kuruluma başlamadan önce sunucuda docker engine ve docker compose kurulu olması gerekmektedir.

Eğer kurulu değilse, kurulum yönergesi için <a href="https://docs.docker.com/engine/install/ubuntu/" target="_blank"> bu sayfayı ziyaret edebilirsiniz.</a>

İşlemlere başlamadan önce sunucunuzun yedeğini almanızı tavsiye ederim. (snapshot, full backup vs.)

**Proxy Server Nedir?**

Proxy sunucusu, client ile server arasında bir köprü görevi görür. 
Proxy serverlar kullanım amacınıza bağlı olarak değişen düzeylerde işlevsellik, güvenlik ve gizlilik sağlar.
Proxy server kullanıldığında servera gönderilen istekler proxy üzerinden geçerek servera ulaşır. Aynı şekilde dönen cevaplarda proxy üzerinden clienta döner. 


![Picture description](/assets/img/posts/nginx-proxy-manager/nginc.drawio.png){: .center-image }

**Nginx Nedir**

NGINX açık kaynak bir web sunucu yazılımıdır. Web sunucusunun yanında proxy server, reverse proxy server, cache, load balancer gibi özellikleri bulunmaktadır. <a href="https://www.nginx.com/resources/glossary/nginx/" target="_blank">Bakınız.</a>

**Nginx Proxy Manager Nedir**

Nginx proxy yöneticisi Docker üzerinde çalışan bir reverse proxy yönetim sistemidir. 

+ Konfigürasyonlar için bir web arayüzü vardır.
+ Nginx proxy manager ile ücretsiz let's encrypt ssl sertifikası kullanabilirsiniz.
+ Kullanıcı arayüzünden yapılan işlemleri auditlog altında görebilirsiniz.
+ Kolaylıkla yeni kullanıcı açabilir ve bu kullanıcılar için yetki tanımlaması yapabilirsiniz.

Artık kuruluma geçebiliriz.

**Nginx Proxy Manager Docker Compose Kurulumu**

Kuruluma başlamadan önce yedeğinizi mutlaka alınız. Öncelikle ben update işlemi yapacağım.

{% highlight c %}
sudo apt-get update -y
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/apt-get-update.png){: .center-image }

Şimdi docker uygulamalarımı koyduğum bir klasör açıyorum onunda içinde nginx proxy manager compose dosyasını koymak için **nginx-proxy-manager** isminde bir klasör oluşturuyorum ve oluşturduğum klasörün içine giriyorum.

{% highlight c %}
mkdir docker
cd docker
mkdir nginx-proxy-manager
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/mkdir-nginx.png){: .center-image }

Oluşturduğumuz klasörün içinde docker-compose.yml ismiyle bir dosya açıyoruz. Burada nginx proxy manager için gerekli docker compose tanımlamalarını yapacağız.

{% highlight c %}
nano docker-compose.yml
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/nano-docker-compose.png){: .center-image }

Aşağıdaki kodun tamamını dosyanın içine yapıştırınız. Boşluk ve syntax kaynaklı hatalar alabilirsiniz.
Güncel docker compose dosyasına <a href="https://nginxproxymanager.com/setup/" target="_blank"> bu linkten ulaşabilirsiniz.</a>  

{% highlight c %}

version: "3"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port      
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
      
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

{% endhighlight %}

Conf dosyamızı kayıt edip çıktıktan sonra docker compose up edebiliriz.

{% highlight c %}
docker compose up -d
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/docker-compose-up.png){: .center-image }

Kurulum bittikten sonra çalışan containerların listesini çekebilir ve durumlarını görebiliriz.

{% highlight c %}
docker ps
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/docker-ps.png){: .center-image }

Eğer container up olmazsa önce conf dosyasını kontrol edin bu yeterli olmazsa container loglarını görüntüleyin.

{% highlight c %}
docker compose logs
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/logs.png){: .center-image }

Şimdi sunucu ip adresimizin sonuna admin web portunu ekleyerek admin paneline girebiliriz.
Ben http://192.168.2.71:81 yazarak giriyorum. Defaul admin user bilgileri ile giriş yaptıktan sonra bu bilgileri değiştirmenizi isteyecektir.

{% highlight c %}
Email:    admin@example.com
Password: changeme
{% endhighlight %}

![Picture description](/assets/img/posts/nginx-proxy-manager/login-1.png){: .center-image }

Giriş yaptıktan hemen sonra admin **user email adresini değiştirmenizi** isteyecektir.

![Picture description](/assets/img/posts/nginx-proxy-manager/login-2.png){: .center-image }

Eposta adresini değiştirdikten sonra **defaul password** değişimini isteyecektir.

![Picture description](/assets/img/posts/nginx-proxy-manager/login-3.png){: .center-image }

Tüm bunları yaptıktan sonra herşey hazır hale gelmiştir.

![Picture description](/assets/img/posts/nginx-proxy-manager/homepage.png){: .center-image }

Resmi sitesinden örnek uygulamaları<a href="https://nginxproxymanager.com/screenshots/" target="_blank"> bu linke tıklayarak inceleyebilirsiniz.</a>  

<time datetime="{{ page.date}}">{{ page.date }}</time>



**Not: Bazı kelime ve kavramları yanlış kullanmış olabilirim. Düzeltme için lütfen <a href="mailto:furkanbilgin@windowslive.com" target="_blank">e-mail atınız</a>.**