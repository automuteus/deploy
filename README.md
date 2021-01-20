# deploy
Instructions and Resources for Deploying and Developing AutoMuteUs

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
If, for whatever reason, you _really_ want to self host, but also don't want to figure out Docker or use Windows and hate Docker because of it (I don't blame you) you can self host [2.4.3](https://github.com/denverquane/automuteus/releases/tag/2.4.3) instead. **If you are using this method, continue using the newest capture!**

## Development Instructions
The easiest way to test changes is to use docker-compose, but instead of using a pre-built image, building the automuteus docker image from source. Thankfully, this is easy to do:

1. In the `docker-compose.yml` comment out the line `image: denverquane/amongusdiscord:${AUTOMUTEUS_TAG:?err}` and uncomment the `build .` line.
2. Make any changes to the code or sql file that you would like.
3. Use the command `docker-compose build` to build the set of docker images with your change
4. Start the stack with `docker-compose up`

Just remember that you will need to do a rebuild of the docker images every time you make a change.

## Environment Variables

### Required

- `GALACTUS_ADDR`: Address at which Galactus is accessible. Typically something like `http://localhost:5858` (or see docker-compose.yml)
- `REDIS_ADDR`: The host and port at which your Redis database instance is accessible. Ex: `192.168.1.42:6379`
- `POSTGRES_ADDR`: Address (host:port) at which Postgres is accessible. Used by automuteus to store game statistics. 
- `POSTGRES_USER`: Username for authentication with Postgres.
- `POSTGRES_PASS`: Password for authentication with Postgres.

### Optional

- `EMOJI_GUILD_ID`: If your bot is a member of multiple guilds, this ID can be used to specify the single guild that it should use for emojis (no need to add the emojis to ALL servers).
- `HOST`: The **externally-accessible URL** for Galactus. For example, `http://test.com:8123`.
  This is used to provide the linking URI to the capture, via the Direct Message the bot sends you when typing `.au new`.
  **You must specify `http://` or `https://` accordingly, and specify the port if non-8123. For example, `https://your-app.herokuapp.com:443`**
- `LOCALE_PATH`: Path to localization files. Defaults to ./locales
- `LOG_PATH`: Filesystem path for log files. Defaults to `./`
- `CAPTURE_TIMEOUT`: How many seconds of no capture events received before the Bot will terminate the associated game/connection. Defaults to 36000 seconds.
- `REDIS_PASS`: Your Redis database password, if necessary.
- `AUTOMUTEUS_LISTENING`: What the bot displays it is "Listening to" in the online presence message. Recommend putting your custom command prefix here
- `AUTOMUTEUS_GLOBAL_PREFIX`: A universal default for the bot's command prefix. The bot will respond to **both** this prefix, and any guild-specific prefixes set in settings.

## Galactus

Galactus is a program used to speed up muting and deafening (which is typically constrained by Discord rate-limits). It allows for an arbitrary number of tokens to be provided for faster muting/deafening, but also supports capture-side bots.
A guide to setup your own capture-side bot can be found [here,](https://youtu.be/jKcEW5qpk8E) and the repo for Galactus is [here.](https://github.com/automuteus/galactus)
