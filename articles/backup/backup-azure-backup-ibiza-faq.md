---
title: "Recovery Services-Tresor – häufig gestellte Fragen | Microsoft Docs"
description: "Diese Version der häufig gestellten Fragen dient als Unterstützung für die öffentliche Vorschauversion des Azure Backup-Diensts. Hier finden Sie Antworten auf häufig gestellte Fragen zum Backup-Agent, zu Sicherung und Aufbewahrung, Wiederherstellung, Sicherheit und auf andere allgemeine Fragen zur Azure Backup-Lösung."
services: backup
documentationcenter: 
author: markgalioto
manager: carmonm
editor: 
keywords: "Backup-Lösung; Backup-Dienst"
ms.assetid: 5f55b500-1ee9-4f64-9306-02d6f7a8eded
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/21/2016
ms.author: markgal;trinadhk;
translationtype: Human Translation
ms.sourcegitcommit: e12d533ac5befe020c0aad4aa64ca9ed50349c3d
ms.openlocfilehash: 271f447a36f42330ed7b8dea78b32c06eebdeda6


---
# <a name="recovery-services-vault---faq"></a>Recovery Services-Tresor – häufig gestellte Fragen
Dieser Artikel enthält Informationen zum Recovery Services-Tresor und ergänzt die [häufig gestellten Fragen zu Azure Backup](backup-azure-backup-faq.md). Der Artikel „Azure Backup – Häufig gestellte Fragen“ enthält alle Fragen und Antworten zum Azure Backup-Dienst.  

Sie können im Disqus-Abschnitt dieses Artikels oder eines verwandten Artikels Fragen zu Azure Backup stellen. Außerdem können Sie Fragen zum Azure Backup-Dienst im [Diskussionsforum](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup)stellen.

## <a name="recovery-services-vaults-are-resource-manager-based-are-backup-vaults-classic-mode-still-supported-br"></a>Recovery Services-Tresor basieren auf Resource Manager. Werden Sicherungstresore (klassischer Modus) weiterhin unterstützt? <br/>
Ja. Backup-Tresore werden weiterhin unterstützt. Erstellen Sie Backup-Tresore im [klassischen Portal](https://manage.windowsazure.com). Erstellen Sie Recovery Services-Tresore im [Azure-Portal](https://portal.azure.com). Wir empfehlen Ihnen jedoch, Recovery Services-Tresore zu erstellen, da alle zukünftigen Verbesserungen nur für Recovery Services-Tresore verfügbar sind.

## <a name="can-i-migrate-a-backup-vault-to-a-recovery-services-vault-br"></a>Kann ich einen Backup-Tresor zu einem Recovery Services-Tresor migrieren? <br/>
Nein, leider nicht. Derzeit ist es nicht möglich, den Inhalt eines Backup-Tresors zu einem Recovery Services-Tresor zu migrieren. Wir arbeiten daran, diese Funktion hinzuzufügen, aber sie steht im Rahmen der öffentlichen Vorschau noch nicht zur Verfügung.

## <a name="do-recovery-services-vaults-support-classic-vms-or-resource-manager-based-vms-br"></a>Unterstützen Recovery Services-Tresore klassische virtuelle Computer oder Resource Manager-basierte virtuelle Computer? <br/>
Recovery Services-Tresore unterstützen beide Modelle.  Sie können einen im klassischen Portal erstellten virtuellen Computer (klassischer Modus) oder im Azure-Portal erstellte virtuelle Computer (Resource Manager-basiert) in einem Recovery Services-Tresor sichern.

## <a name="i-have-backed-up-my-classic-vms-in-backup-vault-now-i-want-to-migrate-my-vms-from-classic-mode-to-resource-manager-mode--how-can-i-backup-them-in-recovery-services-vault"></a>Ich habe meine klassischen virtuellen Computer in einem Sicherungstresor gesichert. Nun möchten ich meine virtuellen Computer vom klassischen Modus zum Resource Manager-Modus migrieren.  Wie kann ich sie in einem Recovery Services-Tresor sichern?
Sicherungen von klassischen virtuellen Computern in einem Sicherungstresor werden nicht automatisch zu Recovery Services-Tresoren migriert, wenn Sie die virtuellen Computer vom klassischen Modus zum Resource Manager-Modus migrieren. Führen Sie die folgenden Schritte aus, um Sicherungen virtueller Computer zu migrieren:

1. Wechseln Sie im Sicherungstresor zur Registerkarte **Geschützte Elemente** , und wählen Sie den virtuellen Computer aus. Klicken Sie auf [Schutz beenden](backup-azure-manage-vms-classic.md#stop-protecting-virtual-machines). Aktivieren Sie die Option *Zugeordnete Sicherungsdaten löschen***nicht**.
2. Migrieren Sie den virtuellen Computer vom klassischen Modus zum Resource Manager-Modus. Stellen Sie sicher, dass Speicher und Netzwerk, die dem virtuellen Computer entsprechen, auch zum Resource Manager-Modus migriert werden.
3. Erstellen Sie einen Recovery Services-Tresor, und konfigurieren Sie die Sicherung auf dem migrierten virtuellen Computer mithilfe der Aktion **Sicherung** oben auf dem Tresordashboard. Erfahren Sie mehr darüber, wie Sie [Sicherungen im Recovery Services-Tresor ermöglichen](backup-azure-vms-first-look-arm.md)



<!--HONumber=Feb17_HO3-->


