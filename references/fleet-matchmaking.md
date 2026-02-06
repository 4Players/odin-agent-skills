# Matchmaking Integration

Integration patterns for GameLift FlexMatch, Nakama, and custom matchmaking.

## Table of Contents

- [AWS GameLift FlexMatch Integration](#aws-gamelift-flexmatch-integration)
- [Nakama Fleet Manager Integration](#nakama-fleet-manager-integration)
- [Custom Matchmaking Implementation](#custom-matchmaking-implementation)

---

## AWS GameLift FlexMatch Integration

ODIN Fleet integrates with GameLift Anywhere + FlexMatch for robust matchmaking.

### Architecture

1. Game clients request matches via backend service
2. Backend calls AWS `StartMatchmaking` → receives ticketId
3. AWS FlexMatch searches for players based on ruleset
4. SNS notifications sent to backend on match events
5. Backend updates game client via database/WebSocket
6. Matched players connect to ODIN Fleet server

### Backend Service (Node.js/Firebase)

```javascript
// Start matchmaking
exports.StartFlexMatch = onRequest(async (req, res) => {
  const input = {
    ConfigurationName: req.body.Config,
    Players: req.body.PlayerData
  };
  const command = new StartMatchmakingCommand(input);
  const response = await gameLiftClient.send(command);
  res.status(200).send(response);  // Returns ticketId
});

// SNS notification endpoint
exports.MatchmakingNotification = onRequest(async (req, res) => {
  const message = JSON.parse(req.body.Message);
  const detail = message.detail;

  switch (detail.type) {
    case "MatchmakingSucceeded":
      await updateTicket(detail.ticketId, {
        status: "COMPLETED",
        ipAddress: detail.gameSessionInfo.ipAddress,
        port: detail.gameSessionInfo.port
      });
      break;

    case "MatchmakingTimedOut":
      await updateTicket(detail.ticketId, { status: "TIMEOUT" });
      break;
  }
  res.status(200).send("ok");
});
```

### FlexMatch Ruleset Example

```json
{
  "name": "SkillBasedRuleset",
  "ruleLanguageVersion": "1.0",
  "playerAttributes": [
    { "name": "skill", "type": "number", "default": 1000 },
    { "name": "gamemode", "type": "string" }
  ],
  "teams": [
    { "name": "players", "minPlayers": 2, "maxPlayers": 8 }
  ],
  "rules": [
    {
      "name": "SameGameMode",
      "type": "comparison",
      "operation": "=",
      "measurements": ["flatten(teams[*].players.attributes[gamemode])"]
    },
    {
      "name": "SkillRange",
      "type": "batchDistance",
      "batchAttribute": "skill",
      "maxDistance": 200
    }
  ]
}
```

### Game Client Flow

```typescript
// 1. Start matchmaking
const response = await backendService.startMatchmaking({
  playerId: "player-123",
  skill: 1200,
  gamemode: "deathmatch"
});
const ticketId = response.ticketId;

// 2. Poll for status updates
setInterval(async () => {
  const status = await backendService.checkTicket(ticketId);

  if (status.status === "COMPLETED") {
    connectToServer(status.ipAddress, status.port);
  }
}, 2000);
```

---

## Nakama Fleet Manager Integration

ODIN Fleet integrates as a fleet provider for Nakama matchmaking.

**Key Concept:** Nakama discovers ODIN servers via metadata filtering.

### Server Metadata Schema

```json
{
  "instance_state": "idle",  // or "occupied"
  "game_port": 7777,
  "game_mode": "deathmatch",
  "player_count": 0,
  "max_players": 16
}
```

### Nakama Module Registration

```go
func InitModule(ctx context.Context, logger runtime.Logger, db *sql.DB,
                nk runtime.NakamaModule, initializer runtime.Initializer) error {

    appID, _ := strconv.Atoi(os.Getenv("ODIN_APP_ID"))
    apiToken := os.Getenv("ODIN_API_TOKEN")

    cfg := odinfleet.GetDefaultConfiguration(int32(appID), apiToken)
    fm, err := odinfleet.CreateOdinFleetManager(logger, cfg,
                                                 odinfleet.ServerToInstanceInfo)

    initializer.RegisterFleetManager(fm)
    return nil
}
```

### Fleet Manager Implementation

```go
// List available servers (filters by metadata)
func (fm OdinFleetManager) List(ctx context.Context, query string,
                                 limit int, previousCursor string) (
    instances []*runtime.InstanceInfo, nextCursor string, err error) {

    params := &GetServersParams{}
    json.Unmarshal([]byte(query), params)

    servers, nextPage, err := fm.GetServers(ctx, params)

    for _, s := range servers {
        instance, _ := fm.serverToInstanceInfoConverterFunc(&fm, &s)
        instances = append(instances, instance)
    }
    return instances, nextCursor, nil
}

// Update server metadata
func (fm OdinFleetManager) Update(ctx context.Context, id string,
                                   playerCount int, metadata map[string]any) error {
    serviceId, _ := strconv.Atoi(id)
    request := fm.client.DockerAPI.DockerServicesMetadataUpdate(ctx, int32(serviceId)).
        PatchMetadataRequest(api.PatchMetadataRequest{Metadata: metadata})
    _, _, err = request.Execute()
    return err
}
```

### Game Server Metadata Updates

```go
// On startup: mark as idle
func (s *server) setReady() error {
    return s.updateMetadata(map[string]any{
        "instance_state": "idle",
        "game_port": 7777
    })
}

// On match start: mark as occupied
func (s *server) handleStartMatch(matchID string) error {
    return s.updateMetadata(map[string]any{
        "instance_state": "occupied",
        "match_id": matchID
    })
}

// On match end: mark as idle again
func (s *server) handleFinishMatch() error {
    return s.updateMetadata(map[string]any{
        "instance_state": "idle",
        "match_id": ""
    })
}
```

### Nakama Query Example

```json
{
  "Metadata": {
    "instance_state": "idle",
    "game_mode": "deathmatch"
  },
  "Location": {
    "Country": "de",
    "City": "frankfurt"
  }
}
```

---

## Custom Matchmaking Implementation

For custom matchmaking without GameLift or Nakama:

### Pattern 1: Server Browser

```typescript
// Backend: Get available servers
app.get('/api/servers', async (req, res) => {
  const servers = await fleetClient.getServers();

  const serverList = servers
    .filter(s => s.metadata.instance_state === 'idle')
    .map(s => ({
      id: s.id,
      name: s.metadata.server_name,
      map: s.metadata.map_name,
      players: s.metadata.player_count,
      maxPlayers: s.metadata.max_players,
      gameMode: s.metadata.game_mode,
      ip: s.addr,
      port: s.ports["Game Port"]?.publishedPort
    }));

  res.json(serverList);
});

// Game client refreshes list periodically
async function refreshServerList() {
  const servers = await fetch('/api/servers').then(r => r.json());
  displayServerList(servers);
}
```

### Pattern 2: Automatic Matchmaking

```typescript
// Backend: Find best server for player
app.post('/api/matchmake', async (req, res) => {
  const { region, gameMode, skillLevel } = req.body;

  const servers = await fleetClient.getServers();

  // Filter by criteria
  const candidates = servers.filter(s =>
    s.metadata.instance_state === 'idle' &&
    s.metadata.game_mode === gameMode &&
    s.location.country === region &&
    Math.abs(s.metadata.skill_level - skillLevel) <= 200
  );

  // Select server with lowest ping or player count
  const server = selectBestServer(candidates);

  // Reserve server
  await fleetClient.updateServerMetadata(server.id, {
    instance_state: 'occupied',
    reserved_for: req.body.playerId
  });

  res.json({
    ip: server.addr,
    port: server.ports["Game Port"]?.publishedPort,
    serverId: server.id
  });
});
```
