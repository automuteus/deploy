# Instructions and Resources for Deploying and Developing AutoMuteUs

If you would prefer to self-host the bot, the steps for doing so are provided below. Self-hosting requires robust knowledge and troubleshooting capability for Docker/Docker-compose and/or any other networking and routing config specific to your hosting solution.

## Pre-Installation Steps, Important!

- Create an Application and Bot account (requires Admin privileges on the Server in question). [Instructions here](BOT_README.md)

Now follow any of the specific hosting options provided below:

## Docker Compose:

Docker compose is the simplest and recommended method for self-hosting AutoMuteUs, but it does require an existing physical machine or VPS to run on.

There is a [`docker-compose.yml`](docker-compose.yml) file in this repository that will provide all the constituent components to run AutoMuteUs.

### Steps:

- Install [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) on the machine you will be using to host AutoMuteUs
- Download the [`docker-compose.yml`](docker-compose.yml) from this repository, and create a `.env` file in the same directory that will contain your Environment Variables. On Linux/UNIX systems you can use `touch .env` to create this file, but a template [`sample.env`](sample.env) is provided in this repository for reference. 
- Provide your specific Environment Variables in the `.env` file, as relevant to your configuration. Please see the Environment Variables reference further down in this Readme for details, as well as the [`sample.env`](sample.env) provided.
- Run `docker compose pull`. This will download the latest built Docker images from Dockerhub that are required to run AutoMuteUs.
- Run `docker compose up -d` to start all the containers required for AutoMuteUs to function. The containers will now be running in the background, but you can view the logs for the containers using `docker compose logs`, or `docker compose logs -f` to follow along as new log entries are generated.

## unRAID

unRAID hosting steps are are not yet updated for v3.0+ of AutoMuteUs, and as such is not supported at this time.

## Heroku

Heroku hosting steps are are not yet updated for v3.0+ of AutoMuteUs, and as such is not supported at this time.

## FreeBSD

AutoMuteUs exists in the FreeBSD Ports tree as [`games/automuteus`](https://www.freshports.org/games/automuteus/). Instructions are included in the Port.

## Old version
If, for whatever reason, you _really_ want to self host, but also don't want to figure out Docker or use Windows and hate Docker because of it (I don't blame you) you can self host [2.4.3](https://github.com/denverquane/automuteus/releases/tag/2.4.3) instead. **If you are using this method, continue using the newest capture!** But note that 2.4.3 does not support 15 players' lobby and new player colors!

## Development Instructions
The easiest way to test changes is to use docker-compose, but instead of using a pre-built image, building the automuteus docker image from source. Thankfully, this is easy to do:

1. Clone [automuteus/automuteus](https://github.com/automuteus/automuteus) next to this `deploy` repository.
2. Make any changes to the code or sql file that you would like.
3. In the `docker-compose.yml` comment out the line `image: automuteus/automuteus:${AUTOMUTEUS_TAG:?err}` and uncomment the `build: ../automuteus` line (and modify path if required).
4. Use the following command to build the set of docker images with your change

   ```bash
   COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_BUILDKIT=1 docker compose build
   ```

5. Start the stack with `docker compose up`

Just remember that you will need to do a rebuild of the docker images every time you make a change.

## Environment Variables

### Required

- `DISCORD_BOT_TOKEN`: The Bot Token used by the bot to authenticate with Discord.
- `POSTGRES_USER`: Username for authentication with Postgres.
- `POSTGRES_PASS`: Password for authentication with Postgres.
- `GALACTUS_HOST`: The **externally-accessible URL** for Galactus. 
  For example, if you only intend on running the capture from the same machine that runs AutoMuteUs, use `http://localhost:8123`.
  This is the URL the bot sends as a response to `/new` in order to link capture clients to Galactus, so it needs to be accessible to any users wishing to run capture clients.
  **You must specify `http://` or `https://` accordingly, and specify the port if non-8123. For example, `https://your-app.herokuapp.com:443`**

### Optional
- `API_PORT`: Port on which the AutoMuteUs API will be accessible. Defaults to `5000`
- `WORKER_BOT_TOKENS`: A comma-separated list of extra tokens to be used for mute/deafen.
- `EMOJI_GUILD_ID`: If your bot is a member of multiple guilds, this ID can be used to specify the single guild that it should use for emojis (no need to add the emojis to ALL servers).
- `CAPTURE_TIMEOUT`: How many seconds of no capture events received before the Bot will terminate the associated game/connection. Defaults to 36000 seconds.
- `REDIS_PASS`: Your Redis database password, if necessary.
- `AUTOMUTEUS_LISTENING`: What the bot displays it is "Listening to" in the online presence message. Defaults to `/help`
- `BASE_MAP_URL`: The URL used as the base for the map images used in lobby message and response to `/map`. The actual URLs will be constructed as the concatenation of the following strings: `BASE_MAP_URL`, map name (`the_skeld`, `mira_hq`, `polus`, or `airship`), version (`_detailed` for detailed version only), extension and query string (`.png?raw=true`)
- `SLASH_COMMAND_GUILD_IDS`: When registering slash commands, what guilds the interactions will be registered in. Multiple guild IDs can be specified by comma-separated list. Leave blank for global.
- `STOP_GRACE_PERIOD`: Specify how long to wait when attempting to stop `automuteus` container before sending SIGKILL. This option prevents the container from exiting with a `SIGKILL` during the stopping process before the command deletion is complete. When using guild commands, about one minute per guild is sufficient. Defaults to `2m` (2 minutes) for safety.

### HIGHLY advanced. Probably don't ever touch these!

- `REDIS_ADDR`: The host and port at which your Redis database instance is accessible. Ex: `redis:6379`
- `POSTGRES_ADDR`: Address (host:port) at which Postgres is accessible. Used by automuteus to store game statistics. 
- `SHARDS`: Comma-separated list of shards to use for this bot instance. For example, `0,1,2` runs shards 0, 1, and 2. *Order is important*, as `0,1` is functionally different than `1,0` (the former would register commands if ran as the official bot, whereas the latter would not).
- `NUM_SHARDS`: Sum of how many shards are being ran across all instances. This needs to be strictly GREATER than than the max value provided for `SHARDS`. For example, if running two bot instances with `0,1` and `2,3` provided for `SHARDS` to the instances, `NUM_SHARDS` should be `4` for both instances (max + 1).

## Galactus

Galactus is the message broker for information sent from capture clients. The repo for Galactus can be found [here](https://github.com/automuteus/galactus)
