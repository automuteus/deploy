# Create an Application and Bot account

1. Navigate to https://discord.com/developers/applications and create New Application (top right). Name it whatever you like.

2. Click "Bot" on the left panel, then click the button on the right to Add Bot.

3. Scroll up to where the Bot Icon is displayed. **Copy the `Token` on the right, and paste it to a safe location.** We will need it later in the installation steps; this is the `DISCORD_BOT_TOKEN` in the `.env` file (or the `config.txt` for the older version).

4. On the left panel, click "OAuth2", and then check the box marked `bot` under `Scopes`. Then scroll down to `Bot Permissions`, Then scroll down to `BOT PERMISSIONS`, and check the boxes as shown in the table below.

5. Scroll back up to `Scopes`, and copy the URL in the field that begins with `https://discord.com/api/oauth2/authorize?`. Paste this in a new browser tab, and grant the App access to whatever server you wish it to access. Close this tab when Finalized.

## Bot Permissions

### For the first bot

| GENERAL PERMISSIONS     | TEXT PERMISSIONS       | VIOCE PERMISSIONS    |
| ----------------------- | ---------------------- | -------------------- |
| ⬜ Administrator         | ✅ Send Messages        | ⬜ Connect            |
| ⬜ View Audit Log        | ⬜ Send TTS Messages    | ⬜ Speak              |
| ⬜ View Server Insights  | ✅ Manage Messages      | ⬜ Video              |
| ⬜ Manage Server         | ✅ Embed Links          | ✅ Mute Members       |
| ⬜ Manage Roles          | ⬜ Attach Files         | ✅ Deafen Members     |
| ⬜ Manage Channels       | ✅ Read Message History | ⬜ Move Members       |
| ⬜ Kick Members          | ⬜ Mention Everyone     | ⬜ Use Voice Activity |
| ⬜ Ban Members           | ✅ Use External Emojis  | ⬜ Priority Speaker   |
| ⬜ Create Instant Invite | ✅ Add Reactions        |                      |
| ⬜ Change Nickname       | ⬜ Use Slash Commands   |                      |
| ⬜ Manage Nicknames      |                        |                      |
| ✅ Manage Emojis         |                        |                      |
| ⬜ Manage Webhooks       |                        |                      |
| ✅ View Channels         |                        |                      |

Permissions Integer: `1086680128`

### For the extra worker bot

| GENERAL PERMISSIONS     | TEXT PERMISSIONS       | VIOCE PERMISSIONS    |
| ----------------------- | ---------------------- | -------------------- |
| ⬜ Administrator         | ⬜ Send Messages        | ⬜ Connect            |
| ⬜ View Audit Log        | ⬜ Send TTS Messages    | ⬜ Speak              |
| ⬜ View Server Insights  | ⬜ Manage Messages      | ⬜ Video              |
| ⬜ Manage Server         | ⬜ Embed Links          | ✅ Mute Members       |
| ⬜ Manage Roles          | ⬜ Attach Files         | ✅ Deafen Members     |
| ⬜ Manage Channels       | ⬜ Read Message History | ⬜ Move Members       |
| ⬜ Kick Members          | ⬜ Mention Everyone     | ⬜ Use Voice Activity |
| ⬜ Ban Members           | ⬜ Use External Emojis  | ⬜ Priority Speaker   |
| ⬜ Create Instant Invite | ⬜ Add Reactions        |                      |
| ⬜ Change Nickname       |                        |                      |
| ⬜ Manage Nicknames      |                        |                      |
| ⬜ Manage Emojis         |                        |                      |
| ⬜ Manage Webhooks       |                        |                      |
| ⬜ View Channels         |                        |                      |

Permissions Integer: `12582912`
