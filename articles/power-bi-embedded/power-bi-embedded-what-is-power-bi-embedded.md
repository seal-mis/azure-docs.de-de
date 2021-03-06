---
title: Was ist Microsoft Power BI Embedded?
description: "Mit Power BI Embedded können Sie Power BI-Berichte in Ihre Web- oder mobilen Anwendungen integrieren, sodass Sie keine benutzerdefinierten Lösungen erstellen müssen."
services: power-bi-embedded
documentationcenter: 
author: guyinacube
manager: erikre
editor: 
tags: 
ms.assetid: 03649b72-b7d7-40ca-b077-12356d72d4f3
ms.service: power-bi-embedded
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: powerbi
ms.date: 03/20/2017
ms.author: asaxton
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: 0b5f7a2c3fd16ac32b0bc382616ca6600d378bb8
ms.lasthandoff: 03/21/2017


---
# <a name="what-is-microsoft-power-bi-embedded"></a>Was ist Microsoft Power BI Embedded?
Mit **Power BI Embedded**können Sie Power BI-Berichte direkt in webbasierte oder mobile Anwendungen integrieren.

![](media/powerbi-embedded-whats-is/what-is.png)

Power BI Embedded ist ein **Azure-Dienst**, mit dessen Hilfe unabhängige Softwarehersteller (Independent Software Vendors, ISVs) und App-Entwickler Power BI-Daten innerhalb ihrer Anwendungen darstellen können. Als Entwickler haben Sie Anwendungen erstellt. Diese Anwendungen verfügen über eigene Benutzer und eine besondere Auswahl an Features. Möglicherweise umfassen diese Apps auch integrierte Datenelemente wie Diagramme und Berichte, die jetzt durch Microsoft Power BI Embedded unterstützt werden können. Sie benötigen für die Verwendung Ihrer App kein Power BI-Konto. Sie können sich weiterhin wie gewohnt bei Ihrer Anwendung anmelden und ganz ohne zusätzliche Lizenzen die Berichtsoberfläche von Power BI anzeigen und damit interagieren.

## <a name="licensing-for-microsoft-power-bi-embedded"></a>Lizenzierung von Microsoft Power BI Embedded
Im Nutzungsmodell von **Microsoft Power BI Embedded** liegt die Verantwortung für die Lizenzierung von Power BI nicht beim Endbenutzer.  Stattdessen erwirbt der Entwickler der App, in der die visuellen Elemente genutzt werden, sogenannte **Sitzungen** , die im Rahmen des Abonnements abgerechnet werden, das diese Ressourcen besitzt. Weiterführende Informationen finden Sie auf der Seite [Preise](https://azure.microsoft.com/en-us/pricing/details/power-bi-embedded/).

## <a name="microsoft-power-bi-embedded-conceptual-model"></a>Microsoft Power BI Embedded – Konzeptmodell

![](media/powerbi-embedded-whats-is/model.png)

Wie bei jedem anderen Azure-Dienst werden Ressourcen für Power BI Embedded über die [Azure Resource Manager-APIs](https://msdn.microsoft.com/library/mt712306.aspx)bereitgestellt. In diesem Fall ist die Ressource, die Sie bereitstellen, eine **Power BI-Arbeitsbereichssammlung**.

## <a name="workspace-collection"></a>Arbeitsbereichssammlung
Eine **Arbeitsbereichssammlung** ist der oberste Azure-Container für Ressourcen, der 0 oder mehr **Arbeitsbereiche** enthält.  Eine **Arbeitsbereichssammlung** **** umfasst alle Azure-Standardeigenschaften sowie die folgenden:

* **Zugriffsschlüssel** – Schlüssel, die beim sicheren Aufrufen der Power BI-APIs verwendet werden (in einem späteren Abschnitt beschrieben).
* **Benutzer** – AAD-Benutzer (Azure Active Directory), die über Administratorrechte verfügen, um die Power BI-Arbeitsbereichssammlung über das Azure-Portal oder die Azure Resource Manager-API zu verwalten.
* **Region** – Im Rahmen der Bereitstellung einer **Arbeitsbereichssammlung** können Sie eine Region wählen, in der die Bereitstellung erfolgt. Weitere Informationen finden Sie unter [Azure-Regionen](https://azure.microsoft.com/regions/).

## <a name="workspace"></a>Arbeitsbereich
Ein **Arbeitsbereich** ist ein Container für Power BI-Inhalte, der Datasets und Berichte enthalten kann. Bei seiner anfänglichen Erstellung ist ein **Arbeitsbereich** leer. Sie werden Inhalte mithilfe von Power BI Desktop erstellen und die PBIX programmgesteuert über die [Power BI-Import-API](https://msdn.microsoft.com/library/mt711504.aspx) in Ihrem Arbeitsbereich bereitstellen. Sie können auch programmgesteuert Ihr Dataset erstellen und dann Berichte innerhalb der Anwendung erstellen, statt Power BI Desktop zu verwenden.

## <a name="using-workspace-collections-and-workspaces"></a>Verwenden von Arbeitsbereichssammlungen und Arbeitsbereichen
**Arbeitsbereichssammlungen** und **Arbeitsbereiche** sind Container mit Inhalten, die so verwendet und strukturiert werden, dass sie optimal dem Design der von Ihnen erstellten Anwendung entsprechen. Es gibt viele verschiedene Möglichkeiten, die Inhalte darin anzuordnen. Sie können alle Inhalte in einem Arbeitsbereich ablegen und später App-Token verwenden, um den Inhalt für Ihre Kunden weiter zu unterteilen. Sie können auch alle Kunden in separaten Arbeitsbereichen ablegen, damit sie getrennt behandelt werden. Alternativ dazu können Sie Benutzer nach Region statt nach Kunden einteilen. Anhand dieses flexiblen Designs können Sie die beste Möglichkeit zum Strukturieren von Inhalten auswählen.

## <a name="cached-datasets"></a>Zwischengespeicherte Datasets
Zwischengespeicherte Datasets können verwendet werden.  Allerdings können zwischengespeicherte Daten nicht mehr aktualisiert werden, nachdem sie in **Microsoft Power BI Embedded**geladen wurden. Ein zwischengespeichertes Dataset bedeutet, dass Sie die Daten in Power BI Desktop importiert haben, anstatt DirectQuery zu verwenden.

## <a name="authentication-and-authorization-with-app-tokens"></a>Authentifizierung und Autorisierung mit App-Token
**Microsoft Power BI Embedded** überlässt Ihrer Anwendung die gesamte erforderliche Benutzerauthentifizierung und -autorisierung. Es gibt keine explizite Anforderung, dass Ihre Endbenutzer Kunden von Azure Active Directory (Azure AD) sein müssen.  Stattdessen erteilt Ihre Anwendung **Microsoft Power BI Embedded** über **Token zur Anwendungsauthentifizierung (App-Token)** die Autorisierung zum Rendern eines Power BI-Berichts.  Diese **App-Token** werden nach Bedarf erstellt, wenn Ihre App einen Bericht rendern soll.

![](media/powerbi-embedded-whats-is/app-tokens.png)

**Token zur Anwendungsauthentifizierung (App-Token)** dienen zum Authentifizieren bei **Microsoft Power BI Embedded**.  Es gibt drei Typen von **App-Token**:

1. Bereitstellungstoken: Werden bei der Bereitstellung eines neuen **Arbeitsbereichs** in einer **Arbeitsbereichssammlung** verwendet.
2. Entwicklungstoken: Werden bei direkten Aufrufen der **Power BI-REST-APIs**
3. Einbettungstoken: Werden bei Aufrufen zum Rendern eines Berichts im eingebetteten IFrame verwendet.

Diese Token werden für die verschiedenen Phasen Ihrer Interaktionen mit **Microsoft Power BI Embedded**verwendet.  Die Token sind so konzipiert, dass Sie über Ihre App Berechtigungen an Power BI delegieren können. Weitere Informationen finden Sie unter [Authentifizieren und Autorisieren mit Power BI Embedded](power-bi-embedded-app-token-flow.md).

## <a name="create-or-edit-reports-within-your-application"></a>Erstellen oder Bearbeiten von Berichten in Ihrer Anwendung

Sie können jetzt vorhandene Berichte bearbeiten oder neue Berichte direkt in der Anwendung erstellen, ohne Power BI Desktop verwenden zu müssen. Dies setzt voraus, dass ein Dataset in Ihrem Arbeitsbereich vorhanden ist.

## <a name="see-also"></a>Weitere Informationen

[Häufige Microsoft Power BI Embedded-Szenarios](power-bi-embedded-scenarios.md)  
[Erste Schritte mit Microsoft Power BI Embedded](power-bi-embedded-get-started.md)  
[Erste Schritte mit dem Beispiel](power-bi-embedded-get-started-sample.md)  
[Einbetten eines Berichts](power-bi-embedded-embed-report.md)  
[Authentifizieren und Autorisieren in Power BI Embedded](power-bi-embedded-app-token-flow.md)  
[JavaScript-Einbettungsbeispiel](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[PowerBI-CSharp Git-Repository](https://github.com/Microsoft/PowerBI-CSharp)  
[PowerBI-Node Git-Repository](https://github.com/Microsoft/PowerBI-Node)  
Weitere Fragen? [Power BI-Community ausprobieren](http://community.powerbi.com/)

