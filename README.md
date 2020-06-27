# ENet bindings for AppGameKit Tier 1
This provides bindings for ENet networking library for AGK. Just copy the `Commands.txt` and `Windows.dll` files to your compiler path, inside the `Plugins` folder. Something like:

    C:\Program Files\Steam\steamapps\common\App Game Kit 2\Tier 1\Compiler\Plugins\EnetAGK
    |_ Commands.txt
    |_ Windows.dll
    
Note that this only works on Windows 32 bits at the moment. If you want to add other platforms/architectures, feel free to make a PR.

# Commands

```
Initialize() // 0 on failure, 1 on success
Deinitialize()
// Only supports `localhost` at the moment
CreateServer(port, maxConnections) // returns an integer, `host_id` on success. 0 on failure.
DestroyHost()
CreateClient() // returns an integer, `host_id` on success. 0 on failure.
HostService() // returns the `event_id` or 0 if failed
GetEventType(host_id) // returns a string, the type can be "connect", "disconnect", "receive" or "none"
PeerSend(host_id, peer_id, message$)
HostBroadcast(host_id, message$)
HostConnect(host_id, host$, port) // returns a `peer_id` or 0 if failed
GetEventData(event_id) // returns a string, the data received
GetEventData,S,I,?get_event_data@@YAPADH@Z,0,0,0,0,?get_event_data@@YAPADH@Z
GetEventPeerAddressHost(event_id) // returns a string, the IP of the peer who sent the message
GetEventPeerAddressPort(event_id) // returns an integer, the port used to receive the message from the peer
```

## Example

```
#import_plugin EnetAGK as Enet

// server
Enet.Initialize()
host = Enet.CreateServer(1234, 32)
if host = 0 then Log("Failed to create server")

do
	event = Enet.HostService(host)
	eventType$ = Enet.GetEventType(event)
	if eventType$ = "connect"
		Log("Peer connected!")
		Log(Enet.GetEventPeerAddressHost(event) + ":" + Str(Enet.GetEventPeerAddressPort(event)))
	elseif eventType$ = "receive"
		Print("Peer sent message: " + Enet.GetEventData(event))
	endif

	Print("[SERVER]")
	Print(Str(ScreenFPS()))
	Sync()
loop

// client
Enet.Initialize()
host = Enet.CreateClient()
peer = Enet.HostConnect(host, "localhost", 1234)

if peer = 0 then Log("Failed to connect")

do
  Enet.PeerSend(host, peer, "Hello!")
	event = Enet.HostService(host)
	eventType$ = Enet.GetEventType(event)
	if not eventType$ = "none" then Log(eventType$)
	if eventType$ = "connect"
		Log("Connected to server!")
	elseif eventType$ = "receive"
		Log("Peer sent message!")
	endif

	Print("[CLIENT]")
  Print(ScreenFPS())
  Sync()
loop
```

__IMPORTANT__ The event ids are reused, as they are re-used so many times. So __do not save a reference to them__, just get whatever you need from them and let them go.