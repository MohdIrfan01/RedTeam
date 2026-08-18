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
