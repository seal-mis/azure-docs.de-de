---
title: "Anzeigen und Verwalten von StorSimple Virtual Array-Aufträgen | Microsoft Docs"
description: "Es wird die Seite „Aufträge“ des StorSimple Manager-Diensts beschrieben, und Sie erfahren, wie Sie damit kürzlich durchgeführte und aktuelle Aufträge für das StorSimple Virtual Array nachverfolgen."
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: 
ms.assetid: 789f7e04-7b3f-456d-be69-37ea48446e9b
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 06/07/2016
ms.author: alkohli
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: d6a0d156e90bfbf16712f9093284e34edd493484


---
# <a name="use-the-storsimple-manager-service-to-view-jobs-for-the-storsimple-virtual-array"></a>Verwenden des StorSimple Manager-Diensts zum Anzeigen von Aufträgen für das StorSimple Virtual Array
## <a name="overview"></a>Übersicht
Die Seite **Aufträge** enthält ein zentrales Portal zum Anzeigen und Verwalten von Aufträgen auf Virtual Arrays (auch als lokale virtuelle Geräte bezeichnet), die über eine Verbindung mit Ihrem StorSimple Manager-Dienst verfügen. Sie können aktive, abgeschlossene und fehlgeschlagene Aufträge für mehrere virtuelle Geräte anzeigen. Die jeweiligen Ergebnisse werden in einem Tabellenformat angezeigt. 

![Seite „Aufträge“](./media/storsimple-ova-manage-jobs/ovajobs1.png)

Sie können die Aufträge, an denen Sie interessiert sind, schnell finden, indem Sie nach folgenden Feldern filtern:

* **Status** – Sie können nach allen, aktiven, abgeschlossenen oder fehlgeschlagenen Aufträgen suchen.
* **Von und Bis** – Aufträge können entsprechend einem Datums- und Zeitbereich gefiltert werden.
* **Typ** – Der Auftragstyp kann „Alle“, „Sicherung“, „Wiederherstellung“, „Failover“, „Updates herunterladen“ oder „Updates installieren“ lauten.
* **Geräte** – Aufträge werden für ein bestimmtes Gerät ausgelöst, das mit Ihrem Dienst verbunden ist. Die gefilterten Aufträge werden dann anhand der folgenden Attribute tabellarisch aufgelistet:
  
  * **Typ** – Der Auftragstyp kann „Alle“, „Sicherung“, „Wiederherstellung“, „Failover“, „Updates herunterladen“ oder „Updates installieren“ lauten.
  * **Status** – Der Status für Aufträge kann „Alle“, „Aktiv“, „Abgeschlossen“ oder „Fehler“ lauten.
  * **Entität** – Die Aufträge können einem Volume, einer Freigabe oder einem Gerät zugeordnet sein. 
  * **Gerät** – Der Name des Geräts, für den der Auftrag gestartet wurde.
  * **Gestartet am** – der Zeitpunkt, zu dem der Auftrag gestartet wurde.
  * **Status** – Der Prozentsatz, bis zu dem ein Auftrag abgearbeitet ist, der ausgeführt wird. Für einen abgeschlossenen Auftrag muss dieser Wert immer gleich 100 % sein.

Die Liste der Aufträge wird alle 30 Sekunden aktualisiert.

## <a name="view-job-details"></a>Anzeigen von Auftragsdetails
Führen Sie die folgenden Schritte aus, um die Details eines Auftrags anzuzeigen.

#### <a name="to-view-job-details"></a>So zeigen Sie Auftragsdetails an
1. Zeigen Sie auf der Seite **Aufträge** die Aufträge an, an denen Sie interessiert sind, indem Sie eine Abfrage mit entsprechenden Filtern ausführen. Sie können nach abgeschlossenen oder aktiven Aufträgen suchen.
2. Wählen Sie in der tabellarischen Auftragsliste einen Auftrag aus.
3. Klicken Sie unten auf der Seite auf **Details**.
4. Im Dialogfeld **Details** können Sie Status, Details und Zeitstatistiken anzeigen. Die folgende Abbildung zeigt ein Beispiel für das Dialogfeld **Auftragsdetails sichern** .
   
    ![Seite „Auftragsdetails“](./media/storsimple-ova-manage-jobs/ovajobs2.png)

#### <a name="job-failures-when-the-virtual-machine-is-paused-in-the-hypervisor"></a>Auftragsfehler, wenn der virtuelle Computer im Hypervisor angehalten wird
Wenn ein Auftrag im virtuellen StorSimple-Array ausgeführt wird und das Gerät (bereitgestellter virtueller Computer im Hypervisor) für mehr als 15 Minuten angehalten wurde, schlägt der Auftrag fehl. Dies ist darauf zurückzuführen, dass die Zeit Ihres virtuellen StorSimple-Arrays nicht mehr synchron mit der Microsoft Azure-Zeit ist. Ein Beispiel für einen Fehler des Wiederherstellungsauftrags wird im folgenden Screenshot gezeigt.

![Fehler bei Wiederherstellungsauftrag](./media/storsimple-ova-manage-jobs/restorejobfailure.png)

Diese Fehler betreffen Sicherungs-, Wiederherstellungs-, Aktualisierungs- und Failover-Aufträge. Wenn der virtuelle Computer in Hyper-V bereitgestellt wird, synchronisiert der Computer die Zeit letztendlich mit dem Hypervisor. Sobald dies geschieht, können Sie den Auftrag neu starten. 

## <a name="next-steps"></a>Nächste Schritte
[Erfahren Sie, wie Sie Ihr StorSimple Virtual Array über die lokale Webbenutzeroberfläche verwalten](storsimple-ova-web-ui-admin.md).




<!--HONumber=Nov16_HO3-->


