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
- Run `docker-compose pull`. This will download the latest built Docker images from Dockerhub that are required to run AutoMuteUs.
- Run `docker-compose up -d` to start all the containers required for AutoMuteUs to function. The containers will now be running in the background, but you can view the logs for the containers using `docker-compose logs`, or `docker-compose logs -f` to follow along as new log entries are generated.

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
4. Use the command `docker-compose build` to build the set of docker images with your change
5. Start the stack with `docker-compose up`

Just remember that you will need to do a rebuild of the docker images every time you make a change.

## Environment Variables

### Required

- `DISCORD_BOT_TOKEN`: The Bot Token used by the bot to authenticate with Discord.
- `REDIS_ADDR`: The host and port at which your Redis database instance is accessible. Ex: `192.168.1.42:6379`
- `POSTGRES_ADDR`: Address (host:port) at which Postgres is accessible. Used by automuteus to store game statistics. 
- `POSTGRES_USER`: Username for authentication with Postgres.
- `POSTGRES_PASS`: Password for authentication with Postgres.
- `GALACTUS_ADDR`: Address at which Galactus is accessible. Typically something like `http://localhost:5858` (or see docker-compose.yml)

### Optional

- `WORKER_BOT_TOKENS`: A comma-separated list of extra tokens to be used for mute/deafen. (Sent to Galactus, and stored in Redis after first start-up)
- `EMOJI_GUILD_ID`: If your bot is a member of multiple guilds, this ID can be used to specify the single guild that it should use for emojis (no need to add the emojis to ALL servers).
- `HOST`: The **externally-accessible URL** for Galactus. For example, `http://test.com:8123`.
  This is used to provide the linking URI to the capture, via the Direct Message the bot sends you when typing `.au new`.
  **You must specify `http://` or `https://` accordingly, and specify the port if non-8123. For example, `https://your-app.herokuapp.com:443`**
- `LOCALE_PATH`: Path to localization files.
- `LOG_PATH`: Filesystem path for log files. Defaults to `./`
- `CAPTURE_TIMEOUT`: How many seconds of no capture events received before the Bot will terminate the associated game/connection. Defaults to 36000 seconds.
- `REDIS_PASS`: Your Redis database password, if necessary.
- `AUTOMUTEUS_LISTENING`: What the bot displays it is "Listening to" in the online presence message. Recommend putting your custom command prefix here
- `AUTOMUTEUS_GLOBAL_PREFIX`: A universal default for the bot's command prefix. The bot will respond to **both** this prefix, and any guild-specific prefixes set in settings.
- `BASE_MAP_URL`: The URL used as the base for the map images used in lobby message and response to `.au map`. The actual URLs will be constructed as the concatenation of the following strings: `BASE_MAP_URL`, map name (`the_skeld`, `mira_hq`, `polus`, or `airship`), version (`_detailed` for detailed version only), extension and query string (`.png?raw=true`)
- `SLASH_COMMAND_GUILD_IDS`: When registering slash commands, what guilds the interactions will be registered in. Multiple guild IDs can be specified by comma-separated list. Leave blank for global.
- `STOP_GRACE_PERIOD`: Specify how long to wait when attempting to stop `automuteus` container before sending SIGKILL. This option prevents the container from exiting with a `SIGKILL` during the stopping process before the command deletion is complete. When using guild commands, about one minute per guild is sufficient. Defaults to `1m`.

### HIGHLY advanced. Probably don't ever touch these!

- `NUM_SHARDS`: Num shards provided to the Discord API.
- `SHARD_ID`: Shard ID used to identify with the Discord API. Needs to be strictly less than `NUM_SHARDS`

## Galactus

Galactus is a program used to speed up muting and deafening (which is typically constrained by Discord rate-limits). It allows for an arbitrary number of tokens to be provided for faster muting/deafening, but also supports capture-side bots.
A guide to setup your own capture-side bot can be found [here,](https://youtu.be/jKcEW5qpk8E) and the repo for Galactus is [here.](https://github.com/automuteus/galactus)
