# Send-and-receive-messages-from-Azure-Queue-storage
In this project, you will create and configure Azure Queue Storage resources, then build a .NET app to send and receive messages using the Azure.Storage.Queues SDK. 
You learn how to provision storage resources, manage queue messages, and clean up your environment when finished.


Tasks performed in this exercise:
>>> 1. Create Azure Queue storage resources
 >   2. Assign a role to your Microsoft Entra user name
  >  3. Create a .NET console app to send and receive messages
   > 4. Clean up resources


Create Azure Queue storage resources
In this section of the exercise, you create the needed resources in Azure with the Azure CLI.
In your browser, navigate to the Azure portal https://portal.azure.com, signing in with your Azure credentials if prompted.

Use the [>_] button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a Bash environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal. If you are prompted to select a storage account to persist your files, select No storage account required, your subscription, and then select Apply.

Note: If you have previously created a cloud shell that uses a PowerShell environment, switch it to Bash.

In the cloud shell toolbar, in the Settings menu, select Go to Classic version (this is required to use the code editor).

Create a resource group for the resources needed for this exercise. 
If you already have a resource group you want to use, proceed to the next step. Replace myResourceGroup with a name you want to use for the resource group. You can replace eastus with a region near you if needed.

In this project, your Resource Group has already been created and named (myResourceGrouplod54676835), and you may safely skip this step.

#BASH 1
>>> az group create --name myResourceGroup --location eastus

>> Many of the commands require unique names and use the same parameters. Creating some variables will reduce the changes needed to the commands that create resources. Run the following commands to create the needed variables. 
Replace myResourceGroup with the name you're using for this exercise. 
If you changed the location in the previous step, make the same change in the location variable.

#BASH 2
>>> resourceGroup=myResourceGrouplod54676835
  > location=eastus
  > storAcctName=storactname54676835
>You will need the name assigned to the storage account later in this exercise.
>Run the following command and record the output.

#BASH 3
>>> echo $storAcctName
>Run the following command to create a storage account using the variable you created earlier.
>The operation takes a few minutes to complete.

#BASH 4
>>> az storage account create --resource-group $resourceGroup \
  > --name $storAcctName --location $location --sku Standard_LRS
>Assign a role to your Microsoft Entra user name
>To allow your app to send and receive messages, assign your Microsoft Entra user to the Storage Queue Data Contributor role.
>This gives your user account permission to create queues and send/receive messages using Azure RBAC.
Perform the following steps in the cloud shell.

Run the following command to retrieve the userPrincipalName from your account. This represents who the role will be assigned to.

#BASH 5
>>> userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
    > --headers 'Content-Type=application/json' \
    > --query userPrincipalName --output tsv)
Run the following command to retrieve the resource ID of the storage account. The resource ID sets the scope for the role assignment to a specific namespace.

#BASH 6
>>> resourceID=$(az storage account show --resource-group $resourceGroup \
  > --name $storAcctName --query id --output tsv)
Run the following command to create and assign the Storage Queue Data Contributor role.

#BASH 7
>>> az role assignment create --assignee $userPrincipal \
    > --role "Storage Queue Data Contributor" \
    > --scope $resourceID
Create a .NET console app to send and receive messages
Now that the needed resources are deployed to Azure, the next step is to set up the console application. The following steps are performed in the cloud shell.

Tip: Resize the cloud shell to display more information and code by dragging the top border. You can also use the minimise and maximise buttons to switch between the cloud shell and the main portal interface.

Run the following commands to create a directory to contain the project and change into the project directory.

#BASH 8
>>> mkdir queuestor
> cd queuestor
Create the .NET console application.

#BASH 9
>>> dotnet new console
Run the following commands to add the Azure.Storage.Queues and Azure.Identity packages to the project.

#BASH 10
>>> dotnet add package Azure.Storage.Queues
>   dotnet add package Azure.Identity
Add the starter code for the project
Run the following command in the Cloud Shell to begin editing the application.

#BASH 11
>>> code Program.cs
Replace any existing contents with the following code. Be sure to review the comments in the code, and replace with the storage account name you recorded earlier.

.NET SECTION
```csharp
TypeCopy
using Azure;
using Azure.Identity;
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;
using System;
using System.Threading.Tasks;

// Create a unique name for the queue
// TODO: Replace the <YOUR-STORAGE-ACCT-NAME> placeholder 
string queueName = "myqueue-" + Guid.NewGuid().ToString();
string storageAccountName = "<YOUR-STORAGE-ACCT-NAME>";


// ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE
// Create a DefaultAzureCredentialOptions object to exclude certain credentials
DefaultAzureCredentialOptions options = new()
{
    ExcludeEnvironmentCredential = true,
    ExcludeManagedIdentityCredential = true
};

// Instantiate a QueueClient to create and interact with the queue
QueueClient queueClient = new QueueClient(
    new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"),
    new DefaultAzureCredential(options));

Console.WriteLine($"Creating queue: {queueName}");

// Create the queue
await queueClient.CreateAsync();

Console.WriteLine("Queue created, press Enter to add messages to the queue...");
Console.ReadLine();
Press ctrl+s to save the file, then continue with the exercise.

Add code to send and list messages in a queue
Locate the // ADD CODE TO SEND AND LIST MESSAGES comment and add the following code directly after the comment. Be sure to review the code and comments.

// ADD CODE TO SEND AND LIST MESSAGES
// Send several messages to the queue with the SendMessageAsync method.
await queueClient.SendMessageAsync("Message 1");
await queueClient.SendMessageAsync("Message 2");

// Send a message and save the receipt for later use
SendReceipt receipt = await queueClient.SendMessageAsync("Message 3");

Console.WriteLine("Messages added to the queue. Press Enter to peek at the messages...");
Console.ReadLine();

// Peeking messages lets you view the messages without removing them from the queue.

foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
{
    Console.WriteLine($"Message: {message.MessageText}");
}

Console.WriteLine("\nPress Enter to update a message in the queue...");
Console.ReadLine();
Press ctrl+s to save the file, then continue with the exercise.

Add code to update a message and list the results
Locate the // ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES comment and add the following code directly after the comment. Be sure to review the code and comments.

// ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES
// Update a message with the UpdateMessageAsync method and the saved receipt
await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Message 3 has been updated");

Console.WriteLine("Message three updated. Press Enter to peek at the messages again...");
Console.ReadLine();


// Peek messages from the queue to compare updated content
foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
{
    Console.WriteLine($"Message: {message.MessageText}");
}

Console.WriteLine("\nPress Enter to delete messages from the queue...");
Console.ReadLine();


// ADD CODE TO DELETE MESSAGES AND THE QUEUE
// Delete messages from the queue with the DeleteMessagesAsync method.
foreach (var message in (await queueClient.ReceiveMessagesAsync(maxMessages: 10)).Value)
{
    // "Process" the message
    Console.WriteLine($"Deleting message: {message.MessageText}");

    // Let the service know we're finished with the message, and it can be safely deleted.
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
Console.WriteLine("Messages deleted from the queue.");
Console.WriteLine("\nPress Enter key to delete the queue...");
Console.ReadLine();

// Delete the queue with the DeleteAsync method.
Console.WriteLine($"Deleting queue: {queueClient.Name}");
await queueClient.DeleteAsync();

Console.WriteLine("Done");
Press Ctrl+S to save the file, then Ctrl+Q to exit the editor.

Sign in to Azure and run the app

```

In the cloud shell command-line pane, enter the following command to sign into Azure.

#BASH
>>> az login
You must sign into Azure, even though the cloud shell session is already authenticated.

Note: In most scenarios, just using az login will be sufficient. However, if you have subscriptions in multiple tenants, you may need to specify the tenant by using the --tenant parameter. See Sign into Azure interactively using Azure CLI for details.

Run the following command to start the console app. The app will pause many times during execution, waiting for you to press any key to continue. 
This allows you to view the messages in the Azure portal.

#BASH
>>> dotnet run
In the Azure portal, navigate to the Azure Storage account you created.

Expand > Data storage in the left navigation and select Queues.

Select the queue the application creates, and you can view the sent messages and monitor what the application is doing.

Clean up resources
Now that you have finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

In your browser, navigate to the Azure portal https://portal.azure.com, signing in with your Azure credentials if prompted.
You can just navigate to the resource group you created and view the contents of the resources used in this exercise.
On the toolbar, select Delete resource group.
Enter the resource group name and confirm that you want to delete it.

CAUTION: Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.

Congratulations
End of project. 
