# Luma Calendar Discord Bot

## Description

The Luma Calendar Discord Bot integrates Luma Calendar events with Discord servers, providing daily notifications about upcoming events directly in Discord channels. This tool helps communities stay informed and organized.

## Architecture

Built with [Express.js](https://expressjs.com/) leveraging the [Discord.js package](https://www.npmjs.com/package/discord.js), the bot follows a modular architecture for maintainability and scalability. It uses [PostgreSQL](postgresql.org/) for data persistence.

Note: The Discord API refers to Discord servers as "guilds". This nomenclature is used throughout the bot for consistency and easier readability.

## Setup and Running

1. Clone the repository:
   ```
   git clone https://github.com/<your-username>/luma-calendar-discord-bot.git
   ```

2. Install dependencies:
   ```
   npm install
   ```

3. Set up your `.env` file in the root directory. An example file is provided (`.env.example`):

   ```
   DISCORD_TOKEN=your_discord_bot_token
   DATABASE_URL=your_postgres_database_url
   NODE_ENV=development // or production
   ```

  Where to find these:

  1.  `DISCORD_TOKEN`: This is your Discord bot token. To get this:
     1. Go to the [Discord Developer Portal](https://discord.com/developers/applications)
     2. Create a new application
     3. Go to the "Bot" tab and click "Add Bot" if you haven't already
     4. Under the bot's username, click "Copy" to copy the token
  2. `DATABASE_URL`: This is the URL to your PostgreSQL database. It should be in the format:
     ```
     postgres://username:password@host:port/database
     ```
     For development, you can use a local database URL with your own username, password, and database name. Remember, you must initialize posgres locally in order to have access to the port in your application.
     ```
     postgres://username:password@localhost:5432/bot_data
     ```
  3. `NODE_ENV`: This is the environment you want to run the application in. For development, this should be `development`. For production, this should be `production`.


4. Start the bot:
   ```
   npm start
   ```

5. Install the bot in your server:

   1. Once you have the bot server running, you can install the bot to your server by heading over to your [Discord Developer Portal](https://discord.com/developers/applications).
   2. Then, head over to the "OAuth2" tab and add the local endpoint to the "Redirects" section. For development, this should be something like `http://localhost:3000/oauth2-callback`. For production, this should be the deployed endpoint.
   3. Once you've added the redirect, then you want to make sure you have all permissions defined in the OAuth2 URL Generator: `identify`, `guilds`, `applications.commands`, `bot`. We're keeping them strategically minimal, but if you want to add more permissions for future functionalities, this is where you'll need to do so.
   4. Then, select the redirect URL you want to work with. This is the URL that Discord will use to redirect the user after they have installed the bot, so it is the endpoint that kickstarts the event listeners within your application.
   5. In "Bot Permissions", make sure you have the following permissions selected:
      - `View Channels`
      - `View Audit Log`
      - `Send Messages`
      - `Manage Messages`
      - `Read Message History`
      - `Use External Emojis`
      - `Add Reactions`
  6. After defining the permissions, make sure you've selected "Guild install" as your integration type and then copy and paste the Generated URL into your browser to install the bot in your server.

## Database

This bot uses PostgreSQL for data persistence. The database stores guild settings and setup status for each Discord server where the bot is installed.

### Schema

The main table is `guild_settings` with the following structure:

- `guild_id` (TEXT): The primary key, representing the unique ID of each Discord guild.
- `settings` (JSONB): A JSON object storing various settings for the guild, including:
  - `calendarUrl`: The Luma calendar URL for the guild.
  - `notificationsChannelId`: The ID of the channel where notifications will be sent.
  - `notificationTime`: The time when daily notifications should be sent.
  - `timezone`: The timezone for the guild.
- `setup_started` (BOOLEAN): Indicates whether the setup process has been initiated for the guild.

### Functions

The `db.js` file provides several functions for interacting with the database:

- `getGuildSettings`: Retrieves settings for a specific guild.
- `setGuildSetting`: Updates a specific setting for a guild.
- `areSettingsComplete`: Checks if all required settings are present for a guild.
- `setSetupStarted`: Updates the setup status for a guild.
- `hasSetupStarted`: Checks if setup has been initiated for a guild.

These functions use parameterized queries to prevent SQL injection and handle potential errors.

## Discord Commands

The bot primarily operates through an interactive setup process when added to a new guild. It doesn't use traditional text commands, but instead guides users through setup via DMs and interactive components.

## Development

Important to note:
- Make sure you're running the server locally when you install it in your server.
- Make sure you add the development endpoint to the "Redirects" section in the Discord Developer Portal.
- To test, make sure you have the local posgreql server running.

### Database Setup

This bot uses PostgreSQL for data persistence.  The database stores guild settings and setup status for each Discord server where the bot is installed.

Follow these steps to set up the database locally:

1. Install PostgreSQL:
   - For macOS: `brew install postgresql`
   - For Ubuntu: `sudo apt-get install postgresql`
   - For Windows: Download and install from the [official PostgreSQL website](https://www.postgresql.org/download/windows/)

2. Start the PostgreSQL service:
   - For macOS: `brew services start postgresql`
   - For Ubuntu: `sudo service postgresql start`
   - For Windows: It should start automatically after installation

3. Create a new database:
   ```
   createdb bot_data
   ```

4. Connect to the database:
   ```
   psql -d bot_data
   ```

5. Update your `.env` file with the local database URL:
   ```
   DATABASE_URL=postgresql://username:password@localhost:5432/bot_data
   ```
   Replace `username` and `password` with your PostgreSQL credentials.

The bot will automatically connect to this database when you start it locally.

-- Optional: The application creates the necessary table upon calling on `npm start`. However, if you'd like to create the table manually, you can do so with the following command once inside the `psql` connection console:

  ```sql
   CREATE TABLE IF NOT EXISTS guild_settings (
     guild_id TEXT PRIMARY KEY,
     settings JSONB,
     setup_started BOOLEAN DEFAULT FALSE
   );
  ```

## Production Deployment

For production deployment, this project is set up to use Heroku, although of course you can select whichever provide you rather work with. I chose Heroku because their free tier is enough, when combined with [UptimeRobot](https://uptimerobot.com/) which pings the server every 5 minutes to ensure it remains active.

Follow these steps to deploy your bot and set up the PostgreSQL database in Heroku:

1. Create a Heroku account if you haven't already, and [install the Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

2. Log in to Heroku via the CLI:
   ```
   heroku login
   ```

3. Create a new Heroku app:
   ```
   heroku create your-app-name
   ```

4. Add the PostgreSQL addon to your Heroku app:
   ```
   heroku addons:create heroku-postgresql:essential-0 --app your-app-name
   ```

   - `heroku addons:create` is the command to add a new addon to your Heroku app.
   - `heroku-postgresql` is the name of the addon, which is Heroku's managed PostgreSQL database service.
   - `essential-0` is the plan tier. It's their cheapest tier of Heroku PostgreSQL, suitable for development and small projects. It has some limitations, including: 10,000 row limit, 1 GB of storage, no automatic backups, limited concurrent connections.
   - `--app your-app-name` specifies which Heroku app to add this database to. Replace `your-app-name` with the actual name of your Heroku app.

5. Set up the necessary environment variables:
   ```
   heroku config:set DISCORD_TOKEN=your_discord_bot_token
   heroku config:set NODE_ENV=production
   ```
   -- In production, Heroku sets the `DATABASE_URL` environment variable for you after you have the Postgress add-on.

6. Push your code to Heroku:
   ```
   git push heroku main
   ```

7. Scale up your dyno so the server is running:
   ```
   heroku ps:scale web=1
   ```

8. To view your app's logs:
   ```
   heroku logs --tail
   ```

9.  To connect to the PostgreSQL database directly, you can call:
    ```
    heroku pg:psql
    ```

    Alternatively, you should see a section called Add-ons in your Heroku dashboard.
    1. Click on the "Heroku Postgres" add-on you should have already installed through step 4.
    2. Go into the "Settings" tab
    3. Go to "Database Credentials"
    4. Copy the "Heroku CLI" value
    5. Paste it your terminal and hit enter
    6. Perform any SQL queries to your production database.

## Folder Structure

- `server.js`: Main server file containing the Express.js application paths.
- `src/`: Main source code directory
  - `index.js`: Main entry point of the application. Handles bot initialization and Discord event listeners.
  - `services/`: Contains logic interacting with external services.
    - `cronJob.js`: Manages the scheduling and execution of periodic tasks, such as fetching and posting event updates.
    - `db.js`: Handles database operations for storing and retrieving guild settings.
    - `lumaScraper.js`: Manages the scraping of Luma Calendar events. This can be improved by using the Luma API for calendars with a paying subscription.
  - `utils/`: Contains utility functions and helpers.
    - `showEvents.js`: Manages the display of events in the Discord channel.
    - `time.js`: Contains functions for handling time-related operations.
  - `discord-interactions/`: Handles interactions with Discord's API.
    - `guildSetup.js`: Manages the setup process for new guilds, including user interactions for configuration.
    - `requestInformation.js`: Contains functions for requesting and handling user input during the setup process.
- `.env`: Stores environment variables.
- `package.json`: Defines the project dependencies and scripts.
- `README.md`: Project documentation.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE]([LICENSE](https://www.gnu.org/licenses/gpl-3.0.en.html)) file for details.

This license allows others to use, modify, and distribute this code, but requires that any modifications or derivative works also be released under the same GPL license. This ensures that improvements and modifications to the code remain open source.
