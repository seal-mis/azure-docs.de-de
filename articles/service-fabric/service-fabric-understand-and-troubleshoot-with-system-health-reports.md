---
title: "Problembehandlung bei Systemintegritätsberichten | Microsoft Docs"
description: "Beschreibt die von Komponenten des Azure Fabric-Diensts versendeten Integritätsberichte und ihre Verwendung bei der Behandlung von Problemen mit Clustern oder Anwendungen."
services: service-fabric
documentationcenter: .net
author: oanapl
manager: timlt
editor: 
ms.assetid: 52574ea7-eb37-47e0-a20a-101539177625
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/12/2017
ms.author: oanapl
translationtype: Human Translation
ms.sourcegitcommit: d20b8d5848d1a11326c60d998099571a4ab8056e
ms.openlocfilehash: 0306b8c38a7dd86dff56f6cc7bb9eab7e0428762


---
# <a name="use-system-health-reports-to-troubleshoot"></a>Verwenden von Systemintegritätsberichten für die Problembehandlung
Für Azure Service Fabric-Komponenten werden für alle Entitäten im Cluster standardmäßig Berichte erstellt. Im [Integritätsspeicher](service-fabric-health-introduction.md#health-store) werden Entitäten basierend auf den Systemberichten erstellt und gelöscht. Darüber hinaus werden sie in einer Hierarchie organisiert, in der Interaktionen zwischen den Entitäten erfasst werden.

> [!NOTE]
> Informationen zu integritätsbezogenen Konzepten finden Sie unter [Einführung in die Service Fabric-Integritätsüberwachung](service-fabric-health-introduction.md).
> 
> 

Die Systemintegritätsberichte sorgen für Transparenz in Bezug auf die Cluster- und Anwendungsfunktionen und weisen auf Probleme mit der Integrität hin. Für Anwendungen und Dienste wird mit den Systemintegritätsberichten überprüft, ob Entitäten implementiert sind und sich aus Sicht von Service Fabric richtig verhalten. Die Berichte umfassen keine Integritätsüberwachung der Geschäftslogik des Diensts und keine Erkennung von hängenden Prozessen. Benutzerdienste können die Integritätsdaten um spezielle Informationen zu ihrer Logik erweitern.

> [!NOTE]
> Watchdog-Integritätsberichte werden erst angezeigt, *nachdem* die Systemkomponenten eine Entität erstellt haben. Wenn eine Entität gelöscht wird, werden vom Integritätsspeicher automatisch alle dazugehörigen Integritätsberichte gelöscht. Das gleiche gilt, wenn eine neue Instanz der Entität erstellt wird (z. B. eine neue Dienstreplikatinstanz). Alle Berichte, die der alten Instanz zugeordnet sind, werden gelöscht und im Speicher bereinigt.
> 
> 

Die Systemkomponentenberichte werden über die Quelle identifiziert. Diese beginnt mit dem „**System.**“- Präfix. Watchdogs können nicht das gleiche Präfix für ihre Quellen verwenden, da Berichte mit ungültigen Parametern abgelehnt werden.
Wir sehen uns nun einige Systemberichte an, um zu verstehen, wodurch sie ausgelöst werden und wie mögliche Probleme behoben werden, die damit verbunden sind.

> [!NOTE]
> Service Fabric fügt weiterhin Berichte für relevante Bedingungen hinzu, die zu einer besseren Transparenz in Bezug auf die Vorgänge im Cluster und in der Anwendung führen.
> 
> 

## <a name="cluster-system-health-reports"></a>Cluster-Systemintegritätsberichte
Die Entität für die Clusterintegrität wird im Integritätsspeicher automatisch erstellt. Wenn alles ordnungsgemäß funktioniert, liegt kein Systembericht vor.

### <a name="neighborhood-loss"></a>Verlust der Nachbarschaft
**System.Federation** meldet einen Fehler, wenn ein Verlust der „Nachbarschaft“ (Neighborhood) erkannt wird. Der Bericht stammt von einzelnen Knoten. Die Knoten-ID wird in den Eigenschaftsnamen eingefügt. Wenn eine Nachbarschaft im gesamten Service Fabric-Ring verloren geht, sind meist zwei Ereignisse zu erwarten. (Für beide Seiten der Lücke wird ein Bericht erstellt.) Falls weitere Nachbarschaften verloren gehen, ist die Anzahl von Ereignissen höher.

Im Bericht wird das Global Lease-Timeout als Gültigkeitsdauer (Time to Live, TTL) angegeben. Der Bericht wird jeweils zur Hälfte der Gültigkeitsdauer erneut gesendet, solange die Bedingung aktiv ist. Wenn das Ereignis abläuft, wird es automatisch entfernt. Durch dieses Verhalten ist selbst bei einem Ausfall des berichtenden Knotens eine ordnungsgemäße Bereinigung des Integritätsspeichers gewährleistet.

* **SourceId**: System.Federation
* **Property**: Beginnt mit **Neighborhood** und enthält Knoteninformationen.
* **Nächste Schritte**: Untersuchen Sie, warum die Nachbarschaft verloren geht (überprüfen Sie beispielsweise die Kommunikation zwischen Clusterknoten).

## <a name="node-system-health-reports"></a>Knoten-Systemintegritätsberichte
**System.FM**steht für den Failover-Manager-Dienst und ist die Autorität, mit der die Informationen zu Clusterknoten verwaltet werden. Jeder Knoten sollte über einen Bericht von System.FM verfügen, in dem der Zustand angegeben wird. Die Knotenentitäten werden entfernt, wenn der Knotenstatus entfernt wird (siehe [RemoveNodeStateAsync](https://msdn.microsoft.com/library/azure/mt161348.aspx)).

### <a name="node-updown"></a>Knoten heraufgefahren/heruntergefahren
System.FM meldet „OK“, wenn der Knoten dem Ring beitritt (betriebsbereit). Ein Fehler wird gemeldet, wenn der Knoten den Ring verlässt (nicht betriebsbereit, entweder aufgrund eines Upgrades oder eines Fehlers). Die vom Integritätsspeicher erstellte Integritätshierarchie wird für bereitgestellte Entitäten in Korrelation mit System.FM-Knotenberichten aktiv. Ein Knoten wird als virtuelles übergeordnetes Element aller bereitgestellten Entitäten angesehen. Die bereitgestellten Entitäten auf diesem Knoten werden über Abfragen verfügbar gemacht, wenn der Knoten von System.FM als aktiv gemeldet wird. Dabei wird die gleiche Instanz verwendet, die auch den Entitäten zugeordnet ist. Wenn System.FM meldet, dass der Knoten inaktiv ist oder neu gestartet wurde (neue Instanz), werden im Integritätsspeicher automatisch die bereitgestellten Entitäten bereinigt, die nur auf dem inaktiven Knoten oder der vorherigen Instanz des Knoten vorhanden sein können.

* **SourceId**: System.FM
* **Property**: State
* **Nächste Schritte**: Wenn der Knoten aufgrund eines Upgrades inaktiv ist, sollte er nach Abschluss des Upgrades wieder hochfahren. In diesem Fall sollte als Integritätsstatus wieder „OK“ gemeldet werden. Falls der Knoten nicht hochfährt oder ein Fehler auftritt, muss das Problem genauer untersucht werden.

Das folgende Beispiel zeigt das System.FM-Ereignis mit dem Integritätsstatus „OK“ für einen aktiven Knoten:

```powershell

PS C:\> Get-ServiceFabricNodeHealth -NodeName Node.1
NodeName              : Node.1
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 2
                        SentAt                : 4/24/2015 5:27:33 PM
                        ReceivedAt            : 4/24/2015 5:28:50 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 5:28:50 PM

```


### <a name="certificate-expiration"></a>Zertifikatablauf
**System.FabricNode** gibt eine Warnung aus, wenn vom Knoten verwendete Zertifikate kurz vor dem Ablauf stehen. Es gibt drei Zertifikate pro Knoten: **Certificate_cluster**, **Certificate_server** und **Certificate_default_client**. Wenn das Ablaufdatum noch mindestens zwei Wochen entfernt ist, lautet der Berichtsintegritätsstatus „OK“. Wenn der Ablauf in weniger als zwei Wochen erfolgt, wird als Berichtstyp eine Warnung erstellt. Die Gültigkeitsdauer (TTL) dieser Ereignisse ist unendlich. Sie werden entfernt, wenn ein Knoten den Cluster verlässt.

* **SourceId**: System.FabricNode
* **Property**: Beginnt mit **Certificate** und enthält weitere Informationen zum Zertifikattyp.
* **Nächste Schritte**: Aktualisieren Sie die Zertifikate, wenn sie in Kürze ablaufen.

### <a name="load-capacity-violation"></a>Verletzung der Ladekapazität
Der Service Fabric Load Balancer gibt eine Warnung aus, wenn eine Verletzung einer Knotenkapazität erkannt wird.

* **SourceId**: System.PLB
* **Property**: Beginnt mit **Capacity**.
* **Nächste Schritte**: Überprüfen Sie die bereitgestellten Metriken, und zeigen Sie die aktuelle Kapazität auf dem Knoten an.

## <a name="application-system-health-reports"></a>Systemintegritätsberichte für Anwendungen
**System.CM**steht für den Cluster-Manager-Dienst und ist die Autorität, die die Informationen zu einer Anwendung verwaltet.

### <a name="state"></a>Zustand
System.CM gibt die Meldung „OK“ aus, wenn die Anwendung erstellt oder aktualisiert wurde. Der Integritätsspeicher wird informiert, wenn die Anwendung gelöscht wurde, damit sie aus dem Speicher entfernt werden kann.

* **SourceId**: System.CM
* **Property**: State
* **Nächste Schritte**: Wenn die Anwendung erstellt wurde, sollte sie den Cluster-Manager-Integritätsbericht enthalten. Überprüfen Sie andernfalls den Zustand der Anwendung, indem Sie eine Abfrage durchführen (z.B. das PowerShell-Cmdlet **Get-ServiceFabricApplication -ApplicationName *applicationName***).

Das folgende Beispiel zeigt das Zustandsereignis für die Anwendung **fabric:/WordCount** :

```powershell
PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount -ServicesFilter None -DeployedApplicationsFilter None

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Ok
ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    :
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 82
                                  SentAt                : 4/24/2015 6:12:51 PM
                                  ReceivedAt            : 4/24/2015 6:12:51 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : ->Ok = 4/24/2015 6:12:51 PM
```

## <a name="service-system-health-reports"></a>Dienst-Systemintegritätsberichte
**System.FM**steht für den Failover-Manager-Dienst und ist die Autorität, die die Informationen zu Diensten verwaltet.

### <a name="state"></a>Zustand
System.FM gibt die Meldung „OK“ aus, wenn der Dienst erstellt wurde. Die Entität wird aus dem Integritätsspeicher gelöscht, wenn der Dienst gelöscht wurde.

* **SourceId**: System.FM
* **Property**: State

Das folgende Beispiel zeigt das Zustandsereignis für den Dienst **fabric:/WordCount/WordCountService**:

```powershell
PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountService

ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Ok
PartitionHealthStates :
                        PartitionId           : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
                        AggregatedHealthState : Ok

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 3
                        SentAt                : 4/24/2015 6:12:51 PM
                        ReceivedAt            : 4/24/2015 6:13:01 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 6:13:01 PM
```

### <a name="unplaced-replicas-violation"></a>Verletzung aufgrund von nicht platzierten Replikaten
**System.PLB** meldet eine Warnung, wenn keine Platzierung für ein oder mehrere Dienstreplikate gefunden werden kann. Der Bericht wird entfernt, wenn er abgelaufen ist.

* **SourceId**: System.FM
* **Property**: State
* **Nächste Schritte**: Überprüfen Sie die Diensteinschränkungen und den aktuellen Zustand der Platzierung.

Das folgende Beispiel zeigt einen Verstoß für einen Dienst, der mit sieben Zielreplikaten in einem Cluster mit fünf Knoten konfiguriert ist:

```xml
PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountService


ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.PLB',
                        Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                        ConsiderWarningAsError=false.

PartitionHealthStates :
                        PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        AggregatedHealthState : Warning

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 10
                        SentAt                : 3/22/2016 7:56:53 PM
                        ReceivedAt            : 3/22/2016 7:57:18 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        HealthState           : Warning
                        SequenceNumber        : 131032232425505477
                        SentAt                : 3/23/2016 4:14:02 PM
                        ReceivedAt            : 3/23/2016 4:14:03 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        fabric:/WordCount/WordCountService Secondary Partition a1f83a35-d6bf-4d39-b90d-28d15f39599b could not be placed, possibly,
                        due to the following constraints and properties:  
                        Placement Constraint: N/A
                        Depended Service: N/A

                        Constraint Elimination Sequence:
                        ReplicaExclusionStatic eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        ReplicaExclusionDynamic eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        Nodes Eliminated By Constraints:

                        ReplicaExclusionStatic:
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status:
                        None/None

                        ReplicaExclusionDynamic:
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status:
                        None/None


                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 3/22/2016 7:57:48 PM, LastOk = 1/1/0001 12:00:00 AM
```

## <a name="partition-system-health-reports"></a>Systemintegritätsberichte für Partitionen
**System.FM**steht für den Failover-Manager-Dienst und ist die Autorität, die die Informationen zu den Dienstpartitionen verwaltet.

### <a name="state"></a>Zustand
System.FM gibt die Meldung „OK“ aus, wenn die Partition erstellt wurde und fehlerfrei ist. Die Entität wird aus dem Integritätsspeicher gelöscht, wenn die Partition gelöscht wird.

Wenn die Replikatanzahl der Partition unterhalb des Mindestwerts liegt, wird ein Fehler gemeldet. Wenn der Mindestwert für die Replikatanzahl der Partition erfüllt ist, die Zielreplikatanzahl aber nicht, wird eine Warnung ausgegeben. Falls für die Partition ein Quorumverlust vorliegt, gibt System.FM einen Fehler aus.

Andere wichtige Ereignisse sind unter anderem eine Warnung, wenn die Neukonfiguration länger als erwartet dauert und wenn die Erstellung länger als erwartet dauert. Die erwarteten Zeiträume für die Erstellung und Neukonfiguration sind basierend auf Dienstszenarien konfigurierbar. Wenn ein Dienst beispielsweise über Zustandsdaten im Terabytebereich verfügt (etwa SQL-Datenbank), dauert die Erstellung länger als bei einem Dienst mit einer geringeren Menge an Zustandsdaten.

* **SourceId**: System.FM
* **Property**: State
* **Nächste Schritte**: Wenn der Integritätsstatus nicht „OK“ lautet, ist es möglich, dass einige Replikate nicht richtig erstellt, geöffnet oder auf primäre oder sekundäre Elemente heraufgestuft wurden. Häufig ist die Hauptursache ein Dienstfehler bei der Implementierung zum Öffnen oder Ändern von Rollen.

Das folgende Beispiel zeigt eine fehlerfreie Partition:

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/StatelessPiApplication/StatelessPiService | Get-ServiceFabricPartitionHealth
PartitionId           : 29da484c-2c08-40c5-b5d9-03774af9a9bf
AggregatedHealthState : Ok
ReplicaHealthStates   : None
HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 38
                        SentAt                : 4/24/2015 6:33:10 PM
                        ReceivedAt            : 4/24/2015 6:33:31 PM
                        TTL                   : Infinite
                        Description           : Partition is healthy.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 6:33:31 PM
```

Das folgende Beispiel zeigt die Integrität einer Partition, bei der die Zielreplikatanzahl nicht erreicht wird. Anschließend rufen Sie die Partitionsbeschreibung ab, in der ihre Konfiguration angegeben ist: **MinReplicaSetSize** entspricht zwei und **TargetReplicaSetSize** entspricht sieben Replikaten. Geben Sie anschließend die Anzahl der Knoten im Cluster an: fünf. In diesem Fall können zwei Replikate also nicht platziert werden.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None

PartitionId           : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   : None
HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 37
                        SentAt                : 4/24/2015 6:13:12 PM
                        ReceivedAt            : 4/24/2015 6:13:31 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 4/24/2015 6:13:31 PM

PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService

PartitionId            : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
PartitionKind          : Int64Range
PartitionLowKey        : 1
PartitionHighKey       : 26
PartitionStatus        : Ready
LastQuorumLossDuration : 00:00:00
MinReplicaSetSize      : 2
TargetReplicaSetSize   : 7
HealthState            : Warning
DataLossNumber         : 130743727710830900
ConfigurationNumber    : 8589934592


PS C:\> @(Get-ServiceFabricNode).Count
5
```

### <a name="replica-constraint-violation"></a>Verletzung der Replikateinschränkung
**System.PLB** gibt eine Warnung aus, wenn eine Verletzung der Replikateinschränkung erkannt wird und Replikate der Partition nicht platziert werden können.

* **SourceId**: System.PLB
* **Property**: Beginnt mit **ReplicaConstraintViolation**.

## <a name="replica-system-health-reports"></a>Systemintegritätsberichte für Replikate
**System.RA**steht für die Reconfiguration Agent-Komponente und ist die Autorität für den Replikatzustand.

### <a name="state"></a>Zustand
**System.RA** gibt die Meldung „OK“ aus, wenn das Replikat erstellt wurde.

* **SourceId**: System.RA
* **Property**: State

Das folgende Beispiel zeigt ein fehlerfreies Replikat:

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth
PartitionId           : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
ReplicaId             : 130743727717237310
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 130743727718018580
                        SentAt                : 4/24/2015 6:12:51 PM
                        ReceivedAt            : 4/24/2015 6:13:02 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 6:13:02 PM
```

### <a name="replica-open-status"></a>Öffnungszustand des Replikats
Die Beschreibung dieses Integritätsberichts enthält den Startzeitpunkt (koordinierte Weltzeit), zu dem der API-Aufruf erfolgt ist.

**System.RA** gibt eine Warnung aus, wenn das Öffnen des Replikats länger länger dauert, als die Konfiguration für den Zeitraum vorgibt (Standardwert: 30 Minuten). Wenn die API sich auf die Dienstverfügbarkeit auswirkt, wird der Bericht deutlich schneller ausgegeben (konfigurierbares Intervall, Standardwert: 30 Sekunden). Die gemessene Zeit enthält auch die Zeit für das Öffnen des Replikators und des Diensts. Die Eigenschaft ändert sich in „OK“, wenn das Öffnen abgeschlossen ist.

* **SourceId**: System.RA
* **Property**: **ReplicaOpenStatus**
* **Nächste Schritte**: Wenn der Integritätsstatus nicht „OK“ lautet, sollten Sie untersuchen, warum das Öffnen des Replikats länger dauert als erwartet.

### <a name="slow-service-api-call"></a>Langsamer Dienst-API-Aufruf
**System.RAP** und **System.Replicator** geben eine Warnung aus, wenn ein Aufruf des Benutzerdienstcodes länger dauert als in der Konfiguration angegeben. Die Warnung wird gelöscht, wenn der Aufruf abgeschlossen ist.

* **SourceId**: System.RAP oder System.Replicator
* **Property**: Name der langsamen API. Die Beschreibung liefert weitere Details dazu, wie lange die API bereits aussteht.
* **Nächste Schritte**: Untersuchen Sie, warum der Aufruf länger dauert als erwartet.

Das folgende Beispiel zeigt eine Partition mit Quorumverlust und die Schritte zur Ermittlung der Ursache. Eines der Replikate weist den Integritätsstatus „Warning“ auf, sodass Sie über die Integrität informiert sind. Es wird angezeigt, dass der Dienstvorgang länger als erwartet dauert. Das Ereignis wird von System.RAP gemeldet. Wenn Sie diese Informationen erhalten haben, besteht der nächste Schritt darin, sich den Dienstcode anzusehen und ihn zu untersuchen. Für diesen Fall löst die **RunAsync**-Implementierung des zustandsbehafteten Diensts einen Ausnahmefehler aus. Da die Replikate recycelt werden, sind unter Umständen keine Replikate mit dem Zustand „Warning“ zu sehen. Sie können den Integritätsstatus erneut abrufen und die Replikat-ID auf Unterschiede überprüfen. In bestimmten Fällen können die erneuten Versuche aufschlussreich sein.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/HelloWorldStatefulApplication/HelloWorldStateful | Get-ServiceFabricPartitionHealth

PartitionId           : 72a0fb3e-53ec-44f2-9983-2f272aca3e38
AggregatedHealthState : Error
UnhealthyEvaluations  :
                        Error event: SourceId='System.FM', Property='State'.

ReplicaHealthStates   :
                        ReplicaId             : 130743748372546446
                        AggregatedHealthState : Ok

                        ReplicaId             : 130743746168084332
                        AggregatedHealthState : Ok

                        ReplicaId             : 130743746195428808
                        AggregatedHealthState : Warning

                        ReplicaId             : 130743746195428807
                        AggregatedHealthState : Ok

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Error
                        SequenceNumber        : 182
                        SentAt                : 4/24/2015 7:00:17 PM
                        ReceivedAt            : 4/24/2015 7:00:31 PM
                        TTL                   : Infinite
                        Description           : Partition is in quorum loss.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Warning->Error = 4/24/2015 6:51:31 PM

PS C:\> Get-ServiceFabricPartition fabric:/HelloWorldStatefulApplication/HelloWorldStateful

PartitionId            : 72a0fb3e-53ec-44f2-9983-2f272aca3e38
PartitionKind          : Int64Range
PartitionLowKey        : -9223372036854775808
PartitionHighKey       : 9223372036854775807
PartitionStatus        : InQuorumLoss
LastQuorumLossDuration : 00:00:13
MinReplicaSetSize      : 2
TargetReplicaSetSize   : 3
HealthState            : Error
DataLossNumber         : 130743746152927699
ConfigurationNumber    : 227633266688

PS C:\> Get-ServiceFabricReplica 72a0fb3e-53ec-44f2-9983-2f272aca3e38 130743746195428808

ReplicaId           : 130743746195428808
ReplicaAddress      : PartitionId: 72a0fb3e-53ec-44f2-9983-2f272aca3e38, ReplicaId: 130743746195428808
ReplicaRole         : Primary
NodeName            : Node.3
ReplicaStatus       : Ready
LastInBuildDuration : 00:00:01
HealthState         : Warning

PS C:\> Get-ServiceFabricReplicaHealth 72a0fb3e-53ec-44f2-9983-2f272aca3e38 130743746195428808

PartitionId           : 72a0fb3e-53ec-44f2-9983-2f272aca3e38
ReplicaId             : 130743746195428808
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.RAP', Property='ServiceOpenOperationDuration', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 130743756170185892
                        SentAt                : 4/24/2015 7:00:17 PM
                        ReceivedAt            : 4/24/2015 7:00:33 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 7:00:33 PM

                        SourceId              : System.RAP
                        Property              : ServiceOpenOperationDuration
                        HealthState           : Warning
                        SequenceNumber        : 130743756399407044
                        SentAt                : 4/24/2015 7:00:39 PM
                        ReceivedAt            : 4/24/2015 7:00:59 PM
                        TTL                   : Infinite
                        Description           : Start Time (UTC): 2015-04-24 19:00:17.019
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Warning = 4/24/2015 7:00:59 PM
```

Wenn Sie die fehlerhafte Anwendung mit aktiviertem Debugger starten, wird in den Fenstern mit den Diagnoseereignissen die von RunAsync ausgelöste Ausnahme angezeigt:

![Visual Studio 2015-Diagnoseereignisse: RunAsync-Fehler in „fabric:/HelloWorldStatefulApplication“.][1]

Visual Studio 2015-Diagnoseereignisse: RunAsync-Fehler in **fabric:/HelloWorldStatefulApplication**.

[1]: ./media/service-fabric-understand-and-troubleshoot-with-system-health-reports/servicefabric-health-vs-runasync-exception.png


### <a name="replication-queue-full"></a>Replikationswarteschlange ist voll
**System.Replicator** gibt eine Warnung aus, wenn die Replikationswarteschlange voll ist. Beim primären Element passiert dies in der Regel, weil sekundäre Replikate beim Bestätigen von Vorgängen sehr langsam sind. Beim sekundären Element geschieht dies gewöhnlich, wenn der Dienst langsam beim Anwenden der Vorgänge ist. Die Warnung wird gelöscht, wenn die Warteschlange nicht mehr voll ist.

* **SourceId**: System.Replicator
* **Property**: **PrimaryReplicationQueueStatus** oder **SecondaryReplicationQueueStatus**, je nach Replikatrolle.

### <a name="slow-naming-operations"></a>Langsame Naming-Vorgänge
**System.NamingService** liefert Informationen zur Integrität des entsprechenden primären Replikats, wenn ein Naming-Vorgang zu lang dauert. Beispiele für Naming-Vorgänge sind [CreateServiceAsync](https://msdn.microsoft.com/library/azure/mt124028.aspx) und [DeleteServiceAsync](https://msdn.microsoft.com/library/azure/mt124029.aspx). Weitere Methoden finden Sie unter FabricClient (beispielsweise unter [Dienstverwaltungsmethoden](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.servicemanagementclient.aspx) sowie unter [Eigenschaftsverwaltungsmethoden](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.propertymanagementclient.aspx)).

> [!NOTE]
> Der Naming-Dienst löst die Dienstnamen in einen Speicherort im Cluster auf und ermöglicht Benutzern, Dienstnamen und -eigenschaften zu verwalten. Hierbei handelt es sich um einen partitionierten, persistenten Service Fabric-Dienst. Eine der Partitionen stellt den Autoritätsbesitzer dar, der Metadaten zu allen Service Fabric-Namen und -Diensten enthält. Die Service Fabric-Namen werden verschiedenen Partitionen (so genannten Namensbesitzerpartitionen) zugeordnet, um die Erweiterung des Diensts zu ermöglichen. Weitere Informationen finden Sie unter [Service Fabric-Architektur](service-fabric-architecture.md).
> 
> 

Wenn ein Naming-Vorgang unerwartet lang dauert, wird der Vorgang im *primären Replikat der Naming-Dienstpartition, die den Vorgang abwickelt*, mit einem Warnungsbericht gekennzeichnet. Ist der Vorgang erfolgreich, wird die Warnung gelöscht. Wird der Vorgang mit einem Fehler abgeschlossen, enthält der Integritätsbericht Einzelheiten zu dem Fehler.

* **SourceId**: System.NamingService
* **Property**: Beginnt mit dem Präfix **Duration_** und identifiziert den langsamen Vorgang und den Service Fabric-Namen, auf den der Vorgang angewendet wird. Ein Beispiel: Wenn die Diensterstellung für den Namen „fabric:/MyApp/MyService“ zu lang dauert, lautet die Eigenschaft „Duration_AOCreateService.fabric:/MyApp/MyService“. AO verweist auf die Rolle der Naming-Partition für diesen Namen und Vorgang.
* **Nächste Schritte**: Überprüfen Sie, warum der Naming-Vorgang nicht erfolgreich ist. Bei jedem Vorgang können andere Ursachen vorliegen. So kann beispielsweise auf einem Knoten ein Problem mit dem Befehl zum Löschen des Diensts vorliegen, da der Anwendungshost auf einem Knoten aufgrund eines Benutzerfehlers im Dienstcode immer wieder abstürzt.

Im Anschluss sehen Sie ein Beispiel für einen Diensterstellungsvorgang. Der Vorgang dauerte länger als in der Konfiguration festgelegt. AO wiederholt den Vorgang und sendet Arbeit an NO. NO hat den letzten Vorgang mit Timeout abgeschlossen. In diesem Fall wird sowohl für die AO- als auch für die NO-Rolle das gleiche Replikat als primäres Replikat verwendet.

```powershell
PartitionId           : 00000000-0000-0000-0000-000000001000
ReplicaId             : 131064359253133577
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.NamingService', Property='Duration_AOCreateService.fabric:/MyApp/MyService', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131064359308715535
                        SentAt                : 4/29/2016 8:38:50 PM
                        ReceivedAt            : 4/29/2016 8:39:08 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 4/29/2016 8:39:08 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_AOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064359526778775
                        SentAt                : 4/29/2016 8:39:12 PM
                        ReceivedAt            : 4/29/2016 8:39:38 PM
                        TTL                   : 00:05:00
                        Description           : The AOCreateService started at 2016-04-29 20:39:08.677 is taking longer than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_NOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064360657607311
                        SentAt                : 4/29/2016 8:41:05 PM
                        ReceivedAt            : 4/29/2016 8:41:08 PM
                        TTL                   : 00:00:15
                        Description           : The NOCreateService started at 2016-04-29 20:39:08.689 completed with FABRIC_E_TIMEOUT in more than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM
```

## <a name="deployedapplication-system-health-reports"></a>DeployedApplication-Systemintegritätsberichte
**System.Hosting** ist die Autorität für bereitgestellte Entitäten.

### <a name="activation"></a>Aktivierung
System.Hosting gibt die Meldung „OK“ aus, wenn eine Anwendung auf dem Knoten aktiviert wurde. Andernfalls wird ein Fehler gemeldet.

* **SourceId**: System.Hosting
* **Property**: Activation (einschließlich der Rolloutversion)
* **Nächste Schritte**: Falls die Anwendung fehlerhaft ist, untersuchen Sie, warum die Aktivierung nicht erfolgreich war.

Das folgende Beispiel zeigt eine erfolgreiche Aktivierung:

```powershell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -NodeName Node.1 -ApplicationName fabric:/WordCount

ApplicationName                    : fabric:/WordCount
NodeName                           : Node.1
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates :
                                     ServiceManifestName   : WordCountServicePkg
                                     NodeName              : Node.1
                                     AggregatedHealthState : Ok

HealthEvents                       :
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 130743727751144415
                                     SentAt                : 4/24/2015 6:12:55 PM
                                     ReceivedAt            : 4/24/2015 6:13:03 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : ->Ok = 4/24/2015 6:13:03 PM
```

### <a name="download"></a>Download
**System.Hosting** meldet einen Fehler, wenn das Herunterladen des Anwendungspakets nicht erfolgreich war.

* **SourceId**: System.Hosting
* **Property**: **Download:*RolloutVersion***.
* **Nächste Schritte**: Untersuchen Sie, warum das Herunterladen auf dem Knoten nicht erfolgreich war.

## <a name="deployedservicepackage-system-health-reports"></a>DeployedServicePackage-Systemintegritätsberichte
**System.Hosting** ist die Autorität für bereitgestellte Entitäten.

### <a name="service-package-activation"></a>Aktivierung des Dienstpakets
System.Hosting gibt die Meldung „OK“ aus, wenn die Aktivierung des Dienstpakets auf dem Knoten erfolgreich ist. Andernfalls wird ein Fehler gemeldet.

* **SourceId**: System.Hosting
* **Property**: Activation
* **Nächste Schritte**: Untersuchen Sie, warum die Aktivierung nicht erfolgreich war.

### <a name="code-package-activation"></a>Aktivierung des Codepakets
**System.Hosting** gibt für jedes Codepaket die Meldung „OK“ aus, wenn die Aktivierung erfolgreich ist. Wenn die Aktivierung nicht erfolgreich ist, wird gemäß Konfiguration eine Warnung ausgegeben. Wenn **CodePackage** nicht aktiviert werden kann oder mit einem Fehler beendet wird, der über den unter **CodePackageHealthErrorThreshold** konfigurierten Wert hinausgeht, meldet Hosting einen Fehler. Wenn ein Dienstpaket mehrere Codepakete enthält, wird jeweils ein eigener Aktivierungsbericht erstellt.

* **SourceId**: System.Hosting
* **Property**: Verwendet das Präfix **CodePackageActivation** und enthält den Namen des Codepakets und den Einstiegspunkt im Format **CodePackageActivation:*CodePackageName*:*SetupEntryPoint/EntryPoint*** (beispielsweise **CodePackageActivation:Code:SetupEntryPoint**).

### <a name="service-type-registration"></a>Diensttypregistrierung
**System.Hosting** gibt die Meldung „OK“ aus, wenn der Diensttyp erfolgreich registriert wurde. Wenn die Registrierung nicht in der vorgegebenen Zeit erfolgt ist (gemäß Konfiguration mit **ServiceTypeRegistrationTimeout**), wird ein Fehler gemeldet. Die Registrierung des Diensttyps wird für den Knoten aufgehoben, weil die Laufzeit geschlossen wurde. In diesem Fall gibt Hosting eine Warnung aus.

* **SourceId**: System.Hosting
* **Property**: Verwendet das Präfix **ServiceTypeRegistration** und enthält den Diensttypnamen (beispielsweise **ServiceTypeRegistration:FileStoreServiceType**).

Das folgende Beispiel zeigt ein fehlerfreies bereitgestelltes Dienstpaket:

```powershell
PS C:\> Get-ServiceFabricDeployedServicePackageHealth -NodeName Node.1 -ApplicationName fabric:/WordCount -ServiceManifestName WordCountServicePkg


ApplicationName       : fabric:/WordCount
ServiceManifestName   : WordCountServicePkg
NodeName              : Node.1
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.Hosting
                        Property              : Activation
                        HealthState           : Ok
                        SequenceNumber        : 130743727751456915
                        SentAt                : 4/24/2015 6:12:55 PM
                        ReceivedAt            : 4/24/2015 6:13:03 PM
                        TTL                   : Infinite
                        Description           : The ServicePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 6:13:03 PM

                        SourceId              : System.Hosting
                        Property              : CodePackageActivation:Code:EntryPoint
                        HealthState           : Ok
                        SequenceNumber        : 130743727751613185
                        SentAt                : 4/24/2015 6:12:55 PM
                        ReceivedAt            : 4/24/2015 6:13:03 PM
                        TTL                   : Infinite
                        Description           : The CodePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 6:13:03 PM

                        SourceId              : System.Hosting
                        Property              : ServiceTypeRegistration:WordCountServiceType
                        HealthState           : Ok
                        SequenceNumber        : 130743727753644473
                        SentAt                : 4/24/2015 6:12:55 PM
                        ReceivedAt            : 4/24/2015 6:13:03 PM
                        TTL                   : Infinite
                        Description           : The ServiceType was registered successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/24/2015 6:13:03 PM
```

### <a name="download"></a>Download
**System.Hosting** meldet einen Fehler, wenn das Herunterladen des Dienstpakets nicht erfolgreich war.

* **SourceId**: System.Hosting
* **Property**: **Download:*RolloutVersion***.
* **Nächste Schritte**: Untersuchen Sie, warum das Herunterladen auf dem Knoten nicht erfolgreich war.

### <a name="upgrade-validation"></a>Upgradeüberprüfung
**System.Hosting** meldet einen Fehler, wenn die Überprüfung während des Upgrades nicht erfolgreich war oder wenn das Upgrade auf dem Knoten zu einem Fehler führt.

* **SourceId**: System.Hosting
* **Property**: Verwendet das Präfix **FabricUpgradeValidation** und enthält die Upgradeversion.
* **Description**: Verweist auf den aufgetretenen Fehler

## <a name="next-steps"></a>Nächste Schritte
[Anzeigen von Service Fabric-Integritätsberichten](service-fabric-view-entities-aggregated-health.md)

[Melden und Überprüfen der Dienstintegrität](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Lokales Überwachen und Diagnostizieren von Diensten](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Service Fabric-Anwendungsupgrade](service-fabric-application-upgrade.md)




<!--HONumber=Feb17_HO2-->


