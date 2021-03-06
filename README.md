# JukeBot
[![Build Status](https://travis-ci.org/TheConnMan/jukebot.svg?branch=master)](https://travis-ci.org/TheConnMan/jukebot)
[![Docker Pulls](https://img.shields.io/docker/pulls/theconnman/jukebot.svg)](https://hub.docker.com/r/theconnman/jukebot/)

Slack-Enabled Syncronized Music Listening

**JukeBot** is for Slack teams who want to listen to music together, add music through Slack, and chat about the music in a Slack channel.

## Local Development
To get started with local development follow these steps:

1. Clone this repo and `cd` into it
1. Install Sails with `npm install -g sails`
1. Install **JukeBot** dependencies with `npm install`
1. Get a Google API key using the setup steps below
1. Run **JukeBot** with `GOOGLE_API_KEY=<your-API-key> sails lift` and go to http://localhost:1337

## Setup
### Google API Key for YouTube
1. Go to https://console.developers.google.com
1. Create a new Google Project named JukeBot
1. Under **Create credentials** choose **API key** and use the API key for the environment variable below
1. Click **Library** in the left panel
1. Go to **YouTube Data API**
1. Click **Enable** at the top

### Slash Command (Optional)
To use a Slack Slash Command you'll need to set one up (preferably after the running the deployment steps below) and follow these instructions:

1. Go to the [Slack Slash Command setup page](https://my.slack.com/apps/A0F82E8CA-slash-commands), add a configuration, and name it **JukeBot**
1. Input the URL you configure during the deployment step and add a trailing `/slack/slash` (e.g. jukebox.my.domain.com/slack/slash)
1. Change the request method to GET
1. Copy the **Token** and use it as the **SLASH_TOKEN** environment variable
1. Customize the name to **JukeBot**
1. Check the box to "Show this command in the autocomplete list"
1. Add the description "Slack-Enabled Syncronized Music Listening"
1. Add the usage hint "add <youtube-url> - Add a YouTube video to the playlist"
1. Click save!

## Deployment
**WARNING:** Data persistence is not currently a priority. Videos are though to be transient and once they are more than an hour old they aren't shown in the UI. Due to this schema migrations are not a high priority and new runs of **JukeBox** will start with a fresh database.

### Deployment without HTTPS
**JukeBot** can easily be run with Docker using the following command:

```bash
docker run -d -p 80:1337 -e GOOGLE_API_KEY=<your-API-key> theconnman/jukebox:latest
```

Make sure to add any additional environment variables as well to the above command. Then go to the URL of your server and listen to some music!

### Deployment with HTTPS (Required when using a Slash Command)
Within this project is a `docker-compose.yml` file for use when deploying this project to your own server. Slack Slash Commands require the slash command endpoint to be HTTPS, so JukeBot uses [Let's Encrypt](https://letsencrypt.org/) to get an SSL cert. You'll need to be hosting JukeBot on a domain you have DNS authority over for this to work as you'll need to create a couple DNS entries.

The first thing to do after cloning this repo (or copying down) onto the deployment box is to replace a few values in `docker-compose.yml` and `traefik.toml`. In `docker-compose.yml` replace the two **localhost** references with your own domain or subdomain which points to the current box (e.g. projects.theconnman.com). In `traefik.toml` replace the email address with your own and the domain from localhost to the same one as before. Also run `touch acme.json` to create an empty credentials file.

You'll need to make sure you have DNS entries for the domain or subdomain above pointing to the current box as well as the wildcard subdomains of the given domain (e.g. \*.projects.theconnman.com). This is not only so Slack can locate your deployment, but also so Let's Encrypt can negotiate for your SSL cert.

Lastly, there is an `example.env` file provided which you should copy to `.env` and replace the contents of. Inside are example environment variables (described below) which you'll need to run JukeBot.

After that run `docker-compose up -d` and you should be able to access the UI at jukebox.my.domain.com (after replacing with your domain or subdomain of course).

## Environment Variables
- **GOOGLE_API_KEY** - Google project API key
- **SLACK_WEBHOOK** (Optional) - [Slack Incoming Webhook URL](https://my.slack.com/apps/A0F7XDUAZ-incoming-webhooks) for sending song addition and currently now playing notifications
- **SLASH_TOKEN** (Optional) - [Slack Slash Token](https://my.slack.com/apps/A0F82E8CA-slash-commands) for verifying a Slash Command request, only required if a Slash Command is set up
- **SERVER_URL** (Optional) - JukeBot server URL for linking back, only needed if **SLACK_WEBHOOK** is provided
