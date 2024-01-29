---
title: 'Azuredaki Storage Accountların TLS Sürümlerini Powershell ile Denetleme Rehberi'
date: 2024-01-29T00:00:00+00:00
author: Furkan Bilgin
layout: post
permalink: /get-azure-storage-account-tls-version/
categories: Azure
tags: [azure, cloud, storage, tls]
img: /assets/img/posts/storage-account/storage-account.png
description: 'Powershell ile Azure üzerinde bulunan Storage Accountların TLS versionunu öğrenme'
---

Azure'daki Storage Account'ların TLS Sürümlerini Powershell ile Denetleme Rehberi

Bilindiği üzere 1 Kasım 2024 tarihi itibariyle, Microsoft Azure, TLS 1.0 ve 1.1 desteğini sonlandıracaktır. 

Bu nedenle, kullanıcıların mümkün olan en kısa sürede TLS 1.2'ye geçmeleri gerekmektedir.

Eğer bir hesapta birden fazla abonelik ve her abonelikte birden fazla bölgede kullanılan depolama hesapları bulunuyorsa, her bir depolama hesabının hangi TLS sürümünü kullandığını belirlemek oldukça zahmetli bir süreç olabilir.

Bu durumu daha etkili bir şekilde yönetebilmek adına, PowerShell kullanılarak Azure'a bağlantı kurulup basit bir script ile tüm aboneliklerdeki depolama hesapları taranarak gerekli bilgiler elde edilebilir.

İşlem adımları şu şekildedir:

1- PowerShell aracılığıyla Azure'a bağlantı kurulur.
2- Azure üzerindeki abonelikler alınır.
3- Abonelikler içinde dolaşılarak depolama hesapları çekilir.
4- Elde edilen depolama hesaplarının bilgileri çekilerek TLS sürümleri elde edilir..

{% highlight c %}

try {
    # YILDIZ * ile işaretlenmiş alana tenant ID yazınız.
    Connect-AzAccount -Tenant ****-*****-***-******-****    
    $AllSubscription = Get-AzSubscription       
}
catch {
    Write-Host  "Login veya Get-AzSubscription işlemi sırasında hata oluştu."
}

#login olduktan sonra Subscriptionlar arasında gezelim.
foreach ($Subscription in $AllSubscription) {    

    #gelen subscription seçerek bu subscription üzerinde işlem yapmasını sağlayalım
    Select-AzSubscription  -SubscriptionId $Subscription 
           
    try { 
        # İlgili subscription'da bulunan storage account bilgilerini çekelim.
        $storageAccounts = Get-AzStorageAccount            
    }
    catch { 
        Write-Host "Get-AzStorageAccount işlemi sırasında hata oluştu." 
    }

    # İlgili subscription'da bulunan storage accountlar içerisinde gezerek tek tek ihtiyacımız olan bilgileri elde edelim.
    foreach ($storageAccount in $storageAccounts) {
                
        $resourceGroupName = $storageAccount.ResourceGroupName
        $storageAccountName = $storageAccount.StorageAccountName               
        $tags = $storageAccount.Tags | Out-String                                 
        $tlsVersion = (Get-AzStorageAccount -Name $storageAccountName -ResourceGroupName  $resourceGroupName).MinimumTlsVersion
        $SubName = $Subscription.Name                                   

        Write-Host "Storage account: $storageAccountName TLS sürümü: $tlsVersion | SubscriptionName: $SubName | ResourceGroupName: $resourceGroupName"
    }
    
}

{% endhighlight %}

Örnek powershell çıktısı aşağıdaki şekildedir.

![Picture description](/assets/img/posts/storage-account/example-screen-shot.png){: .center-image }