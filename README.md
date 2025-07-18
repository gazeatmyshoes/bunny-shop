# Laurel Cafe (Telegram Mini App)

Laurel Cafe is an imaginary cafe that runs on Telegram as a Mini App. The project demonstrates basic concepts and approaches of building Telegram Mini App.

For a quick overview of repository structure see [Structure.md](Structure.md).

## Features

The functionality of the app includes the following features:
- View information about the cafe, available categories and popular dishes
- Display menu by category
- Detailed dish information
- Customization of the dish (variation, quantity) when adding to cart
- Viewing and editing items in the cart
- Order placement and payment using Telegram client
- Saving order between sessions, so users can continue next time the app open
- Basic user interaction via messages (order confirmation, launching the application using Inline Button)
- Admin panel to manage products at `/admin?key=<ADMIN_KEY>`
- Broadcast messages to all buyers using `/broadcast` command
- Orders stored in a SQLite database

![Laurel Cafe Mini App](/screenshots/laurel-cafe-mini-app.png)

## Before we start

The goal of the project is to provide a simple, but at the same time functional project that will be easy to understand, replicate, and develop your own Telegram Mini App.
The project deliberately avoids using different frameworks and libraries to make the project understandable for developers of different levels.

## Project overview

The project consists of two modules: **backend** and **frontend**. This means that the project includes all the necessary code to run the app by yourself. The code in each of modules includes documentation describing the purpose and/or principle of operation of a particular method. In this *README* we will focus on the main concepts, while you can deep dive to the project by reading the source files.

### Backend

Backend provides data displayed in the application, such as cafe information or a list of menu categories, and also acts as middleware for interacting with Telegram API (handling events, sending messages and more).

#### Technologies

The backend is written in **Python** (*3.11* was used for development and deploy). For the most part, the standard set of tools is used, but the project includes some third-party libraries, including:
- [**Flask**](https://pypi.org/project/Flask/) - Lightweight WSGI web application micro-framework. Used for creating API for both app data (e.g. menu data) and Telegram Bot webhook handling.
- [**Flask-CORS**](https://pypi.org/project/Flask-Cors/) - A Flask extension for handling Cross Origin Resource Sharing (CORS), making cross-origin AJAX possible. It's used as a development dependency.
- [**pyTelegramBotAPI**](https://pypi.org/project/pyTelegramBotAPI/) - Python implementation for the Telegram Bot API.
- [**python-dotenv**](https://pypi.org/project/python-dotenv/) - Reads key-value pairs from a .env file and can set them as environment variables. Allows to set environment variables for development without setting them in OS directly.

#### Structure

The backend folder includes two subfolders: **app** and **data**.

**app** folder contains the following Python files:
- `auth.py` - Mini App user validation.
- `bot.py` - Initialization and interaction with bot.
- `main.py` - Entry point. Includes Flask initialization and configuration, as well as supported API endpoints, including bot webhook.

**data** folder сontains JSON files with data that is returned by the corresponding API requests.

#### Environment variables

The application gets all secrets, as well as some variables such as URLs, from environment variables. For production environment variables are set directly in OS on hosting, for development and running the backend locally you will need to create an `.env` file. Below you will find a list of all supported/used variables.

`DEV_MODE` - flag indicating that the application is running in development mode. If this flag is set, TeleBot logs will be enabled and `DEV_APP_URL` (if present) will be added to the CORS exception list.
`DEV_APP_URL` - usually the local address where you run the frontend application. If specified, will be added to the CORS exception list in case when `DEV_MODE` variable is presented.
`APP_URL` - production URL of your Mini App (the address where the backend is hosted). This URL will be added to the CORS origin list to allow host frontend and backend on different servers. It is also used to launch the application using [Inline Button](https://core.telegram.org/bots/webapps#inline-button-mini-apps). Note: this address is the real address of your app (the host on which it is deployed). This is the address you gave BotFather when you created the app. Not to be confused with the app address generated by BotFather (like *https://t.me/mybot/myapp*).
`DOMAIN` - публичный домен, на котором развернут бот и его веб-приложение. Используется для настройки вебхука.
`WEBHOOK_URL` - полный URL обработчика событий (если не указан, формируется из `DOMAIN`). Вебхук переустанавливается при каждом старте приложения.
`BOT_TOKEN` - bot token issued by BotFather when creating the bot.
`PAYMENT_PROVIDER_TOKEN` - payment provider token issued when connecting payments.
`ADMIN_CHAT_ID` - Telegram ID администратора, которому будут отправляться уведомления о заказах.
`ADMIN_KEY` - секретный ключ для доступа к админ-панели и подтверждения команды `/broadcast`.

### Administration

Откройте `/admin?key=<ADMIN_KEY>` в браузере, чтобы добавить или отредактировать товары.
Команда `/broadcast` в чате с ботом позволяет отправить сообщение всем пользователям,
которые оформляли заказы.

#### Running locally

To test the changes you have made to the backend code and/or the data, or just simply play with it you can deploy the backend locally.

If this is your first time after you cloned the project, you need to start with initialization of [**venv**](https://code.visualstudio.com/docs/python/environments) (Python Virtual Environment):

```shell
cd backend
python -m venv .venv
source .venv/bin/activate # For Linux/MacOS
.venv/Scripts/Activate.ps1 # For Windows
```

When you activated the environment, you need to install all the dependencies:

```shell
pip install -r requirements.txt
```

Now you are ready to start the development server:

```shell
flask --app app.main:app run
```

You will see in your console something like that:

```shell
 * Serving Flask app 'app.main:app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```

Here you can see the address where you can reach your API. *Memorize it, you will need it when working with the frontend part.*

#### Production deploy

Since the project uses Flask, you will need a WSGI server to deploy to production. You can find some examples below.

##### Phusion Passenger

Current production version of Laurel Cafe Mini App works on [**Phusion Passenger**](https://www.phusionpassenger.com/) in combination with **cPanel**. To run the project in this combination, in the project root (**backend** folder) you need to create a file `passenger_wsgi.py` with the following contents:

```python
from app.main import app as application
```

Then create a Python application in cPanel and specify the following params:
- *Application startup file* - `passenger_wsgi.py`
- *Application Entry point* - `application`

##### Vercel

[**Vercel**](https://vercel.com/) provides possibility to run Python project using Serverless Functions. To do so create in the project root (**backend** folder) the `vercel.json` file:

```json
{
    "builds": [
        {
            "src": "/app/main.py",
            "use": "@vercel/python"
        }
    ],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "/app/main.py"
        }
    ]
}
```

Then follow the Vercel's [instructions](https://vercel.com/docs/functions/serverless-functions/runtimes/python#web-server-gateway-interface) to deploy the app.

### Frontend

The frontend is designed and works as a SPA (Single Page Application). It uses the usual approaches used in web application development, but takes into account the specifics of the target platform. Thus, first of all, it is adapted for mobile devices, and also part of the key functionality is tied to the features provided by Telegram (for example, the main action button that allows you to add a product to cart or proceed to payment).

#### Technologies

The application is a static site with dynamic content. It uses a standard set of **HTML** + **CSS** + **JS**. In addition, the following set of JS libraries is used:
- [**telegram-web-app**](https://telegram.org/js/telegram-web-app.js) - Connector between web app and Telegram client.
- [**jQuery**](https://jquery.com/) - Small, fast JS library, that makes different HTML manipulations easier.
- [**Transit**](https://rstacruz.github.io/jquery.transit/) Smooth CSS transitions & transformations for jQuery.
- [**lottie-web**](https://github.com/airbnb/lottie-web) - AE animations rendering natively on Web.

There are also some external CSS resources used in the app:
- [**Rubik Font**](https://fonts.google.com/specimen/Rubik?query=rubik) - The font used across the app.
- [**Material Symbols**](https://fonts.google.com/icons) - The latest free icon set from Google.

#### Structure

The Frontend folder includes the following subfolders:
- `css` - Contans single `index.css` file with all the styles using in the app. All styles are divided into sections, each of which relates to relevant content
- `icons` - Icons used in the app.
- `js` - JS files describing the app logic. All files are distributed in folders corresponding to their purpose (e.g., routing or pages). The files also contain documentation to help you understand the code. On top level of the folder you may find `index.js` - app entry point.
- `lottie` - Lottie animation files in JSON format used in the app.
- `pages` - Folder with HTML *pseudo-pages*. Since the application is an SPA, it has only one real page described in `index.html`. By *pseudo-pages* we mean a piece (fragment) of content displayed at a given time inside a real page, such as *home* or *cart* content.

#### Running locally

To run the frontend app locally, you'll need a server that knows how to host static websites.

##### VS Code Live Server

One of the easiest ways to run an application locally is the [**VS Code Live Server**](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension. Simply install the extension to VS Code and click the **Go Live** button that will appear in the bottom right corner once the extension is installed.

##### Netlify

[**Netlify**](https://www.netlify.com/) is a powerful platform that, among other things, allows you to host a static website literally in two clicks. To launch a site locally, you need to install the [**Netlify CLI**](https://www.netlify.com/products/cli/) and run the command:

```shell
netlify dev
```

When running app locally, you will get the address where it's hosted. You need to set this address to `DEV_APP_URL` variable along with setting `DEV_MODE` variable in backend's `.env` file to resolve CORS issues. See backend's [Running locally](#environment-variables) section for more details. If you run the backend locally, you also need to set local server URL (you got it after running the server) update `baseUrl` variable in `js/requests/requests.js` file:
```js
const baseUrl = 'http://<server-host-name>:<server-port>';
```

#### Production deploy

To host the app on production, just upload content from the `frontend` folder to your hosting. Don't forget to set the `baseUrl` of your production server as well as setting production `APP_URL` environment variable on backend's server machine, when frontend is deployed.

## Running with Docker

The repository contains a `Dockerfile` and `docker-compose.yml` to simplify deployment. After setting up the `.env` file you can build and start all services with:

```bash
docker-compose build
docker-compose up -d
```

The compose file launches the Flask application behind Nginx with HTTPS powered by certbot. The SQLite database file is mounted so data persists between restarts.

Remember to set your domain in the `.env` file and configure it for the bot via `/setdomain` in BotFather. Webhook will be refreshed automatically when the container starts.
