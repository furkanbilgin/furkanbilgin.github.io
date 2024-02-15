---
title: 'Azure App Service ile Web Site Yayınlama'
date: 2024-02-15T00:00:00+00:00
author: Furkan Bilgin
layout: post
permalink: /azure-app-service/
categories: Azure
tags: [azure, cloud, appservice, website]
img: /assets/img/posts/appservice/app-service.png
description: 'Azure App Service ile Web Site Yayınlama'
---

Bu makalemizde Microsoft Azure Cloud ortamında terraform ile bir Web App ve bu Web App önüne bir Front Door kurulumu yapacagız.

Azure Web App Nedir?

Windows veya Linux tabanlı ortamlarda çok kolay bir şekilde .NET, .NET Core, Java, Node.js, PHP ve Python gibi yazılım dillerinde geliştirilmiş web projelerinizi yayınlayabileceğiniz bir azure servisidir.

Github, Azure Devops gibi CI/CD toollarına Web App entegre ederek devops süreçlerinizi yönetebilirsiniz.

Azure Web App konusunda daha fazla teknik bilgi için aşağıdaki linki incelemenizi tavsiye ederim. Biz bu makalede terraform ile nasıl web app deploy edeceğimizi göreceğiz.
<a href="https://learn.microsoft.com/tr-tr/azure/app-service/overview" target="_blank">App Service Document</a>

Terraform ile oluşturacağımız kaynaklar için terraform AZURERM Documentation bize yol gösterecektir. Bunun için aşağıdaki linki ziyaret edebilirsiniz.
<a href="https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/ target="_blank">Terraform documents </a>



App Service oluşturmak için gerekli olan resource tipleri aşağıdaki şekildedir.

	1- azurerm_service_plan  :  Bu resource içerisinde plana ait isim, planın azuredaki location bilgisi, os type, sku_name (hardware kapasitesi), worket count (kaç farklı instance) ve etiket gibi bilgiler belirtilmektedir.
	
    2- azurerm_windows_web_app :   Azure Web App resource kendisini oluşturmak için kullanacağız. Burada Web App ait plan, hangi resource groupda olacağı, location bilgisi, eğer diğer azure resourceları ile görüşecekse yapılacak vnet integration, oluşturulacak site göre sipecific site configurasyonları, isterseniz web app erişim kısıtı koymak için ip restrictionlar bu resource'da tanımlanır.

	3- azurerm_application_insights :  Aslında app insight ürünü web app'den bağımsız bir üründür. Web Appi gerçek zamanlı olarak izlemenizi sağlar. Bu hizmet, uygulamanızdaki performans sorunlarından performans metriclerine, oturum bilgilerinden ve diğer bir çok parametreyi otomatik olarak takip ederek size detaylı bir raporlama sağlar. Biz Web App ile App İnsights <a href=" https://learn.microsoft.com/tr-tr/azure/azure-monitor/app/app-insights-overview" target="_blank"> Bakınız </a>
	
    4- azurerm_app_service_custom_hostname_binding :  App service oluştuğunda windows uzantılı bir domain oluşur. Bu domain ile erişmek yerine şirketimize ait domain ile erişmek için bu resource ayarlanır.
	
    5- azurerm_monitor_metric_alert :  Portal üzerinde görünen Metric sekmesinde inceleyebildiğimiz metricleri alerte dönüştürebildiğimiz resourcedur.
	Örnek olarak: Request count, private memory, response time, 4xx,5xx gibi durumlar için alarmlar tanımlanabilir.
    
    6- azurerm_private_endpoint :   Eğer Web App dışarıya açılmak istenmiyorsa, local ortamlarınız ile azure arasında bir vpn varsa, private endpoint ekleyerek private endpoint ip adresi üzerinden local bağlantınız ile web app erişimi yapmanızı sağlar.

App Service oluşturduktan sonra Front Door oluşturmak için aşağıdaki resource tiplerine ihtiyacımız olacaktır.

	1- azurerm_cdn_frontdoor_profile : Frontdoora ait plan burada belirtilir.  
    Detaylara <a href="https://azure.microsoft.com/tr-tr/pricing/details/frontdoor/" target="_blank">buradan bakınız</a>

	2- azurerm_cdn_frontdoor_origin_group: Web veya diğer uygulamanızın originlerini barındıran ve hizmetinize erişim sağlayan origin gruplarını temsil eder.
	
	3- azurerm_cdn_frontdoor_endpoint : Frontdoora ait bir adres tanımı verir, bu adresi kullanarak uygulamamıza erişebiliriz. Bu URL microsofta ait olan bir domaindendir. 

	4- azurerm_cdn_frontdoor_origin :  Frontdoora gelen isteklerin yönlendirileceği origin  tanımı burada yapılır. Örnek vermek gerekirse bir web app veya function app burada origin olarak gösterilebilir.

	5- azurerm_cdn_frontdoor_rule_set :  (zorunlu değil)  HTTP isteklerinin frontdoor'da nasıl işleneceğini özelleştirmenize olanak tanır ve web uygulamanızın davranışı üzerinde daha fazla kontrol sağlar.  Örnek vermek gerekirse 
		a. header şunu içeriyorsa şunu yap
		b. Request body şunu içeriyorsa şunu yap
		c. Hostname şuysa şunu yap vb....
	
	6- azurerm_cdn_frontdoor_route :  Yukarıda oluşturduğumuz kaynakların deyim yerindeyse birbirine bağlandığı kaynaktır. Buradaki tanımlar sayesinde frontdoora gelen istekler origin gruplarına (originlere) gidecektir. Ayrıca bu resource altında cache mekanizmasını, hangi protocol'den istek kabul edeceğimiz gibi durumları yönetebilririz.

	7- azurerm_cdn_frontdoor_custom_domain : Bu resource altında frontdoora bize ait olan bir custom domain ekleyebiliriz. 
        Örnek api.furkanbilgin.com

	8- azurerm_cdn_frontdoor_custom_domain_association : Oluşturulan custom domaini route resource bağlarız. Eğer custom domain eklersek bu resourceda eklenmek zorundadır.
	

İlgili servislerin terraform tarafında oluşturacağımız kaynaklarından bahsettiğimize göre artık kod tarafına geçebiliriz.
Her bir servisi bir module gibi planlayarak yapacağız.
Ana main tf dosyamızda alt klasörde oluşturduğumuz servis modüllerini çağıracağız.

İlgili örnek proje <a href="https://github.com/furkanbilgin/terraform/tree/main/appservice" target="_blank">repo adresi</a>

![Picture description](/assets/img/posts/appservice/app-service.png){: .center-image }