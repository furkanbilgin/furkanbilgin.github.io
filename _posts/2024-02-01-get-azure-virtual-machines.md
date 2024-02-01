---
title: 'Powershell ile Azuredaki Virtual Machine Listesini Çekme'
date: 2024-02-01T00:00:00+00:00
author: Furkan Bilgin
layout: post
permalink: /get-azure-virtual-machine-list-with-powershell/
categories: Azure
tags: [azure, cloud, vm, virtualmachine]
img: /assets/img/posts/get-azvm/microsoft-azure-virtual-machines.jpg
description: 'Powershell ile Azure üzerinde bulunan Storage Accountların TLS versionunu öğrenme'
---

Microsoft Azure'da bulunan virtual machinelere ihtiyacımız olduğunda aşağıdaki script ile hızlıca elde edebiliriz.

İşlem adımları şu şekildedir:

1- PowerShell aracılığıyla Azure'a bağlantı kurulur.
2- Azure üzerindeki abonelikler alınır.
3- Abonelikler içinde dolaşılarak Virtual Machine listesi çekilir.

{% highlight c %}

#login işlemini yapalım.
try {
    # YILDIZ * ile işaretlenmiş alana tenant ID yazınız.    
    Connect-AzAccount -Tenant ****-*****-***-******-****  
    #login olduktan sonra Subscriptionları çekelim.
    $AllSubscription = Get-AzSubscription       
}
catch {
    Write-Host  "Login veya Get-AzSubscription işlemi sırasında hata oluştu."
}

#Subscription arasında gezelim.
foreach ($Subscription  in $AllSubscription) {    
    
    #dizi içerisinden gelen Subscriptionı seçelim.
	Select-AzSubscription  -SubscriptionId $Subscription 
            
        #ilgili Subscriptiondaki tüm VirtualMachineleri bir diziye atalım.
        $AllVirtualMachines = Get-AzVM       
        
        #dizi içine attığımız virtual machineler içerisinde gezerken ihtiyacımız olan bilgileri alalım.
        foreach ($VirtualMachine in $AllVirtualMachines) {
            
            $VmName= $VirtualMachine.Name
            $Location = $VirtualMachine.Location.ToString()          
            $TimeCreated =  $VirtualMachine.TimeCreated.ToString()           
            $tags = $VirtualMachine.Tags | Out-String                 
            $OperatingSystem = $VirtualMachine.StorageProfile.OsDisk.OsType.ToString()
            
            try {
                $powerStateStatus= (Get-AzVM -ResourceGroupName  $VirtualMachine.ResourceGroupName -Name  $VmName -Status).VMAgent.Statuses.DisplayStatus
                $powerStateCode = (Get-AzVM -ResourceGroupName  $VirtualMachine.ResourceGroupName -Name  $VmName -Status).VMAgent.Statuses.Code
                $DisplayStatus = (Get-AzVM -ResourceGroupName  $VirtualMachine.ResourceGroupName -Name  $VmName -Status).Statuses.DisplayStatus          
            }
            catch {
                Write-Host  "Status sorgularını atarken hata alındı.."
            } 

            #değişkenlere atadığımız değerleri ekrana yazdıralım. 
            Write-Host " VmName: $VmName | Location: $Location | OperatingSystem: $OperatingSystem | powerState: $powerStateStatus | powerStateCode: $powerStateCode | DisplayStatus: $DisplayStatus "             
        } 
}

{% endhighlight %}

Örnek powershell çıktısı aşağıdaki şekildedir.

![Picture description](/assets/img/posts/get-azvm/example-screen-shot1.png){: .center-image }