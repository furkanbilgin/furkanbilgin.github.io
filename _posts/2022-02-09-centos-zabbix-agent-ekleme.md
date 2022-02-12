---
title: 'Centos Sunucu Üzerine Zabbix Agent Yüklemek'
date: 2022-02-09
author: Furkan
layout: post
permalink: /install-zabbix-agent-centos7/
categories: centos, rhel
tags: [centos ,rhel ,centos , linux, zabbix]
img: posts/install-zabbix-agent-centos7/cover.png
---

Bu öğreticide sizlerle Centos 7 işletim sistemine sahip bir sunucuda terminal üzerinden zabbix agent yükleyerek zabbix server ile nasıl takip edilebileceğini göreceğiz.

İşlemlere başlamadan önce sunucunuzun yedeğini almanızı tavsiye ederim. (snapshot, full backup vs.)

1- Adım: Repository Eklemesi

Öncelikle zabbix official repository sayfasına giderek zabbix server ile uyumlu sürümde olan zabbix agent repo linkini bulacağız.
https://repo.zabbix.com/zabbix/

Bizim linkimiz
https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm

Şimdi rpm repoyu sunucumuza ekleyelim.
{% highlight c %}
rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
{% endhighlight %}

2- Adım: Zabbix agent kurulumunu 

Bunun için aşağıdaki kodu yazmamız yeterli olacaktır. Kodun sonunda -y kurulum sırasında sorulacak sorulara otomatik bir şekilde onay verecektir.
{% highlight c %}
yum install zabbix zabbix-agent -y
{% endhighlight %}

3- Adım: Zabbix Agent Configurasyon Düzenlemesi

Bu işlem için aşağıdaki yolun altında bulunan zabbix agent configurasyon dosyasındaki ilgili 4 satırı değiştirmemiz gerekmektedir.
3.1- Zabbix Server İp Adresi : Server 

3.2- Dinlenilecek sunucunun ip adresi : ListenIp

3.3- ServerActive ip adresi : ServerActive

3.4- Dinlenilecek sunucu adı : Hostname

4- Adım: Firewall/Port Ayarları

Zabbix Server 10050 portu üzerinden agentları ile haberleşmektedir.
Bunun için aşağıdaki kodu çalıştırmanız yeterlidir.

{% highlight c %}
firewall-cmd --add-port=10050/tcp --permanent
firewall-cmd --reload
{% endhighlight %}

5- Adım: Zabbix Agent servisinin başlatılması
Bunun için öncelikle zabbix-agent servisini start ediyoruz, sonra durumuna bakıyoruz, akabinde sunucu reboot olduktan sonra tekrar çalışabimesi için enable ediyoruz.

{% highlight c %}
service zabbix-agent start
systemctl status zabbix-agent
systemctl enable zabbix-agent
{% endhighlight %}


<b>Not: Bazı kelime ve kavramları yanlış kullanmış olabilirim. Düzeltme için lütfen e-mail atınız.<b>