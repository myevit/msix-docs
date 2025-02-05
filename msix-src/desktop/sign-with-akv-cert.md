---
title: Sign packages with Azure Key Vault
description: This article describes how to sign your app package with a certificate from Azure Key Vault.
ms.date: 05/07/2020
ms.topic: article
keywords: windows 10, msix, uwp, azure key vault, visual studio
---

# Sign packages with Azure Key Vault

In Visual Studio 2019 version 16.6 Preview 3 and later versions, you can sign UWP and desktop app packages with a certificate stored in Azure Key Vault for development and test scenarios. This tool extracts your public and private keys from your Azure Key Vault and loads them in the certificate store on your development computer in order to sign your package with SignTool.exe.

> [!IMPORTANT]
> The process described in this article is intended for development and test scenarios only. This process is not considered best practice for your private keys used for distribution. To ensure best security practices, your private keys for distribution should be handled only by the tooling recommended by your Continuous Integration and Continuous Deployment (CI/CD) platform.

## Prerequisites

- An Azure account. If you do not already have an Azure account, start [here](https://azure.microsoft.com/free/).
- An Azure Key Vault. For more info, see [Create a Key Vault](/azure/key-vault/secrets/quick-create-portal#create-a-vault).
- A valid package signing certificate imported into Azure Key Vault. The default certificate generated by Azure Key Vault will not work for code signing. For details on how to create a package signing certificate, see [Create a certificate for package signing](../package/create-certificate-package-signing.md).

## Import a certificate to your Key Vault

Adding a certificate to your Key Vault is very simple. In this example, we add a valid UWP code signing certificate called **UwpSigningCert.pfx**.

1. On the Key Vault properties pages, select **Certificates**.
2. Click on **Generate/Import**.
3. On the **Create a certificate** screen, choose the following values:
    - **Method of Certificate Creation**: Import
    - **Certificate Name**: UwpSigningCert
    - **Upload Certificate File**: UwpSigningCert.pfx
    - **Decrypt Certificate**: If your certificate is password-protected, provide it in the **Password** field.
4. Click **Create**.

> [!NOTE]
> This self-signed certificate will not be trusted by Windows unless it has been imported and trusted by an administrator. Keep all of your certificates secure including self-signed certificates.

## Configure the access policies for your Key Vault

You can control who has access to the contents of your Key Vault by using **access policies**. Key Vault access policies grant permissions separately to keys, secrets, and certificates. You can grant a user access only to keys and not to secrets. Access permissions for keys, secrets, and certificates are managed at the vault level. For more information, see [Azure Key Vault security](/azure/key-vault/general/overview-security#identity-and-access-management).

> [!NOTE]
> When you create a Key Vault in an Azure subscription, it is automatically associated with the Azure Active Directory tenant of the subscription. Anyone trying to manage or retrieve content from a Key Vault must be authenticated by Azure AD.

1. On the Key Vault properties pages, select **Access policies**.
2. Select **+ Add Access Policy**.
3. Click on the **Key permissions** dropdown and check the boxes for **Get** and **List** under **Key Management Operations**.
4. Click on **Select principal**, search for the user you are granting access to, and click **Select**.
5. Click **Add**.
6. Make sure to save your changes by clicking **Save**.

> [!NOTE]
> Giving users direct access to a key vault is **discouraged**. Ideally, users should be added to an Azure AD group, which is in turn given access to the key vault.

## Select a certificate from your Key Vault in Visual Studio

The **Create App Packages** wizard in Visual Studio enables you to choose the certificate that will be used to sign your app package. You can choose the package signing certificate via Azure Key Vault. You must provide the URI of the Key Vault that contains the certificate, and your Microsoft account authenticated in Visual Studio must have the correct permissions to access it.

1. Open your **UWP application** project or desktop **Windows application packaging project** in Visual Studio.
2. Select **Publish** -> **Package** -> **Create app packages...** to open the **Create App Packages** wizard.
3. On the **Select distribution method** page, select **Sideloading**.
4. On the **Select signing method** page, click **Select from Azure Key Vault...**.
5. After the **Select a certificate from Azure Key Vault** dialog appears, use the account picker to choose the account for which you have configured an access policy.
6. Enter the URI of the Key Vault. The URI can be found on the **Overview** page of the Key Vault and is identified by **DNS Name**.
7. Click the **View Metadata** button.
8. After the certificates have finished loading, select the one you want from the list (for example, **UwpSigningCert**).
9. Click **OK**.

> [!NOTE]
> The certificate will be imported to your local certificate store where it will be used for package signing.

## Decrypt your certificate with a password from Azure Key Vault

If you are using a local password-protected certificate (.pfx) to sign your app package, it can be difficult to manage the password used to decrypt it. When you are importing the certificate in the **Create App Packages** wizard, you will be prompted to manually enter the password. Alternatively, there is an option to choose the password from Azure Key Vault.

1. Open your **UWP application** project or desktop **Windows application packaging project** in Visual Studio.
2. Select **Publish** -> **Package** -> **Create app packages...** to open the **Create App Packages** wizard.
3. On the **Select distribution method** page, select **Sideloading**.
4. On the **Select signing method** page, click **Select From File..**.
5. After the **Certificate is password protected** dialog appears, click **Select Password From Key Vault**.
6. After the **Select a password from Azure Key Vault** dialog appears, use the account picker to choose the account for which you have configured an access policy.
7. Enter the URI of the Key Vault. The URI can be found on the **Overview** page of the Key Vault and is identified by **DNS Name**.
8. Click the **View Metadata** button.
9. After the passwords have finished loading, select the one you want from the list (for example, **UwpSigningCertPassword**).
10. Click **OK**.
