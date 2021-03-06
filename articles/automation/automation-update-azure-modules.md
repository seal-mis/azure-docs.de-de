---
title: Aktualisieren von Azure-Modulen in Azure Automation | Microsoft-Dokumentation
description: "In diesem Artikel wird beschrieben, wie Sie jetzt häufig verwendete Azure PowerShell-Module, die standardmäßig in Azure Automation bereitgestellt werden, aktualisieren können."
services: automation
documentationcenter: 
author: MGoedtel
manager: carmonm
editor: tysonn
ms.assetid: 
ms.service: automation
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/13/2017
ms.author: magoedte
translationtype: Human Translation
ms.sourcegitcommit: 7a2999b3b1a54668f6ef45433efabd5a495418fe
ms.openlocfilehash: ec84df70d4a77e3b81a88aa286fc492d92e3e753


---

# <a name="how-to-update-azure-powershell-modules-in-azure-automation"></a>Aktualisieren von Azure PowerShell-Modulen in Azure Automation

Die am häufigsten verwendeten Azure PowerShell-Module werden standardmäßig in jedem Automation-Konto bereitgestellt.  Das Azure-Team aktualisiert die Azure-Module regelmäßig. Über das Automation-Konto bieten wir Ihnen eine Möglichkeit, die Module im Konto zu aktualisieren, wenn neue Versionen verfügbar sind.

## <a name="updating-azure-modules"></a>Aktualisieren von Azure-Modulen

1. Auf dem Blatt „Module“ Ihres Automation-Kontos gibt es eine Option namens **Azure-Module aktualisieren**.  Sie ist immer aktiviert.<br><br> ![Option zum Aktualisieren von Azure-Modulen auf dem Blatt „Module“](media/automation-update-azure-modules/automation-update-azure-modules-option.png)

2. Klicken Sie auf **Azure-Module aktualisieren**. Dann wird eine Bestätigungsbenachrichtigung angezeigt, in der Sie gefragt werden, ob Sie den Vorgang fortsetzen möchten.<br><br> ![Benachrichtigung zum Aktualisieren von Azure-Modulen](media/automation-update-azure-modules/automation-update-azure-modules-notification.png)

3. Klicken Sie auf **Ja**, und der Aktualisierungsvorgang für die Module wird gestartet.  Der Aktualisierungsvorgang dauert etwa 15 bis 20 Minuten, und die folgenden Module werden aktualisiert:

  * Azure
  *    Azure.Storage
  *    AzureRm.Automation
  *    AzureRm.Compute
  *    AzureRm.Profile
  *    AzureRm.Resources
  *    AzureRm.Sql
  * AzureRm.Storage

    Wenn die Module bereits auf dem neuesten Stand sind, ist der Prozess in wenigen Sekunden abgeschlossen.  Wenn der Aktualisierungsvorgang abgeschlossen ist, werden Sie benachrichtigt.<br><br> ![Aktualisierungsstatus beim Aktualisieren der Azure-Module](media/automation-update-azure-modules/automation-update-azure-modules-updatestatus.png)

Im Rahmen des Aktualisierungsvorgangs werden die Zeitpläne für alle geplanten Runbooks aktualisiert, damit sie die neueste Version des Moduls verwenden.

Wenn Sie Cmdlets aus diesen Azure PowerShell-Modulen in Ihren Runbooks zum Verwalten von Azure-Ressourcen verwenden, dann sollten Sie diesen Aktualisierungsvorgang ungefähr jeden Monat ausführen, um sicherzustellen, dass Sie die neuesten Module haben.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu Integrationsmodulen und zum Erstellen benutzerdefinierter Module, um Automation weiter in andere Systeme, Dienste oder Lösungen zu integrieren, finden Sie unter [Integrationsmodule](automation-integration-modules.md).


<!--HONumber=Feb17_HO2-->


