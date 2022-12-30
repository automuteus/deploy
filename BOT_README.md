# Create an Application and Bot account

1. Navigate to <https://discord.com/developers/applications> and create New Application (top right). Name it whatever you like.

2. Click "Bot" on the left panel, then click the button on the right to Add Bot.

3. Scroll up to where the Bot Icon is displayed. **Copy the `Token` on the right, and paste it to a safe location.** We will need it later in the installation steps; this is the `DISCORD_BOT_TOKEN` in the `.env` file (or the `config.txt` for the older version).

4. On the left panel, click `URL Generator` under `OAuth2`, and then check the boxes marked `bot` (required) and `applications.commands` (for the first bot only) under `SCOPES`. Then scroll down to `BOT PERMISSIONS`, and check the boxes as shown in the table below.

5. Scroll down to `GENERATED URL`, and copy the URL in the field that begins with `https://discord.com/api/oauth2/authorize?`. Paste this in a new browser tab, and grant the App access to whatever server you wish it to access. Close this tab when Finalized.

## Bot Permissions

### For the first bot

- **SCOPES**
  - ✅ bot
  - ✅ applications.commands
- **BOT PERMISSIONS**
  | GENERAL PERMISSIONS                  | TEXT PERMISSIONS           | VOICE PERMISSIONS         |
  | ------------------------------------ | -------------------------- | ------------------------- |
  | ⬜ Administrator                      | ✅ Send Messages            | ⬜ Connect                 |
  | ⬜ View Audit Log                     | ⬜ Create Public Threads    | ⬜ Speak                   |
  | ⬜ Manage Server                      | ⬜ Create Private Threads   | ⬜ Video                   |
  | ⬜ Manage Roles                       | ⬜ Send Messages in Threads | ✅ Mute Members            |
  | ⬜ Manage Channels                    | ⬜ Send TTS Messages        | ✅ Deafen Members          |
  | ⬜ Kick Members                       | ✅ Manage Messages          | ⬜ Move Members            |
  | ⬜ Ban Members                        | ⬜ Manage Threads           | ⬜ Use Voice Activity      |
  | ⬜ Create Instant Invite              | ✅ Embed Links              | ⬜ Priority Speaker        |
  | ⬜ Change Nickname                    | ✅ Attach Files             | ⬜ Request To Speak        |
  | ⬜ Manage Nicknames                   | ✅ Read Message History     | ⬜ Use Embedded Activities |
  | ✅ Manage Emojis and Stickers         | ⬜ Mention Everyone         |                           |
  | ⬜ Manage Webhooks                    | ✅ Use External Emojis      |                           |
  | ✅ Read Messages/View Channels        | ⬜ Use External Stickers    |                           |
  | ⬜ Manage Events                      | ✅ Add Reactions            |                           |
  | ⬜ Moderate Members                   | ⬜ Use Slash Commands       |                           |
  | ⬜ View Server Insights               |                            |                           |
  | ⬜ View Creator Monetization Insights |                            |                           |

Permissions Integer: `1086712896`

### For the extra worker bot

- **SCOPES**
  - ✅ bot
- **BOT PERMISSIONS**
  | GENERAL PERMISSIONS                  | TEXT PERMISSIONS           | VOICE PERMISSIONS         |
  | ------------------------------------ | -------------------------- | ------------------------- |
  | ⬜ Administrator                      | ⬜ Send Messages            | ⬜ Connect                 |
  | ⬜ View Audit Log                     | ⬜ Create Public Threads    | ⬜ Speak                   |
  | ⬜ Manage Server                      | ⬜ Create Private Threads   | ⬜ Video                   |
  | ⬜ Manage Roles                       | ⬜ Send Messages in Threads | ✅ Mute Members            |
  | ⬜ Manage Channels                    | ⬜ Send TTS Messages        | ✅ Deafen Members          |
  | ⬜ Kick Members                       | ⬜ Manage Messages          | ⬜ Move Members            |
  | ⬜ Ban Members                        | ⬜ Manage Threads           | ⬜ Use Voice Activity      |
  | ⬜ Create Instant Invite              | ⬜ Embed Links              | ⬜ Priority Speaker        |
  | ⬜ Change Nickname                    | ⬜ Attach Files             | ⬜ Request To Speak        |
  | ⬜ Manage Nicknames                   | ⬜ Read Message History     | ⬜ Use Embedded Activities |
  | ⬜ Manage Emojis and Stickers         | ⬜ Mention Everyone         |                           |
  | ⬜ Manage Webhooks                    | ⬜ Use External Emojis      |                           |
  | ⬜ Read Messages/View Channels        | ⬜ Use External Stickers    |                           |
  | ⬜ Manage Events                      | ⬜ Add Reactions            |                           |
  | ⬜ Moderate Members                   | ⬜ Use Slash Commands       |                           |
  | ⬜ View Server Insights               |                            |                           |
  | ⬜ View Creator Monetization Insights |                            |                           |

Permissions Integer: `12582912`
