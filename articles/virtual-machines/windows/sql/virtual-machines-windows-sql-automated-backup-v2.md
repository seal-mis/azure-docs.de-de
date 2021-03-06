---
title: "Automatisierte Sicherung v2 für Azure Virtual Machines mit SQL Server 2016 | Microsoft-Dokumentation"
description: "Erläutert das Feature „Automatisierte Sicherung“ für virtuelle Computer mit SQL Server 2016 unter Azure. Dieser Artikel bezieht sich speziell auf virtuelle Computer mit dem Resource Manager-Modell."
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: jhubbard
editor: 
tags: azure-resource-manager
ms.assetid: ebd23868-821c-475b-b867-06d4a2e310c7
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/30/2017
ms.author: jroth
translationtype: Human Translation
ms.sourcegitcommit: 253c504fa433c7ca37c0065ebf01d13dafc76231
ms.openlocfilehash: 303d6768611fbe21ecf5a72a9e071436ad8b9cc9


---
# <a name="automated-backup-v2-for-sql-server-2016-azure-virtual-machines-resource-manager"></a>Automatisierte Sicherung v2 für Azure Virtual Machines mit SQL Server 2016 (Resource Manager)

Die automatisierte Sicherung v2 konfiguriert automatisch [Managed Backup für Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx) für alle vorhandenen und neuen Datenbanken auf einem virtuellen Azure-Computer, auf dem die Standard-, Enterprise- oder Developer-Edition von SQL Server 2016 ausgeführt wird. Dies bietet Ihnen die Möglichkeit, reguläre Datenbanksicherungen zu konfigurieren, die permanenten Azure-Blob-Speicher nutzen. Die automatisierte Sicherung v2 basiert auf der [SQL Server-Agent-Erweiterung für virtuelle SQL Server-Computer](virtual-machines-windows-sql-server-agent-extension.md).

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>Voraussetzungen
Um die automatisierte Sicherung v2 verwenden zu können, müssen die folgenden Voraussetzungen erfüllt sein:

**Betriebssystem:**

- Windows Server 2012 R2
- Windows Server 2016

**SQL Server-Version/-Edition:**

- SQL Server 2016 Standard
- SQL Server 2016 Enterprise
- SQL Server 2016 Developer

> [!IMPORTANT]
> Die automatisierte Sicherung v2 kann mit SQL Server 2016 verwendet werden. Wenn Sie SQL Server 2014 verwenden, können Sie Ihre Datenbanken mithilfe der automatisierten Sicherung v1 sichern. Weitere Informationen finden Sie unter [Automatisierte Sicherung für SQL Server 2014-VMs in Azure](virtual-machines-windows-sql-automated-backup.md).

**Datenbankkonfiguration**:

- Zieldatenbanken müssen das vollständige Wiederherstellungsmodell verwenden.
- Für Systemdatenbanken muss nicht das vollständige Wiederherstellungsmodell verwendet werden. Falls Sie allerdings Protokollsicherungen für das Modell oder für MSDB benötigen, müssen Sie das vollständige Wiederherstellungsmodell verwenden.

Weitere Informationen zu den Auswirkungen des vollständigen Wiederherstellungsmodells auf Sicherungen finden Sie unter [Sichern beim vollständigen Wiederherstellungsmodell](https://technet.microsoft.com/library/ms190217.aspx).

**Azure-Bereitstellungsmodell:**

- Resource Manager

> [!NOTE]
> Die automatisierte Sicherung basiert auf der **Erweiterung für den SQL Server-IaaS-Agent**. Aktuelle Katalogimages für virtuelle SQL-Computer fügen diese Erweiterung standardmäßig hinzu. Weitere Informationen finden Sie unter [Erweiterung für SQL Server-IaaS-Agent](virtual-machines-windows-sql-server-agent-extension.md).

## <a name="settings"></a>Einstellungen
In der folgenden Tabelle werden die Optionen beschrieben, die für die automatisierte Sicherung v2 konfiguriert werden können. Die tatsächlichen Konfigurationsschritte variieren abhängig davon, ob Sie das Azure-Portal oder Azure Windows PowerShell-Befehle verwenden.

### <a name="basic-settings"></a>Basic Settings

| Einstellung | Bereich (Standard) | Beschreibung |
| --- | --- | --- |
| **Automatisierte Sicherung** | Aktivieren/Deaktivieren (deaktiviert) | Aktiviert oder deaktiviert die automatisierte Sicherung v2 für einen virtuellen Azure-Computer mit SQL Server 2016 Standard oder Enterprise. |
| **Aufbewahrungszeitraum** | 1 bis&30; Tage (30 Tage) | Die Anzahl von Tagen, für die Sicherungen aufbewahrt werden. |
| **Speicherkonto** | Azure-Speicherkonto | Ein Azure-Speicherkonto, mit dem Dateien der automatisierten Sicherung im Blob-Speicher gespeichert werden. An diesem Speicherort wird ein Container zum Speichern aller Sicherungsdateien erstellt. Die Namenskonvention für die Sicherungsdatei enthält das Datum, die Uhrzeit und den Computernamen. |
| **Verschlüsselung** |Aktivieren/Deaktivieren (deaktiviert) | Aktiviert oder deaktiviert die Verschlüsselung. Bei aktivierter Verschlüsselung befinden sich die Zertifikate zum Wiederherstellen der Sicherung im angegebenen Speicherkonto im gleichen **automaticbackup**-Container und verwenden die gleiche Namenskonvention. Wenn das Kennwort geändert wird, wird ein neues Zertifikat mit diesem Kennwort generiert, das alte Zertifikat bleibt jedoch zum Wiederherstellen vorheriger Sicherungen erhalten. |
| **Kennwort** |Kennworttext | Ein Kennwort für Verschlüsselungsschlüssel. Ein Kennwort ist nur erforderlich, wenn die Verschlüsselung aktiviert ist. Um eine verschlüsselte Sicherung wiederherzustellen, benötigen Sie das richtige Kennwort und das zugehörige Zertifikat, das beim Erstellen der Sicherung verwendet wurde. |

### <a name="advanced-settings"></a>Erweiterte Einstellungen

| Einstellung | Bereich (Standard) | Beschreibung |
| --- | --- | --- |
| **System Database Backups** (Systemdatenbanksicherungen) | Aktivieren/Deaktivieren (deaktiviert) | Ist dieses Feature aktiviert, werden auch die Systemdatenbanken (Master, MSDB und Modell) gesichert. Vergewissern Sie sich bei der MSDB- und der Modelldatenbank, dass sie sich im vollständigen Wiederherstellungsmodus befinden, wenn Sie Protokollsicherungen erstellen möchten. Für die Masterdatenbank werden niemals Protokollsicherungen erstellt. Und für die TempDB-Datenbank werden gar keine Sicherungen ausgeführt. |
| **Sicherungszeitplan** | Manuell/Automatisiert (Automatisiert) | Der Sicherungszeitplan wird standardmäßig automatisch auf der Grundlage des Protokollwachstums bestimmt. Bei einem manuellen Sicherungszeitplan kann der Benutzer das gewünschte Zeitfenster für Sicherungen angeben. In diesem Fall werden Sicherungen ausschließlich gemäß dem angegebenen Intervall und während des angegebenen Zeitfensters eines bestimmten Tags durchgeführt. |
| **Full backup frequency** (Intervall für vollständige Sicherungen) | Täglich/Wöchentlich | Intervall für vollständige Sicherungen. In beiden Fällen werden vollständige Sicherungen während des nächsten geplanten Zeitfensters gestartet. Bei Verwendung der wöchentlichen Option können sich die Sicherungen über mehrere Tage erstrecken, bis alle Datenbanken erfolgreich gesichert wurden. |
| **Full backup start time** (Startzeit für vollständige Sicherungen) | 00:00–23:00 (01:00) | Die Startzeit eines bestimmten Tags, an dem eine vollständige Sicherung stattfinden kann. |
| **Full backup time window** (Zeitfenster für vollständige Sicherungen) | 1–23 Stunden (1 Stunde) | Das Zeitfenster eines bestimmten Tags, an dem eine vollständige Sicherung stattfinden kann. |
| **Log backup frequency** (Intervall für Protokollsicherungen) | 5–60 Minuten (60 Minuten) | Intervall für Protokollsicherungen. |

## <a name="understanding-full-backup-frequency"></a>Grundlegendes zum Intervall für vollständige Sicherungen
Es ist wichtig, den Unterschied zwischen täglichen und wöchentlichen vollständigen Sicherungen zu verstehen. Daher sehen wir uns im Anschluss zwei Beispielszenarien an.

### <a name="scenario-1-weekly-backups"></a>Szenario 1: Wöchentliche Sicherungen
Sie verfügen über einen virtuellen SQL Server-Computer mit einer Reihe sehr großer Datenbanken.

Am Montag aktivieren Sie die automatisierte Sicherung v2 mit den folgenden Einstellungen:

- Sicherungszeitplan: **Manuell**
- Intervall für vollständige Sicherungen: **Wöchentlich**
- Startzeit für vollständige Sicherungen: **01:00**
- Zeitfenster für vollständige Sicherungen: **1 Stunde**

Das nächste verfügbare Sicherungszeitfenster ist also am Dienstag ab 1 Uhr für eine Stunde. Zu dieser Zeit beginnt die automatisierte Sicherung damit, Ihre Datenbanken nacheinander zu sichern. In diesem Szenario sind Ihre Datenbanken so groß, dass die vollständige Sicherung nur für die ersten Datenbanken abgeschlossen werden kann. Nach einer Stunde wurden also noch nicht alle Datenbanken gesichert.

In diesem Fall beginnt die automatisierte Sicherung am nächsten Tag (Mittwoch, ab 1 Uhr für eine Stunde) damit, die restlichen Datenbanken zu sichern. Sollten dabei nicht alle Datenbanken gesichert werden können, wird am nächsten Tag zur gleichen Zeit ein erneuter Versuch gestartet. Dies wird fortgesetzt, bis alle Datenbanken erfolgreich gesichert wurden.

Am nächsten Dienstag beginnt die automatisierte Sicherung dann wieder mit der Sicherung aller Datenbanken.

Dieses Szenario zeigt, dass sich die Aktivität der automatisierten Sicherung auf das angegebene Zeitfenster beschränkt und jede Datenbank einmal pro Woche gesichert wird. Es zeigt auch, dass sich Sicherungen über mehrere Tage erstrecken können, falls nicht alle Sicherungen an einem einzelnen Tag abgeschlossen werden können.

### <a name="scenario-2-daily-backups"></a>Szenario 2: Tägliche Sicherungen
Sie verfügen über einen virtuellen SQL Server-Computer mit einer Reihe sehr großer Datenbanken.

Am Montag aktivieren Sie die automatisierte Sicherung v2 mit den folgenden Einstellungen:

- Sicherungszeitplan: Manuell
- Intervall für vollständige Sicherungen: Täglich
- Startzeit für vollständige Sicherungen: 22:00
- Zeitfenster für vollständige Sicherungen: 6 Stunden

Das nächste verfügbare Sicherungszeitfenster ist also am Montag ab 10 Uhr für sechs Stunden. Zu dieser Zeit beginnt die automatisierte Sicherung damit, Ihre Datenbanken nacheinander zu sichern.

Am Dienstag wird dann ab 10 Uhr erneut für sechs Stunden eine vollständige Sicherung aller Datenbanken durchgeführt.

> [!IMPORTANT]
> Bei der Planung täglicher Sicherungen empfiehlt sich die Verwendung eines großzügigen Zeitfensters, um sicherzustellen, dass alle Datenbanken gesichert werden können. Dies ist besonders wichtig, wenn große Datenmengen gesichert werden müssen.

## <a name="configuration-in-the-portal"></a>Konfiguration im Portal
Mit dem Azure-Portal können Sie die automatisierte Sicherung v2 während der Bereitstellung oder für vorhandene virtuelle SQL Server 2016-Computer konfigurieren. 

### <a name="new-vms"></a>Neue virtuelle Computer
Verwenden Sie das Azure-Portal zum Konfigurieren der automatisierten Sicherung v2, wenn Sie einen neuen virtuellen Computer mit SQL Server 2016 im Resource Manager-Bereitstellungsmodell erstellen. 

Wählen Sie auf dem Blatt **SQL Server-Einstellungen** die Option **Automatisierte Sicherung** aus. Auf dem folgenden Screenshot des Azure-Portals sehen Sie das Blatt **Automatisierte SQL-Sicherung** .

![Konfigurieren der automatisierten SQL-Sicherung im Azure-Portal](./media/virtual-machines-windows-sql-automated-backup-v2/automated-backup-blade.png)

> [!NOTE]
> Die automatisierte Sicherung v2 ist standardmäßig deaktiviert.

Wenn Sie weiteren Kontext benötigen, lesen Sie das vollständige Thema zum [Bereitstellen eines virtuellen Computers mit SQL Server in Azure](virtual-machines-windows-portal-sql-server-provision.md).

### <a name="existing-vms"></a>Vorhandene virtuelle Computer
Wählen Sie für vorhandene virtuelle Computer mit SQL Server Ihren virtuellen Computer mit SQL Server aus. Wählen Sie dann auf dem Blatt **Einstellungen** den Abschnitt **SQL Server-Konfiguration** aus.

![Automatisierte SQL-Sicherung für vorhandene virtuelle Computer](./media/virtual-machines-windows-sql-automated-backup-v2/sql-server-configuration.png)

Klicken Sie auf dem Blatt **SQL Server-Konfiguration** im Abschnitt für die automatisierte Sicherung auf die Schaltfläche **Bearbeiten**.

![Konfigurieren der automatisierten SQL-Sicherung für vorhandene virtuelle Computer](./media/virtual-machines-windows-sql-automated-backup-v2/sql-server-configuration-edit.png)

Klicken Sie abschließend unten auf dem Blatt **SQL Server-Konfiguration** auf die Schaltfläche **OK**, um die Änderungen zu speichern.

Falls Sie die automatisierte Sicherung zum ersten Mal aktivieren, konfiguriert Azure den SQL Server-IaaS-Agent im Hintergrund. Im Azure-Portal wird währenddessen u.U. nicht angezeigt, dass die automatisierte Sicherung konfiguriert wird. Warten Sie einige Minuten, bis der Agent installiert und konfiguriert wurde. Danach werden die neuen Einstellungen im Azure-Portal angezeigt.

## <a name="configuration-with-powershell"></a>Konfiguration mit PowerShell
Die automatisierte Sicherung v2 kann mithilfe von PowerShell konfiguriert werden. Führen Sie zur Vorbereitung folgende Schritte aus:

- [Laden Sie die aktuelle Version von Azure PowerShell herunter, und installieren Sie sie.](http://aka.ms/webpi-azps)
- Öffnen Sie Windows PowerShell, und ordnen Sie es Ihrem Konto zu. Befolgen Sie hierfür die Schritte im Abschnitt [Konfigurieren Ihres Abonnements](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-ps-sql-create#configure-your-subscription) des Bereitstellungsthemas.

### <a name="install-the-sql-iaas-extension"></a>Installieren der SQL-IaaS-Erweiterung
Wenn Sie einen virtuellen SQL Server-Computer über das Azure-Portal bereitgestellt haben, müsste die SQL Server-IaaS-Erweiterung bereits installiert sein. Rufen Sie den Befehl **Get-AzureRmVM** auf, und sehen Sie sich die Eigenschaft **Extensions** an, um zu ermitteln, ob sie für Ihren virtuellen Computer installiert ist.

```powershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"

(Get-AzureRmVM -Name $vmname -ResourceGroupName $resourcegroupname).Extensions 
```

Wenn die SQL Server-IaaS-Agent-Erweiterung installiert ist, wird „SqlIaaSAgent“ oder „SQLIaaSExtension“ aufgeführt. Außerdem muss für die Erweiterung unter **ProvisioningState** der Zustand „Succeeded“ angezeigt werden. 

Falls die Erweiterung nicht installiert oder nicht erfolgreich bereitgestellt wurde, können Sie sie mithilfe des folgenden Befehls installieren. Neben dem Namen des virtuellen Computers und der Ressourcengruppe müssen Sie auch die Region (**$region**) angeben, in der sich Ihr virtueller Computer befindet.

```powershell
$region = “EASTUS2”
Set-AzureRmVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "1.2" -Location $region 
```

### <a name="a-idverifysettings-verify-current-settings"></a><a id="verifysettings"> Überprüfen der aktuellen Einstellungen
Wenn Sie die automatisierte Sicherung während der Bereitstellung aktiviert haben, können Sie mithilfe von PowerShell Ihre aktuelle Konfiguration überprüfen. Führen Sie den Befehl **Get-AzureRmVMSqlServerExtension** aus, und sehen Sie sich die Eigenschaft **AutoBackupSettings** an:

```powershell
(Get-AzureRmVMSqlServerExtension -VMName $vmname -ResourceGroupName $resourcegroupname).AutoBackupSettings
```

Die Ausgabe sollte in etwa wie folgt aussehen:

```
Enable                      : True
EnableEncryption            : False
RetentionPeriod             : 30
StorageUrl                  : https://test.blob.core.windows.net/
StorageAccessKey            :  
Password                    : 
BackupSystemDbs             : False
BackupScheduleType          : Manual
FullBackupFrequency         : WEEKLY
FullBackupStartTime         : 2
FullBackupWindowHours       : 2
LogBackupFrequency          : 60
```

Falls Sie in der Ausgabe sehen, dass **Enable** auf **False** festgelegt ist, müssen Sie die automatisierte Sicherung aktivieren. Die gute Nachricht: Die Vorgehensweise zum Aktivieren der automatisierten Sicherung unterscheidet sich nicht von der Vorgehensweise zum Konfigurieren. Entsprechende Informationen finden Sie im nächsten Abschnitt.

> [!NOTE] 
> Wenn Sie eine Änderung vorgenommen haben und die Einstellungen direkt danach überprüfen, werden unter Umständen noch die alten Konfigurationswerte zurückgegeben. Warten Sie einige Minuten, und überprüfen Sie die Einstellungen dann erneut, um sich zu vergewissern, dass die Änderungen angewendet wurden.

### <a name="configure-automated-backup-v2"></a>Konfigurieren der automatisierten Sicherung v2
Mithilfe von PowerShell können Sie die automatisierte Sicherung nicht nur aktivieren, sondern auch jederzeit ihre Konfiguration und ihr Verhalten ändern. 

Wählen Sie zunächst ein Speicherkonto für die Sicherungsdateien aus, oder erstellen Sie eines. Das folgende Skript wählt ein Speicherkonto aus. Ist kein Speicherkonto vorhanden, wird eines erstellt.

```powershell
$storage_accountname = “yourstorageaccount”
$storage_resourcegroupname = $resourcegroupname

$storage = Get-AzureRmStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzureRmStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region } 
```

> [!NOTE]
> Die automatisierte Sicherung unterstützt zwar nicht das Speichern von Sicherungen in Storage Premium, kann aber Sicherungen von VM-Datenträgern erstellen, die Storage Premium verwenden.

Verwenden Sie dann den Befehl **New-AzureRmVMSqlServerAutoBackupConfig**, um die Einstellungen der automatisierten Sicherung v2 so zu konfigurieren, dass Sicherungen in dem Azure-Speicherkonto gespeichert werden. In diesem Beispiel ist für die Sicherungen eine Aufbewahrungsdauer von zehn Tagen festgelegt. Systemdatenbanksicherungen sind aktiviert. Vollständige Sicherungen werden wöchentlich ab 20:00 Uhr innerhalb eines zweistündigen Zeitfensters durchgeführt. Für Protokollsicherungen ist ein 30-minütiges Intervall geplant. Der zweite Befehl (**Set-AzureRmVMSqlServerExtension**) aktualisiert den angegebenen virtuellen Azure-Computer mit diesen Einstellungen.

```powershell
$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname 
```

Die Installation und Konfiguration des SQL Server-IaaS-Agents kann mehrere Minuten in Anspruch nehmen. 

Um die Verschlüsselung zu aktivieren, ändern Sie das vorherige Skript, um den **EnableEncryption**-Parameter und ein Kennwort (sichere Zeichenfolge) für den **CertificatePassword**-Parameter zu übergeben. Das folgende Skript aktiviert die Einstellungen der automatisierten Sicherung im vorherigen Beispiel und fügt die Verschlüsselung hinzu.

```powershell
$password = "P@ssw0rd"
$encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  

$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -Enable `
    -EnableEncryption -CertificatePassword $encryptionpassword `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

[Überprüfen Sie die Konfiguration der automatisierten Sicherung](#verifysettings), um sich zu vergewissern, dass Ihre Einstellungen angewendet wurden.

### <a name="disable-automated-backup"></a>Deaktivieren der automatisierten Sicherung
Führen Sie zum Deaktivieren der automatisierten Sicherung das gleiche Skript aus, lassen Sie beim Befehl **New-AzureRmVMSqlServerAutoBackupConfig** aber den Parameter **-Enable** weg. Das Fehlen des Parameters **-Enable** signalisiert dem Befehl, die Funktion zu deaktivieren. Ähnlich wie die Installation kann auch das Deaktivieren der automatisierten Sicherung mehrere Minuten dauern.

```powershell
$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -ResourceGroupName $storage_resourcegroupname

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

### <a name="example-script"></a>Beispielskript
Das folgende Skript stellt einen Satz von Variablen bereit, die Sie anpassen können, um die automatisierte Sicherung für Ihren virtuellen Computer zu aktivieren und zu konfigurieren. Unter Umständen müssen Sie das Skript an Ihre individuellen Anforderungen anpassen. So müssen Sie das Skript beispielsweise ändern, wenn Sie die Sicherung von Systemdatenbanken deaktivieren oder die Verschlüsselung aktivieren möchten.

```powershell
$vmname = "yourvmname"
$resourcegroupname = "vmresourcegroupname"
$region = “Azure region name such as EASTUS2”
$storage_accountname = “storageaccountname”
$storage_resourcegroupname = $resourcegroupname
$retentionperiod = 10
$backupscheduletype = "Manual"
$fullbackupfrequency = "Weekly"
$fullbackupstarthour = "20"
$fullbackupwindow = "2"
$logbackupfrequency = "30"

# ResourceGroupName is the resource group which is hosting the VM where you are deploying the SQL IaaS Extension 

Set-AzureRmVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "1.2" -Location $region

# Creates/use a storage account to store the backups

$storage = Get-AzureRmStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzureRmStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }

# Configure Automated Backup settings

$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays $retentionperiod -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType $backupscheduletype -FullBackupFrequency $fullbackupfrequency `
    -FullBackupStartHour $fullbackupstarthour -FullBackupWindowInHours $fullbackupwindow `
    -LogBackupFrequencyInMinutes $logbackupfrequency

# Apply the Automated Backup settings to the VM

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

## <a name="next-steps"></a>Nächste Schritte
Die automatisierte Sicherung v2 konfiguriert Managed Backup für virtuelle Azure-Computer. Lesen Sie daher unbedingt die [Dokumentation für Managed Backup](https://msdn.microsoft.com/library/dn449496.aspx) , um sich mit dem Verhalten und den Auswirkungen vertraut zu machen.

Weitere Informationen zur Sicherung und Wiederherstellung für SQL Server auf Azure-VMs finden Sie im folgenden Thema: [Sicherung und Wiederherstellung für SQL Server auf virtuellen Azure-Computern](virtual-machines-windows-sql-backup-recovery.md).

Informationen zu anderen verfügbaren Automatisierungsaufgaben finden Sie unter [SQL Server-Agent-Erweiterung für virtuelle SQL Server-Computer (klassisch)](virtual-machines-windows-sql-server-agent-extension.md).

Ausführlichere Informationen zur Ausführung von SQL Server auf virtuellen Azure-Computern finden Sie unter [Übersicht zu SQL Server auf virtuellen Azure-Computern](virtual-machines-windows-sql-server-iaas-overview.md).




<!--HONumber=Jan17_HO5-->


