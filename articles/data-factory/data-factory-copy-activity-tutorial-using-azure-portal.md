---
title: "Tutorial: Erstellen einer Pipeline mit Kopieraktivität mithilfe des Azure-Portals | Microsoft Docs"
description: "In diesem Tutorial erstellen Sie eine Azure Data Factory-Pipeline mit Kopieraktivität mithilfe des Data Factory-Editors im Azure-Portal."
services: data-factory
documentationcenter: 
author: spelluru
manager: jhubbard
editor: monicar
ms.assetid: d9317652-0170-4fd3-b9b2-37711272162b
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 02/14/2017
ms.author: spelluru
translationtype: Human Translation
ms.sourcegitcommit: 5e6ffbb8f1373f7170f87ad0e345a63cc20f08dd
ms.openlocfilehash: a4658f1eee3cdd24b3da47b4c7319c61ea39cb34
ms.lasthandoff: 03/24/2017


---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-azure-portal"></a>Tutorial: Erstellen einer Pipeline mit Kopieraktivität mithilfe des Azure-Portals
> [!div class="op_single_selector"]
> * [Übersicht und Voraussetzungen](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Kopier-Assistent](data-factory-copy-data-wizard-tutorial.md)
> * [Azure-Portal](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure Resource Manager-Vorlage](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST-API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

In diesem Tutorial wird veranschaulicht, wie Sie eine Azure Data Factory mit dem Azure-Portal erstellen und überwachen. Die Pipeline in der Data Factory verwendet eine Kopieraktivität zum Kopieren von Daten aus Azure Blob Storage in Azure SQL-Datenbank.

> [!NOTE]
> Die Datenpipeline in diesem Tutorial kopiert Daten aus einem Quelldatenspeicher in einen Zieldatenspeicher. Sie transformiert keine Eingabedaten in Ausgabedaten. Ein Tutorial zum Transformieren von Daten mithilfe von Azure Data Factory finden Sie unter [Tutorial: Erstellen der ersten Pipeline zum Verarbeiten von Daten mithilfe eines Hadoop-Clusters](data-factory-build-your-first-pipeline.md).
> 
> Sie können zwei Aktivitäten verketten (nacheinander ausführen), indem Sie das Ausgabedataset einer Aktivität als Eingabedataset der anderen Aktivität festlegen. Ausführliche Informationen finden Sie unter [Data Factory – Planung und Ausführung](data-factory-scheduling-and-execution.md). 

Hier sind die Schritte angegeben, die Sie im Rahmen dieses Tutorials ausführen:

| Schritt | Beschreibung |
| --- | --- |
| [Erstellen einer Azure Data Factory](#create-data-factory) |In diesem Schritt erstellen Sie eine Azure Data Factory mit dem Namen **ADFTutorialDataFactory**. |
| [Erstellen von verknüpften Diensten](#create-linked-services) |In diesem Schritt erstellen Sie zwei verknüpfte Dienste: **AzureStorageLinkedService** und **AzureSqlLinkedService**. <br/><br/>„AzureStorageLinkedService“ verbindet den Azure-Speicher und „AzureSqlLinkedService“ die Azure SQL-Datenbank mit „ADFTutorialDataFactory“. Die Eingabedaten für die Pipeline befinden sich in einem Blobcontainer im Azure-Blobspeicher. Ausgabedaten werden in einer Tabelle in der Azure SQL-Datenbank gespeichert. Daher fügen Sie diese beiden Datenspeicher als verknüpfte Dienste der Data Factory hinzu. |
| [Erstellen von Eingabe- und Ausgabedatasets](#create-datasets) |Im vorherigen Schritt haben Sie verknüpfte Dienste erstellt, die auf Datenspeicher verweisen, die Ein- und Ausgabedaten enthalten. In diesem Schritt definieren Sie zwei Datasets: **InputDataset** und **OutputDataset**. Die Datasets stehen für die Eingabe- bzw. Ausgabedaten, die in den Datenspeichern gespeichert sind. <br/><br/>Für „InputDataset“ geben Sie den Blobcontainer an, der ein Blob mit den Quelldaten enthält. Für „OutputDataset“ geben Sie die SQL-Tabelle an, in der die Ausgabedaten gespeichert werden. Sie können auch andere Eigenschaften wie Struktur, Verfügbarkeit und die Richtlinie angeben. |
| [Erstellen einer Pipeline](#create-pipeline) |In diesem Schritt erstellen Sie die Pipeline **ADFTutorialPipeline** in der ADFTutorialDataFactory. <br/><br/>Sie fügen der Pipeline eine **Kopieraktivität** hinzu, mit der Eingabedaten aus dem Azure-Blob in die Azure SQL-Ausgabetabelle kopiert werden. Die Kopieraktivität dient zum Verschieben von Daten in Azure Data Factory. Sie basiert auf einem global verfügbaren Dienst, mit dem Daten zwischen verschiedenen Datenspeichern sicher, zuverlässig und skalierbar kopiert werden können. Ausführliche Informationen zur Kopieraktivität finden Sie im Artikel [Datenverschiebungsaktivitäten](data-factory-data-movement-activities.md) . |
| [Überwachen der Pipeline](#monitor-pipeline) |In diesem Schritt überwachen Sie die Slices von Eingabe- und Ausgabetabellen im Azure-Portal. |

## <a name="prerequisites"></a>Voraussetzungen
Führen Sie vor dem Durcharbeiten dieses Tutorials die Schritte zur Erfüllung der Voraussetzungen aus, die im Artikel [Übersicht über das Tutorial](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) aufgeführt sind.

## <a name="create-data-factory"></a>Erstellen einer Data Factory
In diesem Schritt erstellen Sie im Azure-Portal eine Azure Data Factory namens **ADFTutorialDataFactory**.

1. Klicken Sie nach dem Anmelden am [Azure-Portal](https://portal.azure.com/) auf **Neu**, wählen Sie **Intelligence + Analytics** (Intelligence und Analyse), und klicken Sie auf **Data Factory**. 
   
   ![Neu -> Data Factory](./media/data-factory-copy-activity-tutorial-using-azure-portal/NewDataFactoryMenu.png)    
2. Gehen Sie auf dem Blatt **Neue Data Factory** wie folgt vor:
   
   1. Geben Sie **ADFTutorialDataFactory** als **Namen** ein. 
      
         ![Blatt "Neue Data Factory"](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-new-data-factory.png)
      
       Der Name der Azure Data Factory muss **global eindeutig**sein. Sollte der folgende Fehler auftreten, ändern Sie den Namen der Data Factory (beispielsweise in „<IhrName>ADFTutorialDataFactory“), und wiederholen Sie den Vorgang. Im Thema [Data Factory – Benennungsregeln](data-factory-naming-rules.md) finden Sie Benennungsregeln für Data Factory-Artefakte.
      
           Data factory name “ADFTutorialDataFactory” is not available  
      
       ![Data Factory-Name nicht verfügbar](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-data-factory-not-available.png)
   2. Wählen Sie Ihr Azure- **Abonnement**aus.
   3. Führen Sie für die Ressourcengruppe einen der folgenden Schritte aus:
      
      - Wählen Sie die Option **Use existing**(Vorhandene verwenden) und dann in der Dropdownliste eine vorhandene Ressourcengruppe. 
      - Wählen Sie **Neu erstellen**, und geben Sie den Namen einer Ressourcengruppe ein.   
         
          Bei einigen Schritten dieses Lernprogramms wird davon ausgegangen, dass Sie die Ressourcengruppe namens **ADFTutorialResourceGroup** verwenden. Weitere Informationen über Ressourcengruppen finden Sie unter [Verwenden von Ressourcengruppen zum Verwalten von Azure-Ressourcen](../azure-resource-manager/resource-group-overview.md).  
   4. Wählen Sie den **Standort** für die Data Factory aus. In der Dropdownliste werden nur Regionen angezeigt, die vom Data Factory-Dienst unterstützt werden.
   5. Wählen Sie **An Startmenü anheften**aus.     
   6. Klicken Sie auf **Erstellen**.
      
      > [!IMPORTANT]
      > Zum Erstellen von Data Factory-Instanzen müssen Sie Mitglied der Rolle [Data Factory-Mitwirkender](../active-directory/role-based-access-built-in-roles.md#data-factory-contributor) auf Abonnement- bzw. Ressourcengruppenebene sein.
      > 
      > Der Name der Data Factory kann in Zukunft als DNS-Name registriert und so öffentlich sichtbar werden.                
      > 
      > 
3. Klicken Sie in der Symbolleiste auf das Glockensymbol, um Status-/Benachrichtigungsmeldungen anzuzeigen. 
   
   ![Benachrichtigungsmeldungen](./media/data-factory-copy-activity-tutorial-using-azure-portal/Notifications.png) 
4. Nach Abschluss der Erstellung wird das Blatt **Data Factory** wie in dieser Abbildung angezeigt:
   
   ![Data Factory-Startseite](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-data-factory-home-page.png)

## <a name="create-linked-services"></a>Erstellen von verknüpften Diensten
Verknüpfte Dienste verknüpfen Datenspeicher oder Serverdienste mit einer Azure Data Factory. Informationen zu allen Quellen und Senken, die von der Kopieraktivität unterstützt werden, finden Sie unter [Unterstützte Datenspeicher](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Eine Liste mit Computediensten, die von Data Factory unterstützt werden, finden Sie unter [Verknüpfte Computedienste](data-factory-compute-linked-services.md) . In diesem Tutorial werden keine Computedienste verwendet. 

In diesem Schritt erstellen Sie zwei verknüpfte Dienste: **AzureStorageLinkedService** und **AzureSqlLinkedService**. Der verknüpfte Dienst „AzureStorageLinkedService“ verknüpft ein Azure-Speicherkonto und „AzureSqlLinkedService“ eine Azure SQL-Datenbank mit der **ADFTutorialDataFactory**. Später in diesem Tutorial erstellen Sie eine Pipeline, die Daten aus einem Blobcontainer in „AzureStorageLinkedService“ in eine SQL-Tabelle in „AzureSqlLinkedService“ kopiert.

### <a name="create-a-linked-service-for-the-azure-storage-account"></a>Erstellen eines verknüpften Diensts für das Azure-Speicherkonto
1. Klicken Sie auf dem Blatt **Data Factory** auf die Kachel **Erstellen und bereitstellen**, um den **Editor** für die Data Factory zu starten.
   
   ![Kachel „Erstellen und bereitstellen“](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-author-deploy-tile.png) 
2. Klicken Sie im **Editor** auf der Symbolleiste auf die Schaltfläche **Neuer Datenspeicher**, und wählen Sie im Dropdownmenü **Azure-Speicher** aus. Die JSON-Vorlage zum Erstellen eines mit einem Azure-Speicher verknüpften Diensts sollte im rechten Bereich angezeigt werden. 
   
    ![Editor – Schaltfläche „Neuer Datenspeicher“](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-editor-newdatastore-button.png)    
3. Ersetzen Sie `<accountname>` und `<accountkey>` durch die Werte für den Kontonamen und -schlüssel Ihres Azure-Speicherkontos. 
   
    ![Editor – Blobspeicher – JSON](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-editor-blob-storage-json.png)    
4. Klicken Sie in der Symbolleiste auf **Bereitstellen** . Das bereitgestellte **AzureStorageLinkedService** -Element wird jetzt in der Strukturansicht angezeigt. 
   
    ![Editor – Blobspeicher bereitstellen](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-editor-blob-storage-deploy.png)

> [!NOTE]
> Details zu JSON-Eigenschaften finden Sie unter [Verschieben von Daten in einen und aus einem Azure-Blob mithilfe von Azure Data Factory](data-factory-azure-blob-connector.md#azure-storage-linked-service) .
> 
> 

### <a name="create-a-linked-service-for-the-azure-sql-database"></a>Erstellen eines verknüpften Diensts für die Azure SQL-Datenbank
1. Klicken Sie im **Data Factory-Editor** auf der Symbolleiste auf die Schaltfläche **Neuer Datenspeicher**, und wählen Sie im Dropdownmenü die Option **Azure SQL-Datenbank** aus. Die JSON-Vorlage zum Erstellen eines mit der Azure SQL-Datenbank verknüpften Diensts sollte im rechten Bereich angezeigt werden.
2. Ersetzen Sie `<servername>`, `<databasename>`, `<username>@<servername>` und `<password>` durch die Namen für Ihren Azure SQL-Server, die Datenbank, das Benutzerkonto und das Kennwort. 
3. Klicken Sie auf der Symbolleiste auf **Bereitstellen**, um den verknüpften Dienst **AzureSqlLinkedService** zu erstellen und bereitzustellen.
4. Vergewissern Sie sich, dass **AzureSqlLinkedService** in der Strukturansicht angezeigt wird. 

> [!NOTE]
> Details zu JSON-Eigenschaften finden Sie unter [Verschieben von Daten in und aus Azure SQL-Datenbank mithilfe von Azure Data Factory](data-factory-azure-sql-connector.md#linked-service-properties) .
> 
> 

## <a name="create-datasets"></a>Erstellen von Datasets
Im vorherigen Schritt haben Sie die verknüpften Dienste **AzureStorageLinkedService** und **AzureSqlLinkedService** erstellt, um ein Azure Storage-Konto und eine Azure SQL-Datenbank mit der Data Factory **ADFTutorialDataFactory** zu verknüpfen. In diesem Schritt definieren Sie die beiden Datasets **InputDataset** und **OutputDataset**. Sie stellen die Ein- und Ausgabedaten in den Datenspeichern dar, auf die mit „AzureStorageLinkedService“ und „AzureSqlLinkedService“ verwiesen wird. Für „InputDataset“ geben Sie den Blobcontainer an, der ein Blob mit den Quelldaten enthält. Für „OutputDataset“ geben Sie die SQL-Tabelle an, in der die Ausgabedaten gespeichert werden. 

### <a name="create-input-dataset"></a>Erstellen eines Eingabedatasets
In diesem Schritt erstellen Sie ein Dataset namens **InputDataset**, das auf einen Blobcontainer im Azure-Speicher verweist. Dieser wird vom verknüpften Dienst **AzureStorageLinkedService** dargestellt.

1. Klicken Sie im **Editor** für die Data Factory auf **... More** (Mehr), klicken Sie auf **Neues Dataset**, und klicken Sie im Dropdownmenü auf **Azure-Blobspeicher**. 
   
    ![Menü „Neues Dataset“](./media/data-factory-copy-activity-tutorial-using-azure-portal/new-dataset-menu.png)
2. Ersetzen Sie den JSON-Code im rechten Bereich durch den folgenden JSON-Codeausschnitt: 
   
    ```JSON
    {
      "name": "InputDataset",
      "properties": {
        "structure": [
          {
            "name": "FirstName",
            "type": "String"
          },
          {
            "name": "LastName",
            "type": "String"
          }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
          "folderPath": "adftutorial/",
          "fileName": "emp.txt",
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ","
          }
        },
        "external": true,
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }
    ```   
    Beachten Sie folgende Punkte: 
   
   * „dataset **type**“ ist auf **AzureBlob** festgelegt.
   * **linkedServiceName** ist auf **AzureStorageLinkedService** festgelegt. Sie haben diesen verknüpften Dienst in Schritt 2 erstellt.
   * **folderPath** ist auf den Container **adftutorial** festgelegt. Mithilfe der **fileName** -Eigenschaft können Sie auch den Namen eines im Ordner enthaltenen Blobs angeben. Da Sie nicht den Namen des Blobs angeben, werden Daten aus allen Blobs im Container als Eingabedaten betrachtet.  
   * „format **type**“ ist auf **TextFormat** festgelegt.
   * Die Textdatei enthält die beiden Felder **FirstName** und **LastName**, die durch ein Komma getrennt sind (**columnDelimiter**).    
   * Die Verfügbarkeit (**availability**) ist auf **hourly**, (**frequency** auf **hour** und **interval** auf **1**) festgelegt. Der Data Factory-Dienst sucht also stündlich im Stammordner des angegebenen Blobcontainers (**adftutorial**) nach Eingabedaten. 
     
     Wenn Sie keinen Dateinamen (**fileName**) für ein Eingabedataset (**input**) angeben, werden alle Dateien/Blobs aus dem Eingabeordner (**folderPath**) als Eingaben betrachtet. Wenn Sie einen Dateinamen in der JSON-Datei angeben, wird nur die angegebene Datei/der angegebene Blob als Eingabe betrachtet.
     
     Wenn Sie keinen **fileName** für eine **Ausgabetabelle** angeben, werden die generierten Dateien in **folderPath** im folgenden Format benannt: Data.&lt;Guid&gt;.txt (Beispiel: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.).
     
     Um **folderPath** und **fileName** dynamisch basierend auf der **SliceStart**-Zeit festzulegen, verwenden Sie die **partitionedBy**-Eigenschaft. Im folgenden Beispiel verwendet folderPath die Angaben für Jahr, Monat und Tag aus „SliceStart“ (Startzeit des zu verarbeitenden Slices) und „fileName“ die Angabe für Stunde aus „SliceStart“. Wenn beispielsweise ein Slice für den Zeitpunkt „2016-09-20T08:00:00“ erzeugt wird, wird „folderName“ auf „wikidatagateway/wikisampledataout/2016/09/20“ und „fileName“ auf „08.csv“ festgelegt. 

    ```JSON     
    "folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
    "fileName": "{Hour}.csv",
    "partitionedBy": 
    [
       { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
       { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } }, 
       { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }, 
       { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } } 
    ],
    ```
3. Klicken Sie in der Symbolleiste auf **Bereitstellen**, um das Dataset **InputDataset** zu erstellen und bereitzustellen. Vergewissern Sie sich, dass **InputDataset** in der Strukturansicht angezeigt wird.

> [!NOTE]
> Details zu JSON-Eigenschaften finden Sie unter [Verschieben von Daten in einen und aus einem Azure-Blob mithilfe von Azure Data Factory](data-factory-azure-blob-connector.md#dataset-properties) .
> 
> 

### <a name="create-output-dataset"></a>Erstellen des Ausgabedatasets
In diesem Teilschritt erstellen Sie ein Ausgabedataset namens **OutputDataset**. Dieses Dataset verweist auf eine SQL-Tabelle in der durch **AzureSqlLinkedService**dargestellten Azure SQL-Datenbank. 

1. Klicken Sie im **Editor** für die Data Factory auf **... More** (Mehr), klicken Sie auf **Neues Dataset**, und klicken Sie im Dropdownmenü auf **Azure SQL**. 
2. Ersetzen Sie den JSON-Code im rechten Bereich durch den folgenden JSON-Codeausschnitt:

    ```JSON   
    {
      "name": "OutputDataset",
      "properties": {
        "structure": [
          {
            "name": "FirstName",
            "type": "String"
          },
          {
            "name": "LastName",
            "type": "String"
          }
        ],
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
          "tableName": "emp"
        },
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }
    ```       
    Beachten Sie folgende Punkte: 
   
   * „dataset **type**“ ist auf **AzureSQLTable** festgelegt.
   * **linkedServiceName** ist auf **AzureSqlLinkedService** festgelegt (diesen verknüpften Dienst haben Sie in Schritt 2 erstellt).
   * **tablename** ist auf **emp** festgelegt.
   * Die emp-Tabelle in der Datenbank enthält drei Spalten: **ID**, **FirstName** und **LastName**. Da es sich bei „ID“ um eine Identitätsspalte handelt, müssen Sie hier lediglich **FirstName** und **LastName** angeben.
   * Die Verfügbarkeit (**availability**) ist auf **hourly**, (**frequency** auf **hour** und **interval** auf **1**) festgelegt.  Der Data Factory-Dienst generiert in der Tabelle **emp** in der Azure SQL-Datenbank stündlich einen Ausgabedatenslice.
3. Klicken Sie in der Symbolleiste auf **Bereitstellen**, um das Dataset **OutputDataset** zu erstellen und bereitzustellen. Vergewissern Sie sich, dass **OutputDataset** in der Strukturansicht angezeigt wird. 

> [!NOTE]
> Details zu JSON-Eigenschaften finden Sie unter [Verschieben von Daten in und aus Azure SQL-Datenbank mithilfe von Azure Data Factory](data-factory-azure-sql-connector.md#linked-service-properties) .
> 
> 

## <a name="create-pipeline"></a>Erstellen der Pipeline
In diesem Schritt erstellen Sie eine Pipeline mit einer **Kopieraktivität**, für die **InputDataset** als Eingabe und **OutputDataset** als Ausgabe verwendet wird.

1. Klicken Sie im **Editor** für die Data Factory auf **... More** (Mehr), und klicken Sie auf **Neue Pipeline**. Alternativ können Sie in der Strukturansicht mit der rechten Maustaste auf **Pipelines** und dann auf **Neue Pipeline** klicken.
2. Ersetzen Sie den JSON-Code im rechten Bereich durch den folgenden JSON-Codeausschnitt: 

    ```JSON   
    {
      "name": "ADFTutorialPipeline",
      "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
          {
            "name": "CopyFromBlobToSQL",
            "type": "Copy",
            "inputs": [
              {
                "name": "InputDataset"
              }
            ],
            "outputs": [
              {
                "name": "OutputDataset"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "SqlSink",
                "writeBatchSize": 10000,
                "writeBatchTimeout": "60:00:00"
              }
            },
            "Policy": {
              "concurrency": 1,
              "executionPriorityOrder": "NewestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
        ],
        "start": "2016-07-12T00:00:00Z",
        "end": "2016-07-13T00:00:00Z"
      }
    } 
    ```   
    
    Beachten Sie folgende Punkte:
   
   * Der Abschnitt „Activities“ enthält nur eine Aktivität, deren **Typ** auf **Copy** festgelegt ist.
   * Die Eingabe für die Aktivität ist auf **InputDataset** und die Ausgabe für die Aktivität ist auf **OutputDataset** festgelegt.
   * Im Abschnitt **typeProperties** ist **BlobSource** als Quelltyp und **SqlSink** als Senkentyp angegeben.
     
     Ersetzen Sie den Wert der **start**-Eigenschaft durch den aktuellen Tag und den der **end**-Eigenschaft durch den nächsten Tag. Sie können auch nur den Datumsteil angeben und den Uhrzeitteil überspringen. „2016-02-03“ entspricht beispielsweise „2016-02-03T00:00:00Z“.
     
     Die Start- und Endzeit von Datums-/Uhrzeitangaben müssen im [ISO-Format](http://en.wikipedia.org/wiki/ISO_8601)angegeben werden. Beispiel: 2016-10-14T16:32:41Z. Die Zeitangabe **end** ist optional, wird aber in diesem Tutorial verwendet. 
     
     Wenn für die **end**-Eigenschaft kein Wert angegeben wird, wird sie als „**start + 48 Stunden**“ berechnet. Um die Pipeline auf unbestimmte Zeit auszuführen, geben Sie als Wert für die **end**-Eigenschaft **9999-09-09** an.
     
     Im obigen Beispiel ergeben sich 24 Datenslices, da jede Stunde ein Datenslice erstellt wird.
3. Klicken Sie in der Symbolleiste auf **Bereitstellen**, um die Pipeline **ADFTutorialPipeline** bereitzustellen. Vergewissern Sie sich, dass die Pipeline in der Strukturansicht angezeigt wird. 
4. Schließen Sie jetzt das Blatt **Editor**, indem Sie auf **X** klicken. Klicken Sie erneut auf **X**, um die **Data Factory**-Startseite für **ADFTutorialDataFactory** anzuzeigen.

**Glückwunsch!** Sie haben erfolgreich eine Azure Data Factory, verknüpfte Dienste, Tabellen und eine Pipeline erstellt und die Pipeline geplant.   

### <a name="view-the-data-factory-in-a-diagram-view"></a>Anzeigen einer Diagrammansicht der Data Factory
1. Klicken Sie auf dem Blatt **Data Factory** auf **Diagramm**.
   
    ![Blatt "Data Factory-Blade" – Kachel "Diagramm"](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-datafactoryblade-diagramtile.png)
2. Ein Diagramm wie in der folgenden Abbildung wird angezeigt: 
   
    ![Diagrammansicht](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-diagram-blade.png)
   
    Sie können die Ansicht vergrößern, verkleinern, auf 100 % anpassen, an die Fenstergröße anpassen, Pipelines und Tabellen automatisch positionieren und Informationen zur Datenherkunft anzeigen (d. h. vor- und nachgelagerte Elemente ausgewählter Elemente hervorheben).  Sie können auf ein Objekt (in der Ein-/Ausgabetabelle oder Pipeline) doppelklicken, um seine Eigenschaften anzuzeigen. 
3. Klicken Sie in der Diagrammansicht mit der rechten Maustaste auf **ADFTutorialPipeline**, und klicken Sie dann auf **Pipeline öffnen**. 
   
    ![Pipeline öffnen](./media/data-factory-copy-activity-tutorial-using-azure-portal/DiagramView-OpenPipeline.png)
4. Die Aktivitäten in der Pipeline sowie Ein- und Ausgabedatasets für die Aktivitäten sollten angezeigt werden. In diesem Tutorial gibt es nur eine Aktivität in der Pipeline (Kopieraktivität) mit „InputDataset“ als Eingabedataset und „OutputDataset“ als Ausgabedataset.   
   
    ![Ansicht mit geöffneter Pipeline](./media/data-factory-copy-activity-tutorial-using-azure-portal/DiagramView-OpenedPipeline.png)
5. Klicken Sie auf der Breadcrumb-Leiste links oben auf **Data Factory** , um zur Diagrammansicht zurückzukehren. In der Diagrammansicht werden alle Pipelines angezeigt. Bei diesem Beispiel haben Sie nur eine Pipeline erstellt.   

## <a name="monitor-pipeline"></a>Überwachen der Pipeline
In diesem Schritt verwenden Sie das Azure-Portal zur Überwachung der Aktivitäten in einer Azure Data Factory. 

### <a name="monitor-pipeline-using-diagram-view"></a>Überwachen der Pipeline mit der Diagrammansicht
1. Klicken Sie auf **X**, um die Ansicht **Diagramm** zu schließen und die Data Factory-Startseite für die Data Factory anzuzeigen. Führen Sie die folgenden Schritte aus, wenn Sie den Webbrowser geschlossen haben: 
   1. Navigieren Sie zum [Azure-Portal](https://portal.azure.com/). 
   2. Doppelklicken Sie im **Startmenü** auf **ADFTutorialDataFactory**, oder klicken Sie im Menü auf der linken Seite auf **Data Factorys**, und suchen Sie nach „ADFTutorialDataFactory“. 
2. In diesem Fenster sollten die Anzahl und die Namen der von Ihnen erstellten Tabellen und Pipelines angezeigt werden.
   
    ![Startseite mit Namen](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-datafactory-home-page-pipeline-tables.png)
3. Klicken Sie nun auf die Kachel **Datasets** .
4. Klicken Sie auf dem Blatt **Datasets** auf **InputDataset**. Bei diesem Dataset handelt es sich um das Eingabedataset für **ADFTutorialPipeline**.
   
    ![Datasets mit ausgewähltem InputDataset-Element](./media/data-factory-copy-activity-tutorial-using-azure-portal/DataSetsWithInputDatasetFromBlobSelected.png)   
5. Klicken Sie auf **… (Auslassungspunkte)** , um alle Datenslices anzuzeigen.
   
    ![Alle Eingabedatenslices](./media/data-factory-copy-activity-tutorial-using-azure-portal/all-input-slices.png)  
   
    Beachten Sie, dass alle Datenslices bis zum aktuellen Zeitpunkt den Status **Bereit** aufweisen, da die Datei **emp.txt** ständig im Blobcontainer **adftutorial\input** vorhanden ist. Überprüfen Sie, ob unten im Abschnitt **Letzte fehlerhafte Slices** keine Slices angezeigt werden.
   
    Die Listen **Letzte aktualisierte Slices** und **Letzte fehlerhafte Slices** werden anhand der **UHRZEIT DER LETZTEN AKTUALISIERUNG** sortiert. 
   
    Klicken Sie auf der Symbolleiste auf **Filter** , um die Slices zu filtern.  
   
    ![Eingabeslices filtern](./media/data-factory-copy-activity-tutorial-using-azure-portal/filter-input-slices.png)
6. Schließen Sie die Blätter, bis das Blatt **Datasets** angezeigt wird. Klicken Sie auf **OutputDataset**. Bei diesem Dataset handelt es sich um das Ausgabedataset für **ADFTutorialPipeline**.
   
    ![Blatt "Datasets"](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-datasets-blade.png)
7. Das Blatt **OutputDataset** sollte wie in der folgenden Abbildung dargestellt angezeigt werden:
   
    ![Blatt "Tabelle"](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-table-blade.png) 
8. Beachten Sie, dass die Datenslices bis zum aktuellen Zeitpunkt bereits erstellt wurden und den Status **Bereit**aufweisen. Im Abschnitt **Problemslices** am unteren Rand werden keine Slices angezeigt.
9. Klicken Sie auf **… (Auslassungspunkte)** , um alle Slices anzuzeigen.
   
    ![Blatt "Datenslices"](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-dataslices-blade.png)
10. Klicken Sie in der Liste auf einen beliebigen Datenslice. Das Blatt **Datenslice** wird angezeigt.
    
     ![Blatt "Datenslice"](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-dataslice-blade.png)
    
     Wenn der Slice nicht den Status **Bereit** hat, sehen Sie die vorgelagerten Slices, die nicht bereit sind und das Ausführen des aktuellen Slices blockieren, in der Liste **Vorgelagerte Slices, die nicht bereit sind**.
11. Auf dem Blatt **DATENSLICE** sollten in der Liste im unteren Bereich alle Aktivitätsausführungen angezeigt werden. Klicken Sie auf eine **Aktivitätsausführung**, um das Blatt **Details zur Aktivitätsausführung** anzuzeigen. 
    
    ![Aktivitätsausführung – Details](./media/data-factory-copy-activity-tutorial-using-azure-portal/ActivityRunDetails.png)
12. Klicken Sie auf **X**, um alle Blätter zu schließen, bis Sie sich wieder im Startblatt für **ADFTutorialDataFactory** befinden.
13. (Optional) Klicken Sie auf der Startseite von **ADFTutorialDataFactory** auf **Pipelines**, dann auf dem Blatt **Pipelines** auf **ADFTutorialPipeline**, und führen Sie eine Detailsuche in den Eingabetabellen (**Consumed**) oder Ausgabetabellen (**Produced**) durch.
14. Starten Sie **SQL Server Management Studio**, stellen Sie eine Verbindung mit der Azure SQL-Datenbank her, und überprüfen Sie, ob die Zeilen in die Tabelle **emp** der Datenbank eingefügt wurden.
    
    ![SQL-Abfrageergebnisse](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-sql-query-results.png)

### <a name="monitor-pipeline-using-monitor--manage-app"></a>Überwachen der Pipeline mit der App „Überwachung und Verwaltung“
Sie können die App „Überwachung und Verwaltung“ auch zum Überwachen Ihrer Pipelines verwenden. Ausführliche Informationen zur Verwendung dieser App finden Sie unter [Überwachen und Verwalten von Azure Data Factory-Pipelines mit der neuen App „Überwachung und Verwaltung“](data-factory-monitor-manage-app.md).

1. Klicken Sie auf der Startseite Ihrer Data Factory auf die Kachel **Überwachung und Verwaltung**.
   
    ![Kachel „Überwachung und Verwaltung“](./media/data-factory-copy-activity-tutorial-using-azure-portal/monitor-manage-tile.png) 
2. Die **App „Überwachung und Verwaltung“** wird angezeigt. Ändern Sie die **Startzeit** und **Endzeit** in die Startzeit (2016-07-12) und Endzeit (2016-07-13) Ihrer Pipeline, und klicken Sie auf **Übernehmen**. 
   
    ![App „Überwachung und Verwaltung“](./media/data-factory-copy-activity-tutorial-using-azure-portal/monitor-and-manage-app.png) 
3. Wählen Sie in der Liste **Aktivitätsfenster** ein Aktivitätsfenster aus, um die Details dazu anzuzeigen. 
    ![Details zum Aktivitätsfenster](./media/data-factory-copy-activity-tutorial-using-azure-portal/activity-window-details.png)

## <a name="summary"></a>Zusammenfassung
In diesem Lernprogramm haben Sie eine Azure Data Factory erstellt, um Daten aus einem Azure-Blob in eine Azure SQL-Datenbank zu kopieren. Sie haben mithilfe des Azure-Portals die Data Factory, verknüpfte Dienste, Datasets und eine Pipeline erstellt. Nachfolgend sind die allgemeinen Schritte aufgeführt, die Sie in diesem Tutorial ausgeführt haben:  

1. Sie haben eine Azure **Data Factory**erstellt.
2. Sie haben **verknüpfte Dienste**erstellt:
   1. Einen verknüpften **Azure Storage** -Dienst zum Verknüpfen Ihres Azure Storage-Kontos, in dem Eingabedaten enthalten sind.     
   2. Einen verknüpften **Azure SQL** -Dienst zum Verknüpfen Ihrer Azure SQL-Datenbank, in der die Ausgabedaten enthalten sind. 
3. Sie haben **Datasets** erstellt, die Eingabedaten und Ausgabedaten für Pipelines beschreiben.
4. Sie haben eine **Pipeline** mit einer **Kopieraktivität**, mit **BlobSource** als Quelle und mit **qlSink** als Senke erstellt.  

## <a name="see-also"></a>Weitere Informationen
| Thema | Beschreibung |
|:--- |:--- |
| [Pipelines](data-factory-create-pipelines.md) |Dieser Artikel enthält Informationen zu Pipelines und Aktivitäten in Azure Data Factory. |
| [Datasets](data-factory-create-datasets.md) |Dieser Artikel enthält Informationen zu Datasets in Azure Data Factory. |
| [Planung und Ausführung](data-factory-scheduling-and-execution.md) |In diesem Artikel werden die Planungs- und Ausführungsaspekte des Azure Data Factory-Anwendungsmodells erläutert. |

