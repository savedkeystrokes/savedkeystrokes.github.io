# Powershell: CIMInstance
## or how I came to love ciminstance

Discussions were started with how you go about removing services. Some say Remove-Service will do, but if someone has managed to name the service with special charters like quotes, fixing that service can be difficult.

CimInstance comes with a Get Set and Remove option and with that comes a few standard parameters. My favourite is `-query`.

Once you have this you can wire up Sql-like statements 'Select * from win32process'