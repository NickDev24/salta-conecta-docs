# Matriz de entornos y variables

| Variable                                       | Local (dev)                            | Stage/Preview                                            | Prod                            |
| ---------------------------------------------- | -------------------------------------- | -------------------------------------------------------- | ------------------------------- |
| NEXT_PUBLIC_APP_URL                            | http://localhost:3000                  | https://stage.saltaconecta.online (o URL Preview Vercel) | https://saltaconecta.online     |
| NEXT_PUBLIC_BASE_URL                           | http://localhost:3000                  | https://stage.saltaconecta.online                        | https://saltaconecta.online     |
| NEXT_PUBLIC_API_URL                            | http://localhost:3000/api              | https://stage.saltaconecta.online/api                    | https://saltaconecta.online/api |
| NEXTAUTH_URL                                   | http://localhost:3000                  | https://stage.saltaconecta.online                        | https://saltaconecta.online     |
| NEXTAUTH_SECRET                                | local secret dev                       | secret en Secret Manager                                 | secret en Secret Manager        |
| MONGODB_URI                                    | mongodb://localhost:27017/saltaconecta | cluster stage                                            | cluster prod                    |
| REDIS_URL                                      | redis://localhost:6379                 | redis stage                                              | redis prod                      |
| JWT_SECRET                                     | dev secret                             | secret stage                                             | secret prod                     |
| ENCRYPTION_KEY                                 | dev key                                | key stage                                                | key prod                        |
| NEXT_PUBLIC_GOOGLE_MAPS_API_KEY                | key dev restringida por dominio        | key stage restringida                                    | key prod restringida            |
| NEXT_PUBLIC_MERCADOPAGO_PUBLIC_KEY             | test key                               | test/prod según entorno                                  | prod key                        |
| MERCADOPAGO_ACCESS_TOKEN/MP_ACCESS_TOKEN       | test token                             | test/prod según entorno                                  | prod token                      |
| NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET            | dev secret                             | stage secret                                             | prod secret                     |
| RESEND_API_KEY                                 | dev key                                | stage key                                                | prod key                        |
| RESEND_FROM                                    | no-reply@dev.local                     | no-reply@stage.saltaconecta.online                       | no-reply@saltaconecta.online    |
| CLOUDINARY_CLOUD_NAME                          | dev cloud                              | stage cloud                                              | prod cloud                      |
| CLOUDINARY_API_KEY/SECRET                      | dev                                    | stage                                                    | prod                            |
| NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME              | dev cloud                              | stage cloud                                              | prod cloud                      |
| UPSTASH_REDIS_REST_URL/TOKEN                   | opcional                               | stage                                                    | prod                            |
| VAPID_PRIVATE_KEY/NEXT_PUBLIC_VAPID_PUBLIC_KEY | dev                                    | stage                                                    | prod                            |

Notas:

- Secrets deben residir en gestor de secretos (Vercel/CI/infra). No versionarlos.
- CORS: orígenes exactos por entorno, sin `*` en prod.
- Healthchecks: `/health` y `/ready` habilitados en stage/prod.
