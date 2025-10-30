# QuakeWorld streambot [![Test](https://github.com/quakeworld/streambot/actions/workflows/test.yml/badge.svg)](https://github.com/quakeworld/streambot/actions/workflows/test.yml) [![codecov](https://codecov.io/gh/quakeworld/streambot/branch/main/graph/badge.svg)](https://codecov.io/gh/quakeworld/streambot) [![Go Report Card](https://goreportcard.com/badge/github.com/quakeworld/streambot)](https://goreportcard.com/report/github.com/quakeworld/streambot)

> Setup for automated QuakeWorld client accepting commands via Twitch chat.

![image](https://user-images.githubusercontent.com/1616817/188150714-cc81b7a6-d97d-4cb4-ad22-2100a17533ed.png)

* **Visit [twitch.tv/QuakeWorld](https://www.twitch.tv/QuakeWorld)** to see it in action.


## How does it work? (TLDR version)

ezQuake reads from a pipe located at `/tmp/ezquake_[username]` on posix systems, where `username` is the username of the user who started the ezQuake process.

So basically all you have to do is to write commands to `/tmp/ezquake_[username]`.

Most of the code in this project is related to making sure that the client always is connected to the "best" server available (check ezQuake process, current server, available servers, handling commands from Twitch etc).

## Stack

* Written in [Go (Golang)](https://github.com/golang/go)
* [ZeroMQ](https://zeromq.org/) - Communication/messages (single proxy and multiple subscribers/publishers)
* [serverstat](https://github.com/vikpe/serverstat) - Get info from QuakeWorld servers

## Overview

![image](https://user-images.githubusercontent.com/1616817/186941072-cc99679d-b1d0-41f7-bdba-913bb733e140.png)

* **Message Proxy**: Central point for communication.
* **Quake Manager**: Interaction with ezQuake
    * Server monitor (thread): Server events (map change etc)
    * Process monitor (thread): ezQuake events (started, stopped)

* **Twitch Manager**: Interaction with Twitch channel (e.g. set title).
* **Twitch Bot**: Interaction with Twitch chat.

### Quake Manager - evaluation loop
An evaluation is run to make sure that the client is connected to the "best" server available (auto mode enabled) unless commanded to join a specific server (auto mode disabled).

* Run every 10 seconds
* Join "best server" available. Servers are ranked using a
  custom [scoring algorithm](https://github.com/vikpe/serverstat/blob/main/qserver/mvdsv/qscore/qscore.go).
* Only change server in between matches or if current server has enabled a custom game mode (e.g. `race`).

![image](https://user-images.githubusercontent.com/1616817/186950564-2d731d49-c92a-4928-9ab2-b03dc931cf23.png)

## Chatbot commands
Example, find `XantoM` and join that server.

![image](https://user-images.githubusercontent.com/1616817/188152018-f4c43219-e969-4330-bb7d-2f89e4d45154.png)

| Command        | Description                                             |
|----------------|---------------------------------------------------------|
| `!auto`        | Enable auto-mode (join best server)                     |
| `!autotrack`   | Toggle autotrack                                        |
| `!cfg_load`    | Re-load config                                          |
| `!cmd [x]`     | Run command `x` (mod only)                              |
| `!console`     | Toggle console                                          |
| `!find [name]` | Find `[name]` and join server                           |
| `!lastscores ` | Show list of last scores                                |
| `!restart `    | Restart ezQuake client                                  |
| `!showscores ` | Show scoreboard                                         |
| `!track [x] `  | Track player `x`                                        |
| `!attack`      | Toggle between free float spectator mode and player pov |
| `!jump`        | Track next player                                       |

See [`internal/app/twitchbot/twitchbot.go`](https://github.com/vikpe/qw-streambot/blob/main/internal/app/twitchbot/twitchbot.go) for all commands.

## Requirements

* [ezQuake](https://github.com/ezQuake/ezquake-source/releases) (unix) - QuakeWorld client
* Twitch account for the channel
* Twitch account for the chatbot
* Add [Twitch access tokens](https://twitchtokengenerator.com/) to `.env.` (see [.env.example](./.env.example))
  * Channel access token to update title, scope: `channel:manage:broadcast`
    ```
    https://id.twitch.tv/oauth2/authorize
    ?response_type=token
    &client_id=CLIENT_ID
    &redirect_uri=http://localhost:3000
    &scope=channel%3Amanage%3Abroadcast
    ```
  * Chatbot access token to read and write to chat, scopes: `chat:read` and `chat:edit`.
* ZeroMQ: `apt-get install libzmq3-dev`

For streaming I highly recommend [Open Broadcaster Sofware (OBS)](https://obsproject.com/), see [Stream Setup](./stream_setup/) for details on the setup used on twitch.tv/quakeworld.

## Development

### Directory structure

Uses the [Standard Go Project Layout](https://github.com/golang-standards/project-layout).

```bash
cmd/       # Main applications
internal/  # Private application and library code
scripts/   # Various build, install operations
```

### Build

**Build specific app**

Example: build proxy

```shell
cd cmd/proxy
go build
```

**Build all apps**

```shell
./scripts/build.sh
```

### Run

**Single app**

Example: start the proxy.

```shell
./cmd/proxy/proxy
```

**App controller scripts**

Runs app forever (restarts on error/sigint with short timeout in between).

```shell
bash scripts/controllers/proxy.sh
bash scripts/controllers/quake_manager.sh
bash scripts/controllers/twitch_manager.sh
bash scripts/controllers/twitchbot.sh
bash scripts/controllers/ezquake.sh
```

### Test

```shell
go test ./... --cover
```

## Production

Build all apps and run all app controller scripts.

```shell
./scripts/build.sh && ./scripts/start.sh
```

## Credits

Thanks to everyone that has provided feedback and improvement suggestions (andeh, bps, circle, hangtime, milton, splash,
wimpeeh) among others.

## Related projects

* [ezQuake](https://github.com/ezQuake/ezquake-source) - QuakeWorld client
* [qw-streambot-ezquake](https://github.com/vikpe/qw-streambot-ezquake) - ezQuake setup used
  by [twitch.tv/QuakeWorld](https://twitch.tv/QuakeWorld)
* [serverstat](https://github.com/vikpe/serverstat) - Get info from QuakeWorld servers.
* [masterstat](https://github.com/vikpe/masterstat) - Get server addresses from QuakeWorld master servers.
* [QuakeWorld Server Browser](https://github.com/vikpe/qw-server-browser)
* [carmack](https://github.com/vikpe/carmack) - QuakeWorld utility bot for Discord
