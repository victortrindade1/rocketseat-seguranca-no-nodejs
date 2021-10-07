# Utilizando Express Brute

O Express Brute é uma lib que impede o usuário de fazer request a partir de algumas tentativas. É mt bom pra rotas de login. Se um hacker coloca um programa q fica tentando achar senha, este programa não consegue mais. O q faz o express-brute é aumentar o tempo de espera da request a cada vez q a request é reexecutada. Se, por exemplo, eu faço mais de 5 tentativas de login muito rápido, então na sexta, já bloqueia por alguns segundos. Se eu tento mais request alí, o tempo de bloqueio aumenta gradativo.

Instale 2 libs:
`yarn add express-brute express-brute-redis`

## src/routes.js

```diff
import { Router } from 'express';
+import Brute from 'express-brute';
+import BruteRedis from 'express-brute-redis';
import multer from 'multer';
import multerConfig from './config/multer';

import UserController from './app/controllers/UserController';
import SessionController from './app/controllers/SessionController';
import FileController from './app/controllers/FileController';
import ProviderController from './app/controllers/ProviderController';
import AvailableController from './app/controllers/AvailableController';
import AppointmentController from './app/controllers/AppointmentController';
import ScheduleController from './app/controllers/ScheduleController';
import NotificationController from './app/controllers/NotificationController';

import validateUserStore from './app/validators/UserStore';
import validateUserUpdate from './app/validators/UserUpdate';
import validateSessionStore from './app/validators/SessionStore';
import validateAppointmentStore from './app/validators/AppointmentStore';

import authMiddleware from './app/middlewares/auth';

const routes = new Router();
const upload = multer(multerConfig);

+const bruteStore = new BruteRedis({
+  host: process.env.REDIS_HOST,
+  port: process.env.REDIS_PORT,
+});
+
+const bruteForce = new Brute(bruteStore);

routes.post('/users', validateUserStore, UserController.store);
-routes.post('/sessions', validateSessionStore, SessionController.store);
+routes.post(
+  '/sessions',
+  bruteForce.prevent,
+  validateSessionStore,
+  SessionController.store
+);

routes.get('/teste', (req, res) => res.send('ok'));

// Fiz um middleware global para todas as rotas abaixo
// As rotas abaixo são para usuário logado
routes.use(authMiddleware);

// Posso definir authMiddleware de forma local:
// routes.put('/users', authMiddleware, UserController.update);
// Não precisa passar o id pois User possui hash, q lê o id lá no middleware
// auth.js, logo o update fica mais prático neste caso
routes.put('/users', validateUserUpdate, UserController.update);

routes.get('/providers', ProviderController.index);
routes.get('/providers/:providerId/available', AvailableController.index);

routes.post('/appointments', validateAppointmentStore, AppointmentController.store);
routes.get('/appointments', AppointmentController.index);
routes.delete('/appointments/:id', AppointmentController.delete);

routes.get('/schedule', ScheduleController.index);

routes.get('/notifications', NotificationController.index);
routes.put('/notifications/:id', NotificationController.update);

routes.post('/files', upload.single('file'), FileController.store);

export default routes;
```
