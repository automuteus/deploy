# Instructions and Resources for Deploying and Developing AutoMuteUs

If you would prefer to self-host the bot, the steps for doing so are provided below. Self-hosting requires robust knowledge and troubleshooting capability for Docker/Docker-compose and/or any other networking and routing config specific to your hosting solution.

## Architecture

[ARCHITECTURE.md](ARCHITECTURE.md) has a diagram of the containers in this stack, what each one talks to, and how a
game flows from `/new` through the capture client to the bot.

## Pre-Installation Steps, Important!

- Create an Application and Bot account (requires Admin privileges on the Server in question). [Instructions here](BOT_README.md)

Now follow any of the specific hosting options provided below:

## Docker Compose:

Docker compose is the simplest and recommended method for self-hosting AutoMuteUs, but it does require an existing physical machine or VPS to run on.

There is a [`docker-compose.yml`](docker-compose.yml) file in this repository that will provide all the constituent components to run AutoMuteUs.

The HTTP API now runs in its own `api` container, published as `automuteus/api`.
Its image tag defaults to `AUTOMUTEUS_TAG`; `API_TAG` can override it. Use a release
that includes the standalone API image. Existing API URLs and `API_PORT` still
work: Compose now forwards that port to the API container instead of the bot.
The API needs Redis and Postgres, but no Discord token or running bot process.
For the first upgrade, run `docker compose stop automuteus`, then
`docker compose up -d`. This releases the old bot container's published API port
before the new API container binds it. Later API updates can use
`docker compose up -d api` independently of the bot.

The stack includes the web dashboard from [automuteus/web](https://github.com/automuteus/web), which is where guild
settings are managed (with Discord sign-in). It needs `NEXTAUTH_SECRET`, `DISCORD_CLIENT_ID`, and
`DISCORD_CLIENT_SECRET` in your `.env` (see `sample.env`) and the OAuth2 redirect registered in your Discord
application; the stack will not start without them. It is published on `WEB_PORT` (default 3000) and talks to the API
over the compose network, so it needs no other configuration.

### Steps:

- Install [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) on the machine you will be using to host AutoMuteUs
- Download the [`docker-compose.yml`](docker-compose.yml) from this repository, and create a `.env` file in the same directory that will contain your Environment Variables. On Linux/UNIX systems you can use `touch .env` to create this file, but a template [`sample.env`](sample.env) is provided in this repository for reference. 
- Provide your specific Environment Variables in the `.env` file, as relevant to your configuration. Please see the Environment Variables reference further down in this Readme for details, as well as the [`sample.env`](sample.env) provided.
- Run `docker compose pull`. This will download the latest built Docker images from Dockerhub that are required to run AutoMuteUs.
- Run `docker compose up -d` to start all the containers required for AutoMuteUs to function. The containers will now be running in the background, but you can view the logs for the containers using `docker compose logs`, or `docker compose logs -f` to follow along as new log entries are generated.

## Development Instructions

The easiest way to test changes is to run the same `docker compose` stack, but build the `automuteus`, `galactus`, `api`,
and `web` images from source instead of pulling them. Docker Compose merges a `docker-compose.override.yml` file into
`docker-compose.yml` automatically if one exists, and a ready-made override is provided:

1. Clone [automuteus/automuteus](https://github.com/automuteus/automuteus) next to this `deploy` repository, so the
   two directories are siblings. The Bot, Galactus, and the API are built from that one repository. Also clone
   [automuteus/web](https://github.com/automuteus/web) as a sibling for the dashboard.
2. Copy the sample override into place (it is gitignored, so edit it freely):

   ```bash
   cp docker-compose.override.sample.yml docker-compose.override.yml
   ```

   If your checkout lives somewhere else, change the `context` paths in the copy.
3. Make any changes to the code or sql file that you would like.
4. Build and start the stack:

   ```bash
   docker compose up --build
   ```

Just remember to pass `--build` (or run `docker compose build`) every time you make a change, so the images are
rebuilt. Delete `docker-compose.override.yml` to go back to the published images.

The local API is available at `http://localhost:8080`. To use another port, set
`API_PORT=9090` in `.env` and run `docker compose up -d` to recreate the API and bot
containers. With `API_SERVER_URL` left blank, capture links will then use
`http://localhost:9090/open/link`. If capture runs on another machine, also set
`API_SERVER_URL` to the API's reachable URL, including the port.

## Upgrading from v8 (guild settings)

Up to v8, guild settings (language, delays, voice rules, and so on) were stored in Redis. They now live in Postgres,
and **9.2.x is the last release that can move them**. 10.0 and later never read settings from Redis, so jumping
straight from v8 to 10 resets every server to the default settings. Go through 9.2.0 first:

1. Set `AUTOMUTEUS_TAG=9.2.0` in `.env`, then run `docker compose pull` and `docker compose up -d`. The bot now moves
   each server's settings into Postgres the first time that server uses it.
2. Move the servers that have not used the bot yet with the one-off sweep included in the 9.2.0 image. The dry run
   only checks the records and writes nothing:

   ```bash
   docker compose run --rm --no-deps --entrypoint ./migrate-guild-settings automuteus --dry-run
   docker compose run --rm --no-deps --entrypoint ./migrate-guild-settings automuteus
   ```

   Both print a JSON report. The sweep is safe to run while the bot is up, and it never overwrites settings already
   in Postgres. Records it cannot read are listed under `failures` and left in Redis; they are the only settings
   that will not carry over.
3. Run the sweep again until it reports `"examined": 0`, then upgrade to 10.0 or later as usual.

## Upgrading Postgres

The `docker-compose.yml` file now runs `postgres:18-alpine`; earlier versions of this file ran `postgres:12-alpine`.
Postgres cannot open a data directory created by an older major version, so **if you already have a running
installation, pulling the new compose file and running `docker compose up` will leave the `postgres` container
crash-looping** with an error about incompatible database files (or, on 18+, a message about `pg_upgrade`). Nothing
is deleted when that happens, but the bot will not start until you migrate.

Postgres only stores game statistics and premium records; guild settings live in Redis and are not affected. If you
do not care about keeping historical stats, the quickest upgrade is to delete the Postgres volume (step 4 below) and
skip the backup and restore steps. The bot recreates an empty schema on startup.

To keep your data, dump it with the old version and restore it into the new one:

1. **While still running the old compose file**, back up the database (this uses the database name that the
   `postgres` image creates from `POSTGRES_USER`):

   ```bash
   docker compose exec postgres pg_dump -U "$POSTGRES_USER" --clean --if-exists -f /tmp/automuteus.sql
   docker compose cp postgres:/tmp/automuteus.sql ./automuteus.sql
   ```

   On Windows, replace `"$POSTGRES_USER"` with the actual value from your `.env` file. Writing the dump inside the
   container and copying it out avoids shell redirection re-encoding the file.

2. Stop the stack. Do **not** use `docker compose down -v`, which would also delete the Redis volume and with it all
   of your guild settings:

   ```bash
   docker compose down
   ```

3. Update `docker-compose.yml` to the new version from this repository.

4. Delete the old Postgres volume. Its name is the compose project name (by default the directory name) followed by
   `_postgres-data`; `docker volume ls` will show it:

   ```bash
   docker volume rm deploy_postgres-data
   ```

5. Start only Postgres so it can initialize a fresh data directory, then restore the dump into it:

   ```bash
   docker compose up -d postgres
   docker compose cp ./automuteus.sql postgres:/tmp/automuteus.sql
   docker compose exec postgres psql -U "$POSTGRES_USER" -f /tmp/automuteus.sql
   ```

   On first start the image initializes the data directory using a temporary server, then restarts. Wait until
   `docker compose logs postgres` shows `database system is ready to accept connections` for the second time
   before restoring; if you are too early you will see a "connection refused" error, and can simply retry.

6. Start everything else:

   ```bash
   docker compose up -d
   ```

Because the 18+ images keep each major version's data in its own directory under `/var/lib/postgresql`, future
major upgrades can be done in place with `pg_upgrade --link` on the same volume, without repeating this procedure.

## Environment Variables

### Required

- `DISCORD_BOT_TOKEN`: The Bot Token used by the bot to authenticate with Discord.
- `POSTGRES_USER`: Username for authentication with Postgres.
- `POSTGRES_PASS`: Password for authentication with Postgres.
- `NEXTAUTH_SECRET`: Random secret that encrypts the web dashboard's session cookies, e.g. `openssl rand -base64 32`.
- `DISCORD_CLIENT_ID` / `DISCORD_CLIENT_SECRET`: OAuth2 credentials of your Discord application, used by the dashboard
  for sign-in and to build bot invite links. The bot's own application can be used. Register
  `<WEB_URL>/api/auth/callback/discord` (by default `http://localhost:3000/api/auth/callback/discord`) as a
  redirect in the application's OAuth2 settings.
- `GALACTUS_HOST`: The **externally-accessible URL** for Galactus. 
  For example, if you only intend on running the capture from the same machine that runs AutoMuteUs, use `http://localhost:8123`.
  This is the URL the bot sends as a response to `/new` in order to link capture clients to Galactus, so it needs to be accessible to any users wishing to run capture clients.
  **You must specify `http://` or `https://` accordingly, and specify the port if non-8123. For example, `https://your-app.herokuapp.com:443`**

### Optional
- `WORKER_BOT_TOKENS`: A comma-separated list of extra tokens to be used for mute/deafen.
- `EMOJI_GUILD_ID`: If your bot is a member of multiple guilds, this ID can be used to specify the single guild that it should use for emojis (no need to add the emojis to ALL servers).
- `CAPTURE_TIMEOUT`: How many seconds of no capture events received before the Bot will terminate the associated game/connection. Defaults to 36000 seconds.
- `REDIS_PASS`: Your Redis database password, if necessary.
- `AUTOMUTEUS_LISTENING`: What the bot displays it is "Listening to" in the online presence message. Defaults to `/help`
- `SLASH_COMMAND_GUILD_IDS`: When registering slash commands, what guilds the interactions will be registered in. Multiple guild IDs can be specified by comma-separated list. Leave blank to register commands globally.
- `LOG_FORMAT`: `text` (default) or `json`. Applies to the bot, API, and Galactus. JSON pairs well with `docker compose logs | jq` for filtering by `guild` or `code`.
- `LOG_LEVEL`: `debug`, `info` (default), `warn`, or `error`. `debug` shows every individual mute/deafen request; `info` shows one line per batch.

 
### Optional and Advanced
- `BASE_MAP_URL`: The URL used as the base for the map images used in lobby message and response to `/map`. The actual URLs will be constructed as the concatenation of the following strings: `BASE_MAP_URL`, map name (`the_skeld`, `mira_hq`, `polus`, or `airship`), version (`_detailed` for detailed version only), and extension (`.png`). Defaults to `https://raw.githubusercontent.com/automuteus/automuteus/refs/heads/master/assets/maps/`.-
- `STOP_GRACE_PERIOD`: Specify how long to wait when attempting to stop `automuteus` container before sending SIGKILL. This option prevents the container from exiting with a `SIGKILL` during the stopping process before the command deletion is complete. When using guild commands, about one minute per guild is sufficient. Defaults to `2m` (2 minutes) for safety.
- `API_PORT`: Public host port for the separate API container. Defaults to `8080`; set `80` to keep the previous default. `SERVICE_PORT` selects the API's internal listening port (default `5000`).
- `API_TAG`: Optional API image version override; defaults to `AUTOMUTEUS_TAG`.
- `API_SERVER_URL`: Public API URL (including scheme and any nonstandard port), used for capture links and Swagger Docs. Defaults to `http://localhost:${API_PORT:-8080}` in Compose, so changing `API_PORT` also updates capture links. Set this explicitly when capture runs on another machine or the API is behind a reverse proxy.
- `API_ADMIN_PASS`: Password for the API's `admin` account. Defaults to `automuteus`, which the API treats as unset:
  it never accepts the admin credential while the password is blank or default. A non-default value is required for
  raising or clearing platform notices via `POST`/`DELETE /admin/notice` (a banner on every game's status message,
  or ending every game for maintenance), and for the dashboard's admin stats view (see `ADMIN_USER_IDS`).
- `DRAIN_SECONDS`: How long Galactus keeps running after a stop signal, refusing new capture clients, before telling the bot to end the games whose captures were connected to it and exiting. Defaults to `5`. Compose gives the container 30 seconds to complete this.

### Web dashboard
- `WEB_URL`: The dashboard's public URL, which the bot's `/settings` command links to and the dashboard uses for
  sign-in redirects. Defaults to `http://localhost:${WEB_PORT:-3000}`, which only works for people on the host
  machine. Set it when the dashboard is exposed or behind a reverse proxy, and update the OAuth2 redirect in the
  Discord application to match.
- `WEB_PORT`: Public host port for the dashboard. Defaults to `3000`.
- `WEB_TAG`: Dashboard image version. Defaults to `latest`; the dashboard is released separately from the bot.
- `ADMIN_USER_IDS`: Comma-separated Discord user IDs of operators who may open any server's stats pages on the
  dashboard by ID, including servers they are not a member of. Only takes effect when `API_ADMIN_PASS` is also set
  to a non-default value; leave unset to disable. For a listed user, the dashboard forwards the read-only stats,
  match, player, and bot presence routes with the API's admin credential instead of the user's Discord session, and
  shows the leaderboards regardless of the server's premium. Settings, stats resets, and premium always use the
  user's own Discord permissions, so this cannot change a server. The pages show an admin-view notice, and the API
  logs each admin read with the user and server.

### HIGHLY advanced. Probably don't ever touch these!

- `REDIS_ADDR`: The host and port at which your Redis database instance is accessible. Ex: `redis:6379`
- `POSTGRES_ADDR`: Address (host:port) at which Postgres is accessible. Used by automuteus to store game statistics. 
- `SHARDS`: Comma-separated list of shards to use for this bot instance. For example, `0,1,2` runs shards 0, 1, and 2. *Order is important*, as `0,1` is functionally different than `1,0` (the former would register commands if ran as the official bot, whereas the latter would not).
- `NUM_SHARDS`: Sum of how many shards are being ran across all instances. This needs to be strictly GREATER than than the max value provided for `SHARDS`. For example, if running two bot instances with `0,1` and `2,3` provided for `SHARDS` to the instances, `NUM_SHARDS` should be `4` for both instances (max + 1).

## Galactus

Galactus is the message broker for information sent from capture clients. It lives in the
[automuteus/automuteus](https://github.com/automuteus/automuteus) repository under `cmd/galactus` and is released
alongside the bot under the same version tag, which is why `AUTOMUTEUS_TAG` selects both images.

Stopping or restarting the Galactus container (`docker compose restart galactus`, `docker compose down`, or an
image upgrade) severs every capture connection. Since v9, Galactus announces this to the bot before it exits: any
running games are ended, everyone is unmuted and undeafened, and players are told to run `/new` once it is back.
Matches ended this way are recorded as aborted and do not count toward statistics. Bring Galactus back up first when
upgrading, so `/new` works as soon as the bot follows.
