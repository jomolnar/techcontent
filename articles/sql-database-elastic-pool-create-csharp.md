<properties
    pageTitle="创建弹性数据库池 (C#) | Azure"
    description="使用 C# 数据库开发技术创建 Azure SQL 数据库弹性数据库池，以便可以在多个数据库之间共享资源。"
    services="sql-database"
    documentationCenter=""
    authors="stevestein"
    manager="jhubbard"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="03/15/2016"
    wacn.date="04/06/2016"/>

# 创建弹性数据库池 (C&#x23;)

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-pool-create-portal)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-create-powershell)
- [C#](/documentation/articles/sql-database-elastic-pool-create-csharp)


了解如何使用 C&#x23; 创建[弹性数据库池](/documentation/articles/sql-database-elastic-pool)。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤[使用 PowerShell 升级到 V12 并创建池](/documentation/articles/sql-database-upgrade-server-portal)。

这些示例使用[适用于 .NET 的 SQL 数据库库](https://msdn.microsoft.com/zh-cn/library/azure/mt349017.aspx)，因此你需要安装此库。你可以通过在 Visual Studio 中的[程序包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)中运行以下命令（“工具”>“NuGet 程序包管理器”>“程序包管理器控制台”）来进行安装：

    PM> Install-Package Microsoft.Azure.Management.Sql –Pre




## 创建弹性数据库池


    // Create elastic pool: configure create or update parameters and properties explicitly
    ElasticPoolCreateOrUpdateParameters newPoolParameters = new ElasticPoolCreateOrUpdateParameters()
    {
        Location = "China East",
        Properties = new ElasticPoolCreateOrUpdateProperties()
        {
            Edition = "Standard",
            Dtu = 400
            DatabaseDtuMin = 0,
            DatabaseDtuMax = 100
         }
    };

    // Create the pool
    var newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate("resourcegroup-name", "server-name", "ElasticPool1", newPoolParameters);

## 将现有数据库移入弹性数据库池


    // Retrieve current database properties
    currentDatabase = sqlClient.Databases.Get("resourcegroup-name", "server-name", "Database1").Database;

    // Configure create or update parameters with existing property values, override those to be changed.
    DatabaseCreateOrUpdateParameters updatePooledDbParameters = new DatabaseCreateOrUpdateParameters()
    {
        Location = currentDatabase.Location,
        Properties = new DatabaseCreateOrUpdateProperties()
        {
            Edition = "Standard",
            RequestedServiceObjectiveName = "ElasticPool",
            ElasticPoolName = "ElasticPool1",
            MaxSizeBytes = currentDatabase.Properties.MaxSizeBytes,
            Collation = currentDatabase.Properties.Collation,
        }
    };

    // Update the database
    var dbUpdateResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database1", updatePooledDbParameters);




## 在弹性数据库池中创建新数据库


    // Create a database: configure create or update parameters and properties explicitly
    DatabaseCreateOrUpdateParameters newPooledDatabaseParameters = new DatabaseCreateOrUpdateParameters()
    {
        Location = currentServer.Location,
        Properties = new DatabaseCreateOrUpdateProperties()
        {
            Edition = "Standard",
            RequestedServiceObjectiveName = "ElasticPool",
            ElasticPoolName = "ElasticPool1",
            MaxSizeBytes = 268435456000, // 250 GB,
            Collation = "SQL_Latin1_General_CP1_CI_AS"
        }
    };

    var poolDbResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database2", newPooledDatabaseParameters);





## 创建弹性数据库池 C&#x23; 示例


运行此示例需要以下库。你可以通过在 Visual Studio 中的[程序包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)中运行以下命令（“工具”>“NuGet 程序包管理器”>“程序包管理器控制台”）来进行安装

    PM> Install-Package Microsoft.Azure.Management.Sql –Pre
    PM> Install-Package Microsoft.Azure.Management.Resources –Pre
    PM> Install-Package Microsoft.Azure.Common.Authentication –Pre

创建控制台应用并将 Program.cs 的内容替换为以下内容。若要获取必需的客户端 id 和相关的值，请参阅[注册应用并获取所需的客户端值以便将你的应用连接到 SQL 数据库](/documentation/articles/sql-database-client-id-keys)。

    using Microsoft.Azure;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.Azure.Management.Sql;
    using Microsoft.Azure.Management.Sql.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System;
    
    namespace SqlDbElasticPoolsSample
    {
    class Program
    {
        // authentication variables
        static string subscriptionId = "<your Azure subscription>";
        static string clientId = "<your client id>";
        static string redirectUri = "<your redirect URI>";
        static string domainName = "<i.e. microsoft.partner.onmschina.cn>";
        static AuthenticationResult token;

        // resource group variables
        static string datacenterLocation = "<location>";
        static string resourceGroupName = "<resource group name>";

        // server variables
        static string serverName = "<server name>";
        static string adminLogin = "<server admin>";
        static string adminPassword = "<server password (store it securely!)>";
        static string serverVersion = "12.0";

        // elastic database pool variables
        static string elasticPoolName = "<pool name>";
        static string poolEdition = "Standard";
        static int poolDtus = 400;
        static int databaseMinDtus = 0;
        static int databaseMaxDtus = 100;


        static void Main(string[] args)
        {
            // Sign in to Azure
            token = GetAccessToken();
            Console.WriteLine("Logged in as: " + token.UserInfo.DisplayableId);

            // Create a resource group
            ResourceGroup rg = CreateResourceGroup();
            Console.WriteLine(rg.Name + " created.");

            // Create a server
            Console.WriteLine("Creating server... ");
            ServerGetResponse srvr = CreateServer();
            Console.WriteLine("Creation of server " + srvr.Server.Name + ": " + srvr.StatusCode.ToString());

            // Create an elastic database pool
            Console.WriteLine("Creating elastic database pool... ");
            ElasticPoolCreateOrUpdateResponse epool = CreateElasticDatabasePool();
            Console.WriteLine("Creation of pool " + epool.ElasticPool.Name + ": " + epool.Status.ToString());


            // keep the console open until Enter is pressed!
            Console.WriteLine("Press Enter to exit.");
            Console.ReadLine();
        }


        static ElasticPoolCreateOrUpdateResponse CreateElasticDatabasePool()
        {
            // Create a SQL Database management client
            SqlManagementClient sqlClient = new SqlManagementClient(new TokenCloudCredentials(subscriptionId, token.AccessToken));

            // Create elastic pool: configure create or update parameters and properties explicitly
            ElasticPoolCreateOrUpdateParameters newPoolParameters = new ElasticPoolCreateOrUpdateParameters()
            {
                Location = datacenterLocation,
                Properties = new ElasticPoolCreateOrUpdateProperties()
                {
                    Edition = poolEdition,
                    Dtu = poolDtus, 
                    DatabaseDtuMin = databaseMinDtus,
                    DatabaseDtuMax = databaseMaxDtus
                }
            };

            // Create the pool
            var newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate(resourceGroupName, serverName, elasticPoolName, newPoolParameters);
            return newPoolResponse;
        }



        static ServerGetResponse CreateServer()
        {

            // Create a SQL Database management client
            SqlManagementClient sqlClient = new SqlManagementClient(new TokenCloudCredentials(subscriptionId, token.AccessToken));

            // Create a server
            ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
            {
                Location = datacenterLocation,
                Properties = new ServerCreateOrUpdateProperties()
                {
                    AdministratorLogin = adminLogin,
                    AdministratorLoginPassword = adminPassword,
                    Version = serverVersion
                }
            };

            var serverResult = sqlClient.Servers.CreateOrUpdate(resourceGroupName, serverName, serverParameters);
            return serverResult;

        }


        static ResourceGroup CreateResourceGroup()
        {
           
            Microsoft.Rest.TokenCredentials creds = new Microsoft.Rest.TokenCredentials(token.AccessToken);
            // Create a resource management client 
            ResourceManagementClient resourceClient = new ResourceManagementClient(creds);

            // Resource group parameters
            ResourceGroup resourceGroupParameters = new ResourceGroup()
            {
                Location = datacenterLocation,
            };

            //Create a resource group
            resourceClient.SubscriptionId = subscriptionId;
            var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate(resourceGroupName, resourceGroupParameters);
            return resourceGroupResult;
        }


        private static AuthenticationResult GetAccessToken()
        {
            AuthenticationContext authContext = new AuthenticationContext
                ("https://login.chinacloudapi.cn/" + domainName /* Tenant ID or AAD domain */);

            AuthenticationResult token = authContext.AcquireToken
                ("https://management.azure.com/"/* the Azure Resource Management endpoint */,
                    clientId,
            new Uri(redirectUri) /* redirect URI */,
            PromptBehavior.Auto /* with Auto user will not be prompted if an unexpired token is cached */);

            return token;
        }
    }
    }

  

## 后续步骤

- [管理你的池](/documentation/articles/sql-database-elastic-pool-manage-csharp)
- [创建弹性作业](/documentation/articles/sql-database-elastic-jobs-overview)弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。


## 其他资源


- [SQL 数据库](/documentation/services/sql-databases)
- [Azure 资源管理 API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx)
- [弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。

<!---HONumber=Mooncake_0328_2016-->
