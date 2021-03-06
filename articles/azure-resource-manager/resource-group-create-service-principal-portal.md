---
title: "Erstellen einer Identität für die Azure-App im Portal | Microsoft-Dokumentation"
description: Beschreibt das Erstellen einer neuen Active Directory-Anwendung und eines Dienstprinzipals, der mit der rollenbasierten Zugriffskontrolle in Azure-Resource Manager zum Verwalten des Zugriffs auf Ressourcen verwendet werden kann.
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 7068617b-ac5e-47b3-a1de-a18c918297b6
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/17/2017
ms.author: tomfitz
translationtype: Human Translation
ms.sourcegitcommit: 2a9075f4c9f10d05df3b275a39b3629d4ffd095f
ms.openlocfilehash: 3b132bbc89f64928f971f92365691d40c1aab420


---
# <a name="use-portal-to-create-active-directory-application-and-service-principal-that-can-access-resources"></a>Erstellen einer Active Directory-Anwendung und eines Dienstprinzipals mit Ressourcenzugriff mithilfe des Portals
> [!div class="op_single_selector"]
> * [PowerShell](resource-group-authenticate-service-principal.md)
> * [Azure-Befehlszeilenschnittstelle](resource-group-authenticate-service-principal-cli.md)
> * [Portal](resource-group-create-service-principal-portal.md)
>
>

Bei Verwendung einer Anwendung, die auf Ressourcen zugreifen oder diese ändern muss, müssen Sie eine Active Directory-Anwendung (AD) einrichten und ihr die erforderlichen Berechtigungen zuweisen. Dieser Ansatz ist der Ausführung der App mit Ihren Anmeldeinformationen aus folgenden Gründen vorzuziehen:

* Sie können der App-Identität Berechtigungen zuweisen, die sich von Ihren eigenen Berechtigungen unterscheiden. In der Regel sind diese Berechtigungen genau auf die Aufgaben der App beschränkt.
* Sie müssen keine Anmeldeinformationen für die App ändern, wenn sich Ihre Zuständigkeiten ändern. 
* Sie können ein Zertifikat verwenden, um die Authentifizierung beim Ausführen eines unbeaufsichtigten Skripts zu automatisieren.

In diesem Thema erfahren Sie, wie diese Schritte über das Portal ausgeführt werden. Es konzentriert sich auf eine Anwendung mit nur einem Mandanten, die nur zur Ausführung in einer einzigen Organisation vorgesehen ist. Anwendungen mit nur einem Mandanten werden in der Regel für innerhalb Ihrer Organisation ausgeführte Branchenanwendungen verwendet.
 
## <a name="required-permissions"></a>Erforderliche Berechtigungen
Um dieses Thema abzuschließen, müssen Sie über ausreichende Berechtigungen verfügen, um eine Anwendung bei Ihrem Active Directory zu registrieren, und die Anwendung einer Rolle in Ihrem Azure-Abonnement zuzuweisen. Stellen Sie sicher, dass Sie über die richtigen Berechtigungen für diese Schritte verfügen.

### <a name="check-active-directory-permissions"></a>Überprüfen der Active Directory-Berechtigungen
1. Melden Sie sich über das [Azure-Portal](https://portal.azure.com) bei Ihrem Azure-Konto an.
2. Wählen Sie **Azure Active Directory**.

     ![Azure Active Directory auswählen](./media/resource-group-create-service-principal-portal/select-active-directory.png)
3. Wählen Sie in Ihrem Active Directory die Option **Benutzereinstellungen** aus.

     ![Benutzereinstellungen auswählen](./media/resource-group-create-service-principal-portal/select-user-settings.png)
4. Überprüfen Sie die Einstellung **App Registrierungen**. Ist diese Option auf **Ja** festgelegt, können Benutzer ohne Administratorrechte AD-Apps registrieren. Diese Einstellung bedeutet, dass jeder Benutzer in Active Directory Apps registrieren kann. Sie können mit [Berechtigungen des Azure-Abonnements überprüfen](#check-azure-subscription-permissions) fortfahren.

     ![App-Registrierungen anzeigen](./media/resource-group-create-service-principal-portal/view-app-registrations.png)
5. Wenn die App-Registrierungseinstellung auf **Nein** festgelegt ist, können nur Administratorbenutzer Apps registrieren. Sie müssen überprüfen, ob Ihr Konto ein Administrator für Active Directory ist. Wählen Sie aus den schnellen Aufgaben **Übersicht** und **Benutzer suchen** aus.

     ![Benutzer suchen](./media/resource-group-create-service-principal-portal/find-user.png)
6. Suchen Sie nach Ihrem Konto, und wählen Sie es aus, wenn Sie es gefunden haben.

     ![Benutzer durchsuchen](./media/resource-group-create-service-principal-portal/show-user.png)
7. Wählen Sie für Ihr Konto **Verzeichnisrolle** aus. 

     ![Verzeichnisrolle](./media/resource-group-create-service-principal-portal/select-directory-role.png)
8. Zeigen Sie die Ihnen zugewiesene Rolle für Active Directory an. Wenn Ihrem Konto die Rolle „Benutzer“ zugewiesen wurde, die App-Registrierungseinstellung (aus den vorherigen Schritten) aber auf Administratoren begrenzt ist, bitten Sie Ihren Administrator, Ihnen entweder eine Administratorrolle zuzuweisen oder Benutzern zu erlauben, Apps zu registrieren.

     ![Rolle anzeigen](./media/resource-group-create-service-principal-portal/view-role.png)

### <a name="check-azure-subscription-permissions"></a>Überprüfen der Berechtigungen des Azure-Abonnements
Ihr Konto muss in Ihrem Azure-Abonnement über `Microsoft.Authorization/*/Write`-Zugriff verfügen, um einer Rolle eine AD-App zuzuweisen. Diese Aktion wird über die Rolle [Besitzer](../active-directory/role-based-access-built-in-roles.md#owner) oder [Benutzerzugriffsadministrator](../active-directory/role-based-access-built-in-roles.md#user-access-administrator) gewährt. Wenn Ihr Konto der Rolle **Mitwirkender** zugewiesen ist, verfügen Sie nicht über die erforderliche Berechtigung. Beim Versuch, einer Rolle den Dienstprinzipal zuzuweisen, erhalten Sie eine Fehlermeldung. 

So überprüfen Sie die Berechtigungen Ihres Abonnements

1. Wenn Ihr Active Directory-Konto aus den vorherigen Schritten noch nicht angezeigt wird, wählen Sie im linken Bereich **Azure Active Directory** aus.

2. Suchen Sie Ihr Azure Active Directory-Konto. Wählen Sie aus den schnellen Aufgaben **Übersicht** und **Benutzer suchen** aus.

     ![Benutzer suchen](./media/resource-group-create-service-principal-portal/find-user.png)
2. Suchen Sie nach Ihrem Konto, und wählen Sie es aus, wenn Sie es gefunden haben.

     ![Benutzer durchsuchen](./media/resource-group-create-service-principal-portal/show-user.png) 
     
3. Wählen Sie **Azure-Ressourcen** aus.

     ![Ressourcen auswählen](./media/resource-group-create-service-principal-portal/select-azure-resources.png) 
3. Zeigen Sie Ihre zugewiesenen Rollen an, und ermitteln Sie, ob Sie über die erforderlichen Berechtigungen verfügen, um einer Rolle eine AD-App zuzuweisen. Wenn dies nicht der Fall ist, bitten Sie Ihren Abonnementadministrator, Sie zur Rolle „Benutzerzugriffsadministrator“ hinzuzufügen. In der folgenden Abbildung wurde der Benutzer für zwei Abonnements der Rolle „Besitzer“ zugeordnet, das bedeutet, dass dieser Benutzer über die erforderlichen Berechtigungen verfügt. 

     ![Berechtigungen anzeigen](./media/resource-group-create-service-principal-portal/view-assigned-roles.png)

## <a name="create-an-active-directory-application"></a>Erstellen einer Active Directory-Anwendung
1. Melden Sie sich über das [Azure-Portal](https://portal.azure.com) bei Ihrem Azure-Konto an.
2. Wählen Sie **Azure Active Directory**.

     ![Azure Active Directory auswählen](./media/resource-group-create-service-principal-portal/select-active-directory.png)

4. Wählen Sie **App-Registrierungen** aus.   

     ![App-Registrierungen auswählen](./media/resource-group-create-service-principal-portal/select-app-registrations.png)
5. Wählen Sie **Hinzufügen**.

     ![App hinzufügen](./media/resource-group-create-service-principal-portal/select-add-app.png)

6. Geben Sie einen Namen und eine URL für die Anwendung an. Wählen Sie als Typ für die zu erstellende Anwendung entweder **Web-App/API** oder **Nativ** aus. Wählen Sie nach dem Festlegen der Werte **Erstellen** aus.

     ![Anwendung benennen](./media/resource-group-create-service-principal-portal/create-app.png)

Sie haben Ihre Anwendung erstellt.

## <a name="get-application-id-and-authentication-key"></a>Abrufen der Anwendungs-ID und des Authentifizierungsschlüssels
Beim programmgesteuerten Anmelden benötigen Sie die ID für Ihre Anwendung und einen Authentifizierungsschlüssel. Führen Sie die folgenden Schritte aus, um diese Werte abzurufen:

1. Wählen Sie in Active Directory unter **App Registrierungen** Ihre Anwendung aus.

     ![Anwendung auswählen](./media/resource-group-create-service-principal-portal/select-app.png)
2. Kopieren Sie die **Anwendungs-ID**, und speichern Sie sie in Ihrem Anwendungscode. Die Anwendungen im Abschnitt [Beispielanwendungen](#sample-applications) verweisen auf diesen Wert als Client-ID.

     ![CLIENT-ID](./media/resource-group-create-service-principal-portal/copy-app-id.png)
3. Wählen Sie zum Generieren eines Authentifizierungsschlüssels die Option **Schlüssel** aus.

     ![Schlüssel auswählen](./media/resource-group-create-service-principal-portal/select-keys.png)
4. Geben Sie eine Beschreibung des Schlüssels und eine Dauer für den Schlüssel ein. Wählen Sie dann die Option **Schließen**.

     ![Schlüssel speichern](./media/resource-group-create-service-principal-portal/save-key.png)

     Nach dem Speichern des Schlüssels wird der Wert des Schlüssels angezeigt. Kopieren Sie diesen Wert jetzt, da Sie ihn später nicht mehr abrufen können. Sie geben den Schlüsselwert zusammen mit der Anwendungs-ID ein, um sich als Anwendung anzumelden. Speichern Sie die Schlüsselwert an einem Ort, von dem Ihre Anwendung ihn abrufen kann.

     ![gespeicherter Schlüssel](./media/resource-group-create-service-principal-portal/copy-key.png)

## <a name="get-tenant-id"></a>Abrufen der Mandanten-ID
Beim programmgesteuerten Anmelden müssen Sie mit Ihrer Authentifizierungsanforderung die Mandanten-ID übergeben. 

1. Wählen Sie zum Abrufen der Mandanten-ID die Option **Eigenschaften** für Ihr Active Directory aus. 

     ![Active Directory-Eigenschaften auswählen](./media/resource-group-create-service-principal-portal/select-ad-properties.png)

2. Kopieren Sie die **-Verzeichnis-ID**. Dieser Wert ist Ihre Mandanten-ID.

     ![Mandanten-ID](./media/resource-group-create-service-principal-portal/copy-directory-id.png)

## <a name="assign-application-to-role"></a>Zuweisen einer Anwendung zur Rolle
Um auf Ressourcen in Ihrem Abonnement zuzugreifen, müssen Sie die Anwendung einer Rolle zuweisen. Entscheiden Sie, welche Rolle die geeigneten Berechtigungen für die Anwendung darstellt. Informationen zu verfügbaren Rollen finden Sie unter [RBAC: Integrierte Rollen](../active-directory/role-based-access-built-in-roles.md).

Sie können den Umfang auf Abonnement-, Ressourcengruppen- oder Ressourcenebene festlegen. Berechtigungen werden von niedrigeren Ebenen mit geringerem Umfang geerbt. Wenn z.B. der Leserolle für eine Ressourcengruppe eine Anwendung hinzugefügt wird, kann diese Rolle die Ressourcengruppe und alle darin enthaltenen Ressourcen lesen.

1. Navigieren Sie zur Bereichsebene, der Sie die Anwendung zuweisen möchten. Um z.B. einer Gruppe im Abonnementkontext eine Rolle zuzuweisen, wählen Sie **Abonnements** aus. Sie können stattdessen auch eine Ressourcengruppe oder Ressource auswählen.

     ![Abonnement auswählen](./media/resource-group-create-service-principal-portal/select-subscription.png)

2. Wählen Sie das entsprechende Abonnement (Ressourcengruppe oder Ressource) aus, dem die Anwendung zugewiesen werden soll.

     ![Abonnement für Zuweisung auswählen](./media/resource-group-create-service-principal-portal/select-one-subscription.png)

3. Wählen Sie **Access Control (IAM)** aus.

     ![„Zugriff“ auswählen](./media/resource-group-create-service-principal-portal/select-access-control.png)

4. Wählen Sie **Hinzufügen**.

     ![„Hinzufügen“ wählen](./media/resource-group-create-service-principal-portal/select-add.png)
6. Wählen Sie die Rolle aus, die Sie der Anwendung zuweisen möchten. In der folgenden Abbildung ist die Rolle **Leser** dargestellt.

     ![Rolle wählen](./media/resource-group-create-service-principal-portal/select-role.png)

8. Suchen Sie nach Ihrer Anwendung, und wählen Sie sie aus.

     ![Nach App suchen](./media/resource-group-create-service-principal-portal/search-app.png)
9. Wählen Sie **OK** aus, um das Zuweisen der Rolle abzuschließen. Ihre Anwendung wird in der Liste der Benutzer angezeigt, die einer Rolle für diesen Kontext zugewiesen sind.

## <a name="log-in-as-the-application"></a>Anmelden als Anwendung

Die Anwendung wird jetzt in Active Directory eingerichtet. Sie verfügen über eine ID und einen Schlüssel, um sich als Anwendung anzumelden. Die Anwendung ist einer Rolle zugewiesen, die die Ausführung bestimmter Aktionen ermöglicht. 

Anweisungen zur Anmeldung mithilfe von PowerShell finden Sie unter [Provide credentials through PowerShell](resource-group-authenticate-service-principal.md#provide-credentials-through-powershell) (Bereitstellen von Anmeldeinformationen über PowerShell).

Anweisungen zur Anmeldung mithilfe der Azure-CLI finden Sie unter [Provide credentials through Azure CLI](resource-group-authenticate-service-principal-cli.md#provide-credentials-through-azure-cli) (Bereitstellen von Anmeldeinformationen über die Azure-CLI).

Informationen zum Abrufen des Zugriffstokens für REST-Vorgänge finden Sie unter [Create the request](/rest/api/#create-the-request) (Erstellen der Anfrage).

Betrachten Sie die folgenden Beispielanwendungen, um Informationen zum Anmelden über den Anwendungscode zu erhalten.

### <a name="sample-applications"></a>Beispielanwendungen
Die folgenden Beispielanwendungen veranschaulichen die Anmeldung als AD-Anwendung:

**.NET**

* [Deploy an SSH Enabled VM with a Template with .NET (Bereitstellen eines SSH-fähigen virtuellen Computers mit einer Vorlage mit .NET)](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)
* [Manage Azure resources and resource groups with .NET (Verwalten von Azure-Ressourcen und -Ressourcengruppen mit .NET)](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)

**Java**

* [Getting Started with Resources - Deploy Using ARM Template - in Java (Erste Schritte mit Ressourcen – Bereitstellen mithilfe einer Azure Resource Manager-Vorlage – in Java)](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)
* [Getting Started with Resources - Manage Resource Group - in Java (Erste Schritte mit Ressourcen – Verwalten von Ressourcengruppen – in Java)](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group//)

**Python**

* [Deploy an SSH Enabled VM with a Template in Python (Bereitstellen eines SSH-fähigen virtuellen Computers mit einer Vorlage in Python)](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)
* [Manage Azure resources and resource groups with Python (Verwalten von Azure-Ressourcen und -Ressourcengruppen mit Python)](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)

**Node.js**

* [Deploy an SSH Enabled VM with a Template in Node.js (Bereitstellen eines SSH-fähigen virtuellen Computers mit einer Vorlage in Node.js)](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)
* [Manage Azure resources and resource groups with Node.js (Verwalten von Azure-Ressourcen und -Ressourcengruppen mit Node.js)](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)

**Ruby**

* [Deploy an SSH Enabled VM with a Template in Ruby (Bereitstellen eines SSH-fähigen virtuellen Computers mit einer Vorlage in Ruby)](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)
* [Manage Azure resources and resource groups with Ruby (Verwalten von Azure-Ressourcen und -Ressourcengruppen mit Ruby)](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)

## <a name="next-steps"></a>Nächste Schritte
* Informationen zum Einrichten einer Anwendung mit mehreren Mandanten finden Sie im [Entwicklerhandbuch für die Autorisierung mit der Azure Resource Manager-API](resource-manager-api-authentication.md).
* Informationen zum Festlegen von Sicherheitsrichtlinien finden Sie unter [Rollenbasierte Zugriffssteuerung in Azure](../active-directory/role-based-access-control-configure.md).  




<!--HONumber=Jan17_HO4-->


