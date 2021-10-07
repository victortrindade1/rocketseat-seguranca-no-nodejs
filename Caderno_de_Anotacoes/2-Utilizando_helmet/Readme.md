# Utilizando Helmet

O helmet previne uma série de ataques hacker. Um muito comum q previne é o `XSS (Cross-site scripting)`, q consiste em injetar código JS em formulários de sites.

`yarn add helmet`

## src/app.js

```diff
import 'dotenv/config';

import express from 'express';
import * as Sentry from '@sentry/node';
import path from 'path';
import cors from 'cors';
+import helmet from 'helmet';
import Youch from 'youch';
import 'express-async-errors';

import sentryConfig from './config/sentry';
import routes from './routes';

import './database';

// O professor disse não gostar de usar "class" no frontend, mas no backend é
// muito bom de usar, usa bastante.
class App {
  constructor() {
    this.server = express();

    Sentry.init(sentryConfig);

    this.middlewares();
    this.routes();
    this.exceptionHandler();
  }

  middlewares() {
    this.server.use(Sentry.Handlers.requestHandler());
+    this.server.use(helmet());
    /**
     * Em produção, o cors fica assim:
     * this.server.use(cors({ origin: 'https://foobar.com.br' }));
     */
    this.server.use(cors());
    this.server.use(express.json());
    // Para o express aceitar acessar arquivos estáticos por url
    this.server.use(
      '/files',
      express.static(path.resolve(__dirname, '..', 'tmp', 'uploads'))
    );
  }

  routes() {
    this.server.use(routes);
    this.server.use(Sentry.Handlers.errorHandler());
  }

  exceptionHandler() {
    this.server.use(async (err, req, res, next) => {
      if (process.env.NODE_ENV === 'development') {
        const errors = await new Youch(err, req).toJSON();

        return res.status(500).json(errors);
      }

      return res.status(500).json({ error: 'Internal server error' });
    });
  }
}

// Exporto apenas o server da classe, e não a classe toda, pois traz segurança
export default new App().server;
```

Pra testar, vc pode ver no insomnia os headers da rota (tem uns 7). Qnd instala o helmet, os headers passam a ser uns 12 headers.
