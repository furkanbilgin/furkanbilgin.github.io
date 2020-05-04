---
title: 'Windows Server IIS Üzerinde Ücretsiz SSL Sertifikası Tanımlama'
date: 2020-05-02T00:00:00+00:00
author: Furkan
layout: post
permalink: /iis-uzerinde-ucretsiz-ssl-sertifikasi-tanimlama/
categories: Windows
tags: [windows, iis, ssl]
img: posts/ssl/win-acme.png
---


Bu öğreticide Win-Acme2 clientını kullanarak IIS üzerinde barındırdığımız web sitelerimiz için ücretsiz SSL sertifikası oluşturmayı göreceğiz.

Öncelikle kendi sitesinden uygulamanın son sürümünü indiriyoruz. </br><br>
<a href="https://www.win-acme.com/" target="_blank">win-acme web sitesi için tıklayınız.</a>

Uygulamayı indirdikten sonra sunucuda daha sonra silmeyeceğimiz veya yerini değiştirmeyeceğimiz bir yere koyuyuoruz.(Outo Renew için gerekiyor)
Klasör içerisinde bulunan wacs.exe uygulamasını "Administrator" yetkisi ile çalıştırıyoruz.

![Picture description](/assets/img/posts/ssl/win-acme-start.png){: .center-image }

Uygulamayı çalıştırdığımızda bize ana 6 seçenek sunuyor. 
Biz default seçenek olan "N: Create renewal (default settings)" seçeneğini seçiyoruz.
Bunun için dilerseniz hiç bir tuşa basmadan sadece enter'a basabilir veya N yazarak enter'a basabilirsiniz.

![Picture description](/assets/img/posts/ssl/win-acme-start.png){: .center-image }


Daha sonra bize IIS üzerinde barındırdığımız ssl sertifikası ekleyebileceğimiz siteleri listeliyor.
Ben SSL sertifikası eklemek istediğim sitenin sol tarafında olan IDye bakara ve terminale yazıyorum (15).

![Picture description](/assets/img/posts/ssl/id_list.png){: .center-image }

Gelen ekranda bize site içerisinde bulunan site bindings'lerden hangisine ssl eklemek istediğimizi soruyor.
Ben tüm bind'lar için https oluşturmak istiyorum ve 3'ü seçiyorum.

![Picture description](/assets/img/posts/ssl/set-site.png){: .center-image }

Şimdi bizden ana hostu seçmemizi istiyor. Ben 1'i seçiyorum.

![Picture description](/assets/img/posts/ssl/set-site.png){: .center-image }

Daha sonra gelen ekranda bize seçtiğimiz bindingi doğrulamamız istiyor "y" yazdıktan sonra enter'a basıyorum.

![Picture description](/assets/img/posts/ssl/set-ssl.png){: .center-image }

Eğer bu işlemi ilk defa yapıyorsanız olası durumlarda sizinle iletişime geçilmesi için bir eposta adresi talep edecektir.
ve işlem bu kadar. Şimdi ssl sertifikanızı kontrol edebilirsiniz.
https://www.sslshopper.com/ssl-checker.html

Not 1: 
Bu uygulama aynı zamanda zamanlanmış görevlere sertifika bitiş tarihi için yenileyici bir görev oluşturmaktadır.
Bu görevin çalışabilmesi için başta indirdiğimiz klasörün yerini değiştirmeyiniz.

Not 2:
Dilerseniz web.config dosyasından http üzerine gelen tüm istekleri https tarafına yönlendirmek için aşağıdaki kodu kullanabilirsiniz.

{% highlight c %}
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <clear />                
                <rule name="http_to_https" enabled="true" stopProcessing="true">
                    <match url="(.*)" />
                    <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
                        <add input="{HTTPS}" pattern="^OFF$" />
                    </conditions>
                    <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="false" />
                </rule>  
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
{% endhighlight %}

Yöntem 2  : 
Eğer bu işlemlerin tamamını bir arayüz ile yapmak isterseniz certifytheweb uygulamasını deneyebilirsiniz.
https://certifytheweb.com/

Certifytheweb için öğretici video 
<iframe width="560" height="315" src="https://www.youtube.com/embed/aoSUidj4ovA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
