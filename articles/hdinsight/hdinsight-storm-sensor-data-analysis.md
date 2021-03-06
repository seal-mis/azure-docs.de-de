---
title: Analysieren von Sensordaten mit Apache Storm und HBase | Microsoft-Dokumentation
description: Erfahren Sie, wie Sie mit einem virtuellen Netzwerk eine Verbindung mit Apache Storm herstellen. Verwenden Sie Storm mit HBase, um Sensordaten von einem Event Hub zu verarbeiten und mit D3.js darzustellen.
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: a9a1ac8e-5708-4833-b965-e453815e671f
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 03/21/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 4f2230ea0cc5b3e258a1a26a39e99433b04ffe18
ms.openlocfilehash: 138c90a1a9cbf3b85856f372beeb4472edb9e2e8
ms.lasthandoff: 03/25/2017


---
# <a name="analyze-sensor-data-with-apache-storm-event-hub-and-hbase-in-hdinsight-hadoop"></a>Analysieren von Sensordaten mit Apache Storm, Event Hub und HBase in HDInsight (Hadoop)

Erfahren Sie, wie Sie Apache Storm in HDInsight zur Verarbeitung von Sensordaten von Azure Event Hub verwenden. Die Daten werden dann in Apache HBase in HDInsight gespeichert und mithilfe von D3.js visualisiert.

Die in diesem Dokument verwendete Azure Resource Manager-Vorlage veranschaulicht das Erstellen mehrerer Azure-Ressourcen in einer Ressourcengruppe. Mit der Vorlage werden ein virtuelles Azure-Netzwerk, zwei HDInsight-Cluster (Storm und HBase) und eine Azure-Web-App erstellt. Eine node.js-Implementierung eines Echtzeit-Webdashboards wird automatisch für die Web-App bereitgestellt.

> [!NOTE]
> Die Informationen und das Beispiel in diesem Dokument erfordern HDInsight-Version 3.5.
>
> Linux ist das einzige Betriebssystem, das unter HDInsight Version 3.4 oder höher verwendet wird. Weitere Informationen finden Sie unter [Ende des Lebenszyklus von HDInsight unter Windows](hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date).

## <a name="prerequisites"></a>Voraussetzungen

* Ein Azure-Abonnement. Siehe [Kostenlose Azure-Testversion](http://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).
  
  > [!IMPORTANT]
  > Sie benötigen keinen vorhandenen HDInsight-Cluster. Durch die Schritte in diesem Dokument werden die folgenden Ressourcen erstellt:
  > 
  > * Ein virtuelles Azure-Netzwerk
  > * Ein Storm-Cluster in HDInsight (Linux-basiert, zwei Workerknoten)
  > * Ein HBase-Cluster in HDInsight (Linux-basiert, zwei Workerknoten)
  > * Eine Azure-Web-App, die das Webdashboard hostet

* [Node.js](http://nodejs.org/): Wird für eine lokale Vorschau des Webdashboards in Ihrer Entwicklungsumgebung verwendet.
* [Java und JDK 1.7](http://www.oracle.com/technetwork/java/javase/downloads/index.html): Wird zum Entwickeln der Storm-Topologie verwendet.
* [Maven](http://maven.apache.org/what-is-maven.html): Dient zum Erstellen und Kompilieren des Projekts.
* [Git](http://git-scm.com/): Dient zum Herunterladen des Projekts von GitHub.
* Ein **SSH** -Client: Dient zum Herstellen der Verbindung mit den Linux-basierten HDInsight-Clustern. Weitere Informationen finden Sie unter [Verwenden von SSH mit Linux-basiertem Hadoop in HDInsight unter Linux, Unix oder OS X](hdinsight-hadoop-linux-use-ssh-unix.md).
    
    > [!NOTE]
    > Außerdem benötigen Sie Zugriff auf den `scp`-Befehl, der zum Kopieren von Dateien zwischen Ihrer lokalen Entwicklungsumgebung und dem HDInsight-Cluster über SSH verwendet wird.


## <a name="architecture"></a>Architektur

![Architekturdiagramm](./media/hdinsight-storm-sensor-data-analysis/devicesarchitecture.png)

Dieses Beispiel umfasst die folgenden Komponenten:

* **Azure Event Hubs**: Enthalten von Sensoren gesammelte Daten.
* **Storm in HDInsight**: Bietet Echtzeitverarbeitung von Event Hub-Daten.
* **HBase in HDInsight**: Stellt einen persistenten NoSQL-Datenspeicher für Daten bereit, nachdem sie von Storm verarbeitet wurden.
* **Azure Virtual Network-Dienst**: Ermöglicht die sichere Kommunikation zwischen Storm in HDInsight-Clustern und HBase in HDInsight-Clustern.
  
  > [!NOTE]
  > Ein virtuelles Netzwerk ist erforderlich, wenn die Java-HBase-Client-API verwendet wird. Sie ist nicht über das öffentliche Gateway für HBase-Cluster verfügbar gemacht. Die Installation von HBase- und Storm-Clustern in dem gleichen virtuellen Netzwerk ermöglicht dem Storm-Cluster (oder beliebigen anderen Systemen im virtuellen Netzwerk) direkten HBase-Zugriff mithilfe der Client-API.

* **Dashboard-Website**: Ein Beispiel-Dashboard, das Daten in Echtzeit aufzeichnet.
  
  * Die Website ist in "Node.js" implementiert, sodass sie auf jedem Clientbetriebssystem für Tests ausgeführt oder auf Azure-Websites bereitgestellt werden kann.
  * [Socket.io](http://socket.io/) wird für die Echtzeitkommunikation zwischen der Storm-Topologie und der Website verwendet.
    
    > [!NOTE]
    > Die Verwendung von Socket.io für die Kommunikation ist ein Implementierungsdetail. Sie können alle Kommunikationsframeworks, wie z. B. reine WebSockets oder SignalR, verwenden.

  * [D3.js](http://d3js.org/) wird verwendet, um die Daten grafisch darzustellen, die an die Website gesendet werden.

> [!IMPORTANT]
> Zwei Cluster sind erforderlich, da keine unterstützte Methode zum Erstellen eines HDInsight-Clusters für Storm und HBase verfügbar ist.

Die Topologie liest mithilfe der [org.apache.storm.eventhubs.spout.EventHubSpout](http://storm.apache.org/releases/0.10.1/javadocs/org/apache/storm/eventhubs/spout/class-use/EventHubSpout.html)-Klasse Daten aus Event Hub und schreibt mithilfe der [org.apache.storm.hbase.bolt.HBaseBolt](https://storm.apache.org/javadoc/apidocs/org/apache/storm/hbase/bolt/class-use/HBaseBolt.html)-Klasse Daten in HBase. Kommunikation mit der Website erfolgt mithilfe von [socket.io client.java](https://github.com/nkzawa/socket.io-client.java).

Im Folgenden ist die Topologieanordnung dargestellt:

![Topologiediagramm](./media/hdinsight-storm-sensor-data-analysis/sensoranalysis.png)

> [!NOTE]
> Dies ist eine vereinfachte Ansicht der Topologie. Zur Laufzeit wird eine Instanz jeder Komponente für jede Partition des Event Hubs erstellt, der gelesen wird. Diese Instanzen werden über die Knoten im Cluster verteilt, und Daten werden wie folgt zwischen ihnen weitergeleitet:
> 
> * Für Daten vom Spout an den Parser wird ein Lastenausgleich ausgeführt.
> * Vom Parser zum Dashboard und zu HBase übertragene Daten werden nach Geräte-ID gruppiert, sodass Nachrichten vom gleichen Gerät immer an die gleiche Komponente weitergeleitet werden.

### <a name="topology-components"></a>Topologiekomponenten

* **EventHub-Spout**: Der Spout wird als Teil der Apache-Storm-Version 0.10.0 und höher bereitgestellt.
  
  > [!NOTE]
  > Für den in diesem Beispiel verwendeten Event Hub-Spout ist ein Storm-Cluster der Version 3.3 oder 3.4 in HDInsight erforderlich. Informationen zur Verwendung von Event Hubs mit einer älteren Version von Storm in HDInsight finden Sie unter [Verarbeitung von Ereignissen von Azure Event Hubs mit Storm auf HDInsight](hdinsight-storm-develop-java-event-hub-topology.md).

* **ParserBolt.java**: Der Spout gibt reine JSON-Daten aus, und gelegentlich werden mehrere Ereignisse gleichzeitig ausgegeben. Dieser Bolt veranschaulicht, wie vom Spout ausgegebene Daten gelesen und als ein Tupel, das mehrere Felder enthält, an einen neuen Datenstrom ausgegeben werden.
* **DashboardBolt.java**: Diese Komponente veranschaulicht, wie die Socket.io-Clientbibliothek für Java zum Senden von Daten an das Webdashboard in Echtzeit verwendet wird.
* **Temperature.java**: Mit dieser Komponente werden die Topologie definiert und Konfigurationsdaten aus der Datei **Config.properties** geladen.

## <a name="prepare-your-environment"></a>Vorbereiten der Umgebung

Bevor Sie dieses Beispiel verwenden können, müssen Sie einen Azure Event Hub erstellen, der von der Storm-Topologie gelesen wird.

### <a name="configure-event-hub"></a>Konfigurieren von Event Hub

Event Hub ist die Datenquelle für dieses Beispiel. Führen Sie die folgenden Schritte aus, um einen Event Hub zu erstellen.

1. Wählen Sie im [Azure-Portal](https://portal.azure.com) die Optionen **+ Neu** -> **Internet der Dinge (IoT)** -> **Event Hubs** aus.
2. Führen Sie auf dem Blatt **Namespace erstellen** die folgenden Aufgaben durch:
   
   1. Geben Sie einen **Namen** für den Namespace ein.
   2. Wählen Sie einen Tarif. **Basic** ist für dieses Beispiel ausreichend.
   3. Wählen Sie aus, welches Azure- **Abonnement** verwendet werden soll.
   4. Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe.
   5. Wählen Sie den **Speicherort** für den Event Hub aus.
   6. Wählen Sie **An Dashboard anheften** aus, und klicken Sie dann auf **Erstellen**.

3. Nachdem der Erstellungsvorgang abgeschlossen ist, wird das Blatt „Event Hubs“ für Ihren Namespace angezeigt. Wählen Sie hier die Option **+ Event Hub hinzufügen**. Geben Sie auf dem Blatt **Event Hub erstellen** den Namen **sensordata** ein, und wählen Sie dann **Erstellen** aus. Behalten Sie für die anderen Felder den Standardwert bei.
4. Wählen Sie auf dem Blatt „Event Hubs“ für Ihren Namespace die Option **Event Hubs**. Wählen Sie den Eintrag **sensordata** aus.
5. Wählen Sie auf dem Blatt für den Event Hub „sensordata“ die Option **Richtlinien für gemeinsamen Zugriff**. Verwenden Sie den Link **+ Hinzufügen** , um die folgenden Richtlinien hinzuzufügen:

    | Richtlinienname | Ansprüche |
    | ----- | ----- |
    | devices | Senden |
    | storm | Empfangen |

1. Wählen Sie beide Richtlinien aus, und notieren Sie sich den Wert von **PRIMÄRER SCHLÜSSEL** . Sie benötigen den Wert für beide Richtlinien für die weiteren Schritte.

## <a name="download-and-configure-the-project"></a>Herunterladen und Konfigurieren des Projekts

Führen Sie den folgenden Befehl aus, um das Projekt von GitHub herunterzuladen.

    git clone https://github.com/Blackmist/hdinsight-eventhub-example

Nach Abschluss des Befehls verfügen Sie über die folgende Verzeichnisstruktur:

    hdinsight-eventhub-example/
        TemperatureMonitor/ - this contains the topology
            resources/
                log4j2.xml - set logging to minimal.
                hbase-site.xml - connection information for the HBase cluster.
                Config.properties - configuration information for the topology. How to read from Event Hub and the URI to the dashboard.
            src/ - the Java bolts and topology definition.
        dashboard/nodejs/ - this is the node.js web dashboard.
        SendEvents/ - utilities to send fake sensor data.

> [!NOTE]
> Dieses Dokument geht nicht ausführlich auf alle Details des in diesem Beispiel enthaltenen Codes ein. Der Code ist jedoch vollständig kommentiert.

Öffnen Sie die Datei **hdinsight-eventhub-example/TemperatureMonitor/resources/Config.properties**, und fügen Sie die Event Hub-Informationen in den folgenden Zeilen ein:

    eventhubspout.username = <shared access policy name that can read from Event Hub>
    eventhubspout.password = <shared access policy key>
    eventhubspout.namespace = <namespace of your Event Hub
    eventhubspout.entitypath = <name of your event hub>
    eventhubspout.partitions.count = 2

Speichern Sie die Datei, nachdem Sie diese Informationen hinzugefügt haben.

## <a name="compile-and-test-locally"></a>Kompilieren und lokales Testen

Vor dem Testen müssen Sie das Dashboard zur Anzeige der Ausgabe der Topologie und zum Generieren der Daten starten, die im Event Hub gespeichert werden sollen.

> [!IMPORTANT]
> Die HBase-Komponente dieser Topologie ist beim lokalen Testen nicht aktiv. Dies liegt daran, dass von außerhalb des virtuellen Azure-Netzwerks, das die Cluster enthält, kein Zugriff auf die Java-API für den HBase-Cluster möglich ist.

### <a name="start-the-web-application"></a>Starten der Webanwendung

1. Öffnen Sie eine neue Eingabeaufforderung oder ein Terminal, und wechseln Sie zum Verzeichnis **hdinsight-eventhub-example/dashboard**. Verwenden Sie den folgenden Befehl, um die von der Webanwendung benötigten Anwendungen zu installieren:
   
        npm install

2. Verwenden Sie den folgenden Befehl, um die Webanwendung zu starten.
   
        node server.js
   
    Es wird eine Meldung ähnlich der folgenden angezeigt:
   
        Server listening at port 3000

3. Öffnen Sie einen Webbrowser, und geben Sie **http://localhost:3000/** als Adresse ein. Eine Seite ähnlich der folgenden Abbildung wird angezeigt:
   
    ![Webdashboard](./media/hdinsight-storm-sensor-data-analysis/emptydashboard.png)
   
    Lassen Sie die Eingabeaufforderung oder das Terminal geöffnet. Verwenden Sie nach dem Testen STRG+C, um den Webserver zu beenden.

### <a name="start-generating-data"></a>Starten der Datengenerierung

> [!NOTE]
> Die Schritte in diesem Abschnitt verwenden „Node.js“, sodass sie auf einer beliebigen Plattform ausgeführt werden können. Andere Sprachbeispiele finden Sie im **SendEvents** -Verzeichnis.

1. Öffnen Sie eine neue Eingabeaufforderung oder Shell oder ein Terminal, und ändern Sie das Verzeichnis in **hdinsight-eventhub-example/SendEvents/nodejs**. Verwenden Sie den folgenden Befehl, um die für die Anwendung erforderlichen Abhängigkeiten zu installieren:
   
        npm install

2. Öffnen Sie die Datei **app.js** in einem Text-Editor, und fügen Sie die Event Hub-Informationen hinzu, die Sie zuvor abgerufen haben:
   
        // ServiceBus Namespace
        var namespace = 'YourNamespace';
        // Event Hub Name
        var hubname ='sensordata';
        // Shared access Policy name and key (from Event Hub configuration)
        var my_key_name = 'devices';
        var my_key = 'YourKey';
   
   > [!NOTE]
   > In diesem Beispiel wird vorausgesetzt, dass Sie **sensordata** als Name für den Event Hub und **devices** als Name für die Richtlinie mit dem Anspruch **Send** (Senden) verwendet haben.

3. Verwenden Sie den folgenden Befehl zum Einfügen neuer Einträge in Event Hub:
   
        node app.js
   
    Es werden mehrere Ausgabezeilen angezeigt, die die an Event Hub gesendeten Daten enthalten:
   
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"0","Temperature":7}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"1","Temperature":39}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"2","Temperature":86}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"3","Temperature":29}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"4","Temperature":30}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"5","Temperature":5}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"6","Temperature":24}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"7","Temperature":40}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"8","Temperature":43}
        {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"9","Temperature":84}

### <a name="start-the-topology"></a>Starten der Topologie

1. Öffnen Sie eine neue Eingabeaufforderung, eine Shell oder ein Terminal, und wechseln Sie zum Verzeichnis **hdinsight-eventhub-example/TemperatureMonitor**. Verwenden Sie dann den folgenden Befehl, um die Topologie zu starten:
   
        mvn compile exec:java
   
    Mit diesem Befehl wird die Topologie im lokalen Modus gestartet. Die Werte in der Datei **Config.properties** enthalten die Verbindungsinformationen für Event Hubs und die lokale Dashboardwebsite. Nach dem Starten liest die Topologie Einträge aus dem Event Hub und sendet sie an das Dashboard, das auf Ihrem lokalen Computer ausgeführt wird. Im Webdashboard sollten nun Linien angezeigt werden, die der folgenden Abbildung ähneln:
   
    ![Dashboard mit Daten](./media/hdinsight-storm-sensor-data-analysis/datadashboard.png)

2. Verwenden Sie während der Ausführung des Dashboards den Befehl `node app.js` aus den vorherigen Schritten, um neue Daten an Event Hubs zu senden. Da die Temperaturwerte zufällig generiert werden, sollte das Diagramm aktualisiert werden, um große Temperaturänderungen anzuzeigen.
   
   > [!NOTE]
   > Sie müssen sich im Verzeichnis **hdinsight-eventhub-example/SendEvents/Nodejs** befinden, wenn Sie den Befehl `node app.js` verwenden.

3. Beenden Sie die Topologie durch Eingabe von STRG+C, nachdem Sie überprüft haben, dass die Updates der Dashboards funktionieren. Sie können STRG+C auch verwenden, um den lokalen Webserver zu beenden.

## <a name="create-a-storm-and-hbase-cluster"></a>Erstellen eines Storm- und HBase-Clusters

In den Schritten in diesem Abschnitt wird mit einer [Azure Resource Manager-Vorlage](../azure-resource-manager/resource-group-template-deploy.md) ein virtuelles Azure-Netzwerk sowie ein Storm- und HBase-Cluster im virtuellen Netzwerk erstellt. Die Vorlage erstellt auch eine Azure-Web-App und stellt eine Kopie des Dashboards darin bereit.

> [!NOTE]
> Ein virtuelles Netzwerk wird verwendet, sodass die Topologie im Storm-Cluster direkt mithilfe der HBase-Java-API mit dem HBase-Cluster kommunizieren kann.

Die in diesem Dokument verwendete Resource Manager-Vorlage befindet sich in einem öffentlichen Blobcontainer unter **https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-hbase-storm-cluster-in-vnet.json**.

1. Klicken Sie auf die folgende Schaltfläche, um sich bei Azure anzumelden und die Resource Manager-Vorlage im Azure-Portal zu öffnen.
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-hbase-storm-cluster-in-vnet-3.5.json" target="_blank"><img src="./media/hdinsight-storm-sensor-data-analysis/deploy-to-azure.png" alt="Deploy to Azure"></a>

2. Geben Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die folgenden Werte ein:
   
    ![HDInsight-Parameter](./media/hdinsight-storm-sensor-data-analysis/parameters.png)
   
   * **Base Cluster Name** (Cluster-Basisname): Dieser Wert wird als Basisname für Storm- und HBase-Cluster verwendet. Bei Eingabe von **abc** wird z.B. ein Storm-Cluster mit dem Namen **storm-abc** und ein HBase-Cluster mit dem Namen **hbase-abc** erstellt.
   * **Benutzername für Clusteranmeldung**: Der Administratorbenutzername für die Storm- und HBase-Cluster.
   * **Kennwort für Clusteranmeldung**: Das Administratorbenutzerkennwort für die Storm- und HBase-Cluster.
   * **SSH-Benutzername**: Der für den Storm- und HBase-Cluster zu erstellende SSH-Benutzer.
   * **SSH-Kennwort**: Das Kennwort für den SSH-Benutzer für Storm- und HBase-Cluster.
   * **Standort**: Die Region, in der die Cluster erstellt werden.
     
     Klicken Sie auf **OK** , um die Parameter zu speichern.

3. Erstellen Sie im Abschnitt **Grundlagen** eine Ressourcengruppe, oder wählen Sie eine vorhandene aus.
4. Wählen Sie im Dropdownmenü **Ressourcengruppenstandort** den gleichen Speicherort aus, den Sie für den Parameter **Speicherort** im Abschnitt **Einstellungen** ausgewählt haben.
5. Lesen Sie die Geschäftsbedingungen, und wählen Sie anschließend die Option **Ich stimme den oben genannten Geschäftsbedingungen zu** aus.
6. Aktivieren Sie zum Schluss **An Dashboard anheften**, und wählen Sie dann **Kaufen** aus. Das Erstellen der Cluster dauert ca. 20 Minuten.

Sobald die Ressourcen erstellt sind, werden Sie zu einem Blatt für die Ressourcengruppe weitergeleitet, das die Cluster und das Webdashboard enthält.

![Ressourcengruppenblatt für vnet und Cluster](./media/hdinsight-storm-sensor-data-analysis/groupblade.png)

> [!IMPORTANT]
> Beachten Sie, dass die Namen der HDInsight-Cluster **storm-BASENAME** und **hbase-BASENAME** lauten, wobei BASENAME der Name ist, den Sie für die Vorlage angegeben haben. Sie verwenden diese Namen in einem späteren Schritt, wenn Sie eine Verbindung mit den Clustern herstellen. Beachten Sie auch, dass der Name der Dashboardwebsite **basename-Dashboard**lautet. Dieser Wert wird weiter unten in diesem Dokument verwendet.

## <a name="configure-the-dashboard-bolt"></a>Konfigurieren des Dashboard-Bolts

Um Daten an das Dashboard zu senden, das als Web-App bereitgestellt ist, müssen Sie die folgende Zeile in der Datei **Config.properties** ändern:

    dashboard.uri: http://localhost:3000

Ändern Sie `http://localhost:3000` in `http://BASENAME-dashboard.azurewebsites.net`, und speichern Sie die Datei. Ersetzen Sie **BASENAME** durch den Basisnamen, den Sie im vorherigen Schritt angegeben haben. Sie können auch die zuvor erstellte Ressourcengruppe verwenden, um das Dashboard auszuwählen und die URL anzuzeigen.

## <a name="create-the-hbase-table"></a>Erstellen der HBase-Tabelle

Um Daten in HBase zu speichern, müssen wir zuerst eine Tabelle erstellen. Erstellen Sie Ressourcen, die Storm zum Schreiben benötigt, vorab, denn der Versuch, Ressourcen innerhalb einer Storm-Topologie zu erstellen, kann dazu führen, dass mehrere Instanzen versuchen, die gleiche Ressource zu erstellen. Erstellen Sie die Ressourcen außerhalb der Topologie, und verwenden Sie Storm für Lesen/Schreiben und Analysen.

1. Verwenden Sie SSH, um mit dem SSH-Benutzernamen und dem Kennwort, die Sie während der Erstellung des Clusters der Vorlage angegeben haben, eine Verbindung mit dem HBase-Cluster herzustellen. Wenn Sie beispielsweise eine Verbindung mit dem `ssh` -Befehl herstellen, verwenden Sie die folgende Syntax:
   
        ssh USERNAME@hbase-BASENAME-ssh.azurehdinsight.net
   
    Ersetzen Sie in diesem Befehl **USERNAME** mit dem SSH-Benutzernamen, den Sie beim Erstellen des Clusters bereitgestellt, und **BASENAME** mit dem Basisnamen, den Sie angegeben haben. Geben Sie nach Aufforderung das Kennwort für den SSH-Benutzer ein.

2. Starten Sie die HBase-Shell in der SSH-Sitzung.
   
        hbase shell
   
    Nachdem die Shell geladen wurde, sehen Sie eine `hbase(main):001:0>`-Eingabeaufforderung.

3. Geben Sie in der HBase-Befehlszeile den folgenden Befehl ein, um eine Tabelle zu erstellen, in der die Sensordaten gespeichert werden:
   
        create 'SensorData', 'cf'

4. Stellen Sie mit folgendem Befehl sicher, dass die Tabelle erstellt wurde:
   
        scan 'SensorData'
   
    Hiermit sollten Informationen wie im folgenden Beispiel zurückgegeben werden, in dem angezeigt wird, dass die Tabelle 0 Zeilen enthält.
   
        ROW                   COLUMN+CELL                                       0 row(s) in 0.1900 seconds

5. Geben Sie `exit` ein, um HBase-Shell zu beenden:

## <a name="configure-the-hbase-bolt"></a>Konfigurieren des HBase-Bolts

Um vom Storm-Cluster aus in HBase zu schreiben, müssen Sie den HBase-Bolt mit den Konfigurationsdetails Ihres HBase-Clusters bereitstellen. In diesem Beispiel wird die Datei **hbase-site.xml** aus dem HBase-Cluster verwendet.

### <a name="download-the-hbase-sitexml"></a>Herunterladen von „hbase-site.xml“

Verwenden Sie über eine Befehlszeile SCP zum Herunterladen der Datei **hbase-site.xml** aus dem Cluster. Ersetzen Sie im folgenden Beispiel **USERNAME** mit dem SSH-Benutzernamen, den Sie beim Erstellen des Clusters bereitgestellt, und **BASENAME** mit dem Basisnamen, den Sie zuvor angegeben haben. Geben Sie nach Aufforderung das Kennwort für den SSH-Benutzer ein. Ersetzen Sie `/path/to/TemperatureMonitor/resources/hbase-site.xml` durch den Pfad zu dieser Datei im TemperatureMonitor-Projekt.

    scp USERNAME@hbase-BASENAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml /path/to/TemperatureMonitor/resources/hbase-site.xml

Durch diesen Befehl wird **hbase-site.xml** in den angegebenen Pfad heruntergeladen.

### <a name="enable-the-hbase-bolt"></a>Aktivieren des HBase-Bolts

Öffnen Sie zum Aktivieren der HBase-Bolt-Komponente die Datei **TemperatureMonitor/src/main/java/com/microsoft/examples/Temperature.java**, und heben Sie die Auskommentierung der folgenden Zeilen auf:

    // topologyBuilder.setBolt("HBase", new HBaseBolt("SensorData", mapper).withConfigKey("hbase.conf"), spoutConfig.getPartitionCount())
    //  .fieldsGrouping("Parser", "hbasestream", new Fields("deviceid")).setNumTasks(spoutConfig.getPartitionCount());

Speichern Sie die Datei, nachdem Sie die Auskommentierung der Zeilen aufgehoben haben.

## <a name="build-package-and-deploy-the-solution-to-hdinsight"></a>Erstellen, Verpacken und Bereitstellen der Lösung in HDInsight

Führen Sie die folgenden Schritte in Ihrer Entwicklungsumgebung aus, um die Storm-Topologie im Storm-Cluster bereitzustellen.

1. Führen Sie im Verzeichnis **TemperatureMonitor** den folgenden Befehl aus, um einen neuen Buildvorgang auszuführen und ein JAR-Paket aus Ihrem Projekt zu erstellen:
   
        mvn clean compile package
   
    Durch diesen Befehl wird eine Datei mit dem Namen **TemperatureMonitor-1.0-SNAPSHOT.jar** im **Ziel**verzeichnis für Ihr Projekt erstellt.

2. Verwenden Sie scp, um die Datei **TemperatureMonitor-1.0-SNAPSHOT.jar** auf Ihren Storm-Cluster hochzuladen. Ersetzen Sie im folgenden Beispiel **USERNAME** mit dem SSH-Benutzernamen, den Sie beim Erstellen des Clusters bereitgestellt, und **BASENAME** mit dem Basisnamen, den Sie zuvor angegeben haben. Geben Sie nach Aufforderung das Kennwort für den SSH-Benutzer ein.
   
        scp target/TemperatureMonitor-1.0-SNAPSHOT.jar USERNAME@storm-BASENAME-ssh.azurehdinsight.net:TemperatureMonitor-1.0-SNAPSHOT.jar

   > [!NOTE]
   > Das Hochladen der Datei kann mehrere Minuten dauern.

3. Wenn die Datei hochgeladen ist, stellen Sie über SSH eine Verbindung mit dem Cluster her.
   
        ssh USERNAME@storm-BASENAME-ssh.azurehdinsight.net

4. Verwenden Sie zum Starten der Topologie den folgenden Befehl in der SSH-Sitzung:
   
        storm jar TemperatureMonitor-1.0-SNAPSHOT.jar com.microsoft.examples.Temperature temperature

5. Nachdem die Topologie gestartet wurde, öffnen Sie die Website, die Sie in Azure veröffentlicht haben, in einem Browser, und verwenden Sie dann den Befehl `node app.js` zum Senden von Daten an Event Hub. Das Webdashboard sollte mit den Informationen aktualisiert werden.
   
    ![Dashboard](./media/hdinsight-storm-sensor-data-analysis/datadashboard.png)

## <a name="view-hbase-data"></a>Anzeigen von HBase-Daten

Befolgen Sie die folgenden Schritte, um eine Verbindung mit HBase herzustellen und zu überprüfen, ob die Daten in die Datei geschrieben wurden:

1. Verwenden Sie SSH zum Herstellen einer Verbindung mit dem HBase-Cluster.
   
        ssh USERNAME@hbase-BASENAME-ssh.azurehdinsight.net

2. Starten Sie die HBase-Shell in der SSH-Sitzung.
   
        hbase shell
   
    Nachdem die Shell geladen wurde, sehen Sie eine `hbase(main):001:0>`-Eingabeaufforderung.

3. Anzeigen von Zeilen aus der Tabelle:
   
        scan 'SensorData'
   
    Durch diesen Befehl sollten Informationen wie im folgenden Text zurückgegeben werden, in dem angezeigt wird, dass die Tabelle Daten enthält.
   
        hbase(main):002:0> scan 'SensorData'
        ROW                             COLUMN+CELL
        \x00\x00\x00\x00               column=cf:temperature, timestamp=1467290788277, value=\x00\x00\x00\x04
        \x00\x00\x00\x00               column=cf:timestamp, timestamp=1467290788277, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x01               column=cf:temperature, timestamp=1467290788348, value=\x00\x00\x00M
        \x00\x00\x00\x01               column=cf:timestamp, timestamp=1467290788348, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x02               column=cf:temperature, timestamp=1467290788268, value=\x00\x00\x00R
        \x00\x00\x00\x02               column=cf:timestamp, timestamp=1467290788268, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x03               column=cf:temperature, timestamp=1467290788269, value=\x00\x00\x00#
        \x00\x00\x00\x03               column=cf:timestamp, timestamp=1467290788269, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x04               column=cf:temperature, timestamp=1467290788356, value=\x00\x00\x00>
        \x00\x00\x00\x04               column=cf:timestamp, timestamp=1467290788356, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x05               column=cf:temperature, timestamp=1467290788326, value=\x00\x00\x00\x0D
        \x00\x00\x00\x05               column=cf:timestamp, timestamp=1467290788326, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x06               column=cf:temperature, timestamp=1467290788253, value=\x00\x00\x009
        \x00\x00\x00\x06               column=cf:timestamp, timestamp=1467290788253, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x07               column=cf:temperature, timestamp=1467290788229, value=\x00\x00\x00\x12
        \x00\x00\x00\x07               column=cf:timestamp, timestamp=1467290788229, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x08               column=cf:temperature, timestamp=1467290788336, value=\x00\x00\x00\x16
        \x00\x00\x00\x08               column=cf:timestamp, timestamp=1467290788336, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x09               column=cf:temperature, timestamp=1467290788246, value=\x00\x00\x001
        \x00\x00\x00\x09               column=cf:timestamp, timestamp=1467290788246, value=2015-02-10T14:43.05.00320Z
        10 row(s) in 0.1800 seconds
   
   > [!NOTE]
   > Dieser Scanvorgang gibt maximal 10 Zeilen aus der Tabelle zurück.

## <a name="delete-your-clusters"></a>Löschen der Cluster
[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Um die Cluster, den Speicher und die Web-App auf einmal zu löschen, löschen Sie die Ressourcengruppe, die sie enthält.

## <a name="next-steps"></a>Nächste Schritte

Weitere Beispiel-Storm-Topologien finden Sie unter [Beispiel-Storm-Topologien für Storm auf HDInsight](hdinsight-storm-example-topology.md).

Weitere Informationen zu Apache Storm finden Sie auf der [Apache Storm](https://storm.incubator.apache.org/) -Website.

Weitere Informationen zu HBase in HDInsight finden Sie unter [HBase mit HDInsight: Übersicht](hdinsight-hbase-overview.md).

Weitere Informationen über Socket.io finden Sie auf der [socket.io](http://socket.io/) -Website (in englischer Sprache).

Weitere Informationen zu D3.js finden Sie unter [D3.js – Data Driven Documents](http://d3js.org/)(in englischer Sprache).

Informationen zum Erstellen von Topologien in Java finden Sie unter [Entwickeln von Java-Topologien für Apache Storm in HDInsight](hdinsight-storm-develop-java-topology.md).

Informationen zum Erstellen von Topologien in .NET finden Sie unter [Entwickeln von C#-Topologien für Apache Storm in HDInsight](hdinsight-storm-develop-csharp-visual-studio-topology.md).

[azure-portal]: https://portal.azure.com

