# Setup

## Requirements
- Python >= 3.12
- [uv package manager](https://docs.astral.sh/uv/)

## Environment Setup
Clone this repo, and navigate into the directory:
```
git clone https://github.com/metagov/telescope-koi-network
cd telescope-koi-network
```
Initialize virtualenv and follow instructions to activate environment:
```
uv venv
```
Sync dependencies:
```
uv sync
```
Rename `.env.example` to `.env`, and set the `PRIV_KEY_PASSWORD`. This is used to encrypt node private keys on disk, any string will work. 

## Slack Bot Setup

1. Navigate to ["Your Apps"](https://api.slack.com/apps) through the Slack API portal.
2. Click `Create New App`, `From a manifest`, and then the `Continue` button.
3. Select the `YAML` tab at the top of the text box, and copy in the [Slack Telescope app manifest](https://raw.githubusercontent.com/metagov/slack-telescope/refs/heads/main/app_manifest.yaml).
4. Select your workspace from the dropdown, and then click the `Next` button.
5. Finally press `Create and Install`, and then `Allow` in the pop up window.
6. Copy down the `Bot token` and `App token` from `Your app credentials`.
7. Click `Go to App Settings`, if you see a pop up that permission scopes have changed, follow the instructions and `reinstall your app`.
8. Fill in the the environment variables for the Slack bot in the `.env` file:
    1. `SLACK_BOT_TOKEN` - previously copied `Bot token`
    2. `SLACK_USER_TOKEN` - navigate to `Install App`, and copy the `User OAuth Token`
    3. `SLACK_SIGNING_SECRET` - navigate to `Basic information`, and copy the `Signing Secret`
    4. `SLACK_APP_TOKEN` - previously copied `App token`


9. Launch the KOI shell from the terminal with `koi-sh`.
10. Within the shell, run the sync command to setup all of the required nodes for the Telescope system: `network sync`. Then quit the shell with `quit`.
11. You can now do manual config of the Telescope node, by editing `slack_telescope/config.yaml`. *Note: the config should only be edited while the network is not running, otherwise your changes will not take effect and be overwritten.*
    1. You may want to update the messages used throughout the consent process to tailor them to your specific usage. Modify any of the text blocks in `telescope -> messages`.
    2. By default, the Telescope uses the `🔭` emoji to tag messages. You can override this by updating `telescope -> emoji`. The string here is based on Slack's internal emoji names, which you can find by highlighting the emoji of interest in the emoji picker, and copying the text between the colons, e.g., `:eyes: -> eyes`.
    3. Users can retract approved messages with a certain time period. This can be configured in `telescope -> retraction_time_limit_days`, which defaults to 28 days. After this time limit is exceeded, users attempting retraction will be notified that the time limit is passed, and directed to reach out to researchers directly.

## KOI Network Setup

After completing setup, bring the network online:
```
koi-sh
network run
```

The next steps require the network to be running. (To shutdown the network, enter `Ctrl+C` in the shell, and then `quit` to exit).

## Telescope Setup in Slack
Now that the Telescope bot is running and in your Slack workspace, we can finish setting up within Slack. *(Note: all commands in this section can only be run by the user who installed the bot to the workspace).* 

1. Set up the observatory and broadcast channels.
    1. The observatory channel is where approved curators can decide to request tagged messages. Usually this should be a private channel with a small number of members. After setting it up, run the `/set-observatory-channel` command in the channel. 
    2. The broadcast channel is where approved messages are shared back to the community, this should be a public channel. After setting it up, run the `/set-observatory-channel` command in the channel.
2. Add Telescope to all channels you are interested in observing.
    1. You can automatically add it to all public channels by running `/add-to-public-channels`.
    2. If you want to selectively enable Telescope for certain channels (public and private), you can use the `/add-to-channel` command. It will join the channel you run the command in, or the one you specify e.g. `/add-to-channel #general`.
    3. You can remove Telescope in the same way by running `/remove-from-channel`.
3. Activate Telescope by running `/activate`, it will now start monitoring for Telescopes in real time going forward.
4. In order to capture past Telescopes, run `/backfill-messages`. This will search all channels the bot is in to find any messages that haven't already shown up in the observatory. If you add new channels in the future, you can rerun this command to backfill their data as well.

## Using Telescope
You've now finished configuring and setting up your Telescope bot! This section will explain how to use it.

### Tagging Messages
Any Slack member can tag interesting messages by reacting to them with the telescope emoji (or other configured emoji) in a channel with the Telescope bot in. After being tagged, the message will show up in the observatory channel.

### Requesting Messages
Within the observatory channel, approved curators can choose to either request tagged messages or ignore them. Requested messages will trigger the consent flow.

If this is a user's first requested message, they will receive the following options:
- Opt-in to data collection
- Anonymously opt-in to data collecton
- Opt-out of data collection

This selection will apply to all of their future requested messages. If a user opts in, they will receive a DM from the bot everytime a new message is approved, with the option to selectively anonymize or retract a specific message.

Messages in the observatory will show their current status, which can be one of:
- Tagged 🏷️
- Requested ⌛
- Ignored 👋
- Approved ✅
- Approved (anonymously) ✅
- Rejected ❌
- Retracted 🚫

Only messages which are approved, or approved anonymously will end up in the broadcast channel. Although anonymously opting-in will remove the authors name with a random pseudonym in the outgoing dataset, they will still be referenced by name within the broadcast channel in Slack, as it links back to the original message, and it is trivial to search for the original author within a workspace. 