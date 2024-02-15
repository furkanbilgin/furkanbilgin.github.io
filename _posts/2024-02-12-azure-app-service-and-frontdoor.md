---
title: 'Azure App Service ve Front Door ile Web Sitenizi Dünya ile Buluşturun'
date: 2024-02-15T00:00:00+00:00
author: Furkan Bilgin
layout: post
permalink: /azure-app-service-with-front-door/
categories: Azure, Cloud
tags: [azure, cloud, appservice, website]
img: /assets/img/posts/appservice/app-service.png
description: 'Azure App Service ve Front Door ile Web Sitenizi Dünya ile Buluşturun'
---

Bu makalede, Microsoft Azure Cloud ortamında Terraform kullanarak bir Web App oluşturacak ve ardından bu Web App önüne bir Front Door konumlandıracağız.

<h2>Azure Web App Nedir?</h2>

Windows veya Linux tabanlı ortamlarda çok kolay bir şekilde .NET, .NET Core, Java, Node.js, PHP ve Python gibi yazılım dillerinde geliştirilmiş web projelerinizi yayınlayabileceğiniz bir azure servisidir.
GitHub, Azure DevOps gibi CI/CD araçlarıyla entegre edilebilir, böylece devops süreçlerinizi yönetebilirsiniz.

Web App hakkında daha fazla teknik bilgi için <a href="https://learn.microsoft.com/tr-tr/azure/app-service/overview" target="_blank">buraya</a> göz atabilirsiniz.

Bu makalede Terraform kullanarak nasıl Web App ve Front Door oluşturacağımızı öğreneceğiz.

Terraform ile oluşturacağımız kaynaklar için terraform AZURERM Documentation bize yol gösterecektir. Bunun için aşağıdaki linki ziyaret edebilirsiniz.
<a href="https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/" target="_blank">Terraform documents </a>

App Service oluşturmak için gerekli olan resource tipleri aşağıdaki şekildedir. İhtiyaçlara göre ek resourcelar create etmek gerekebilir. 

<b>azurerm_service_plan </b> :  Bu resource içerisinde plana ait isim, planın azuredaki location bilgisi, os type, sku_name (hardware kapasitesi), worket count (kaç farklı instance) ve etiket gibi bilgiler belirtilmektedir.
	
<b>azurerm_windows_web_app </b>:   Azure Web App resource kendisini oluşturmak için kullanacağız. Burada Web App ait plan, hangi resource groupda olacağı, location bilgisi, eğer diğer azure resourceları ile görüşecekse yapılacak vnet integration, oluşturulacak site göre sipecific site configurasyonları, isterseniz web app erişim kısıtı koymak için ip restrictionlar bu resource'da tanımlanır.

<b>azurerm_application_insights </b>:  Aslında app insight ürünü web app'den bağımsız bir üründür. Web Appi gerçek zamanlı olarak izlemenizi sağlar. Bu hizmet, uygulamanızdaki performans sorunlarından performans metriclerine, oturum bilgilerinden ve diğer bir çok parametreyi otomatik olarak takip ederek size detaylı bir raporlama sağlar. Biz Web App ile App İnsights <a href=" https://learn.microsoft.com/tr-tr/azure/azure-monitor/app/app-insights-overview" target="_blank"> Bakınız </a>
	
<b>azurerm_app_service_custom_hostname_binding </b>:  App service oluştuğunda windows uzantılı bir domain oluşur. Bu domain ile erişmek yerine şirketimize ait domain ile erişmek için bu resource ayarlanır.
	
<b>azurerm_monitor_metric_alert </b>:  Portal üzerinde görünen Metric sekmesinde inceleyebildiğimiz metricleri alerte dönüştürebildiğimiz resourcedur.
Örnek olarak: Request count, private memory, response time, 4xx,5xx gibi durumlar için alarmlar tanımlanabilir.
    
<b>azurerm_private_endpoint </b>:   Eğer Web App dışarıya açılmak istenmiyorsa, local ortamlarınız ile azure arasında bir vpn varsa, private endpoint ekleyerek private endpoint ip adresi üzerinden local bağlantınız ile web app erişimi yapmanızı sağlar.

App Service oluşturduktan sonra Front Door oluşturmak için aşağıdaki resource tiplerine ihtiyacımız olacaktır. Yine ihtiyaçlara göre ek resourcelar create etmek gerekebilir. 

<h2>Peki Azure Front Door Nedir?</h2>

Microsoft Azure Front Door özellikle web uygulamalarımızı hızlı, güvenli ve ölçeklenebilir bir şekilde dağıtılmasını sağlamak için tasarlanmış bir hizmettir. 
Azure Front Door, kullanıcılara daha iyi performans, güvenlik ve yüksek erişilebilirlik sağlamaktadır.

Azure Front Door'un temel özellikleri şunlardır:

<b>Global Dağıtım:</b> Azure Front Door, dünya genelinde dağılmış bir ağ üzerinde hizmet vererek kullanıcılara en yakın konumdan içeriğe erişim sağlar. Bu, web uygulamalarının daha hızlı yüklenmesine olanak tanır.

<b>Yük Dengeleme:</b> Azure Front Door, gelen trafikleri yönetmek ve birden çok hedef sunucuya trafikleri dengeli bir şekilde yönlendirmek için yük dengeleme özelliklerine sahiptir. Bu sayede uygulamanın ölçeklenebilirliği artar ve yüksek erişilebilirlik sağlanır.

<b>Güvenlik:</b> Hizmet, DDoS saldırılarına karşı koruma ve Web Application Firewall (WAF) özellikleri ile geliştirilmiş güvenlik sağlar. Web uygulamalarınızı kötü amaçlı trafiğe karşı korur.

<b>SSL Termination:</b> Azure Front Door, SSL/TLS sertifikalarını yönetir. App service katmanında ayrıca ssl eklemenize gerek kalmaz.

Bu özellikler sayesinde, Azure Front Door, web uygulamalarınızı daha güvenli, hızlı ve ölçeklenebilir bir şekilde sunmanıza yardımcı olan bir hizmettir.
Daha fazla front door detayı için <a href="https://learn.microsoft.com/tr-tr/azure/frontdoor/front-door-overview" target="_blank">burayı inceleyiniz.</a>

![Picture description](/assets/img/posts/appservice/front-door-overview-expanded.png){: .center-image }

Şimdi front door için gerekli terraform resourcelarına bakalım.

<b>azurerm_cdn_frontdoor_profile </b>: Frontdoora ait plan burada belirtilir.  
    Detaylara <a href="https://azure.microsoft.com/tr-tr/pricing/details/frontdoor/" target="_blank">buradan bakınız</a>

<b>azurerm_cdn_frontdoor_origin_group</b>: Web veya diğer uygulamanızın originlerini barındıran ve hizmetinize erişim sağlayan origin gruplarını temsil eder.
	
<b>3- azurerm_cdn_frontdoor_endpoint</b> : Frontdoora ait bir adres tanımı verir, bu adresi kullanarak uygulamamıza erişebiliriz. Bu URL microsofta ait olan bir domaindendir. 

<b>azurerm_cdn_frontdoor_origin </b>:  Frontdoora gelen isteklerin yönlendirileceği origin  tanımı burada yapılır. Örnek vermek gerekirse bir web app veya function app burada origin olarak gösterilebilir.

<b>azurerm_cdn_frontdoor_rule_set </b> :  (zorunlu değil)  HTTP isteklerinin frontdoor'da nasıl işleneceğini özelleştirmenize olanak tanır ve web uygulamanızın davranışı üzerinde daha fazla kontrol sağlar.  Örnek vermek gerekirse 
		a. header şunu içeriyorsa şunu yap
		b. Request body şunu içeriyorsa şunu yap
		c. Hostname şuysa şunu yap vb....
	
<b>azurerm_cdn_frontdoor_route </b>:  Yukarıda oluşturduğumuz kaynakların deyim yerindeyse birbirine bağlandığı kaynaktır. Buradaki tanımlar sayesinde frontdoora gelen istekler origin gruplarına (originlere) gidecektir. Ayrıca bu resource altında cache mekanizmasını, hangi protocol'den istek kabul edeceğimiz gibi durumları yönetebilririz.

<b>azurerm_cdn_frontdoor_custom_domain </b>: Bu resource altında frontdoora bize ait olan bir custom domain ekleyebiliriz. 
        Örnek api.furkanbilgin.com

<b>azurerm_cdn_frontdoor_custom_domain_association </b>: Oluşturulan custom domaini route resource bağlarız. Eğer custom domain eklersek bu resourceda eklenmek zorundadır.
	

İlgili servislerin terraform tarafında oluşturacağımız kaynaklarından bahsettiğimize göre artık örnek proje repomuzu inceleyebiliriz
Kod tarafındaki detayları reponun ilgili klasörlerindeki readme file altında belirtilmiştir.
<a href="https://github.com/furkanbilgin/terraform/tree/main/appservice" target="_blank">repo adresi</a>

![Picture description](/assets/img/posts/appservice/app-service.png){: .center-image }