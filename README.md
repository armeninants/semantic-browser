# OhMyGraph Semantic Browser

> An RDF browser and SPARQL client.

This instruction will help you to use OhMyGraph Semantic Browser for both development and production.

## Demo

[OhMyGraph.com](http://ohmygraph.com)

![OhMyGraph](https://raw.githubusercontent.com/armeninants/ohmygraph/master/client/static/images/screencast1.gif)

## Prerequisites

Make sure you have [Node](https://nodejs.org/en/download/) and NPM installed.
Run `npm install -g forever` to install `forever` globally.

If you are going to run the application in development mode, install `webpack-dev-server` as well:

``` bash
npm install -g webpack-dev-server
```

## Installing

``` bash
git clone https://github.com/armeninants/ohmygraph.git
cd ohmygraph/
chmod gu+x ./*.sh
```

Copy `/.env.example` to `/.env` and edit it.
``` bash
cp .env.example .env
nano .env #use your preferred editor
```

Finally, pull from the repo and build the application:
``` bash
npm run update
```

## How to run
To choose between development and production modes, set the SB_ENV environment variable in the `/.env` file to a respective value.

Run `npm run start` to start.
The application is accessible now at [http://localhost:8031](http://localhost:8031).
You can change the port by setting the SB_PORT environment variable.

To stop, run `npm run stop`.

## Pulling updates


``` bash
npm run update
npm run start
```

`npm run update` is intended for production only, since it will reset all your local changes.

## License
OhMyGraph Semantic Browser is written by [Armen Inants](http://armen.inants.com).
The project is licensed under the Apache License 2.0 - see the [LICENSE](https://raw.githubusercontent.com/armeninants/ohmygraph/master/LICENSE) file for details.