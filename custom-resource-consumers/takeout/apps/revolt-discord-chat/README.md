## Additional steps to deploy Revolt

### Secrets

Create these secrets:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: revolt-${chat_name}-chat-env
  namespace: revolt-${chat_name}-chat
type: Opaque
stringData:
  MONGODB: mongodb://${chat_name}_chat:password@revolt-${chat_name}-chat-mongodb
  REDIS_URI: redis://revolt-${chat_name}-chat-redis/
  HOSTNAME: https://${chat_name}.chat
  REVOLT_APP_URL: https://${chat_name}.chat
  REVOLT_PUBLIC_URL: https://${chat_name}.chat/api
  VITE_API_URL: https://${chat_name}.chat/api
  REVOLT_EXTERNAL_WS_URL: ws://${chat_name}.chat/ws
  AUTUMN_PUBLIC_URL: http://${chat_name}.chat/autumn
  JANUARY_PUBLIC_URL: http://${chat_name}.chat/january
  REVOLT_UNSAFE_NO_CAPTCHA: "0"
  REVOLT_HCAPTCHA_KEY: 0x12345
  REVOLT_HCAPTCHA_SITEKEY: your-site-key-here
  REVOLT_UNSAFE_NO_EMAIL: "0"
  REVOLT_SMTP_HOST: email-smtp.us-east-1.amazonaws.com
  REVOLT_SMTP_USERNAME: SMTP_USERNAME_HERE
  REVOLT_SMTP_PASSWORD: SMTP_PASSWORD_HERE
  REVOLT_SMTP_FROM: Revolt <noreply@${chat_name}.chat>
  REVOLT_INVITE_ONLY: "1"
  REVOLT_MAX_GROUP_SIZE: "1000"
  REVOLT_VAPID_PRIVATE_KEY: See steps below
  AUTUMN_S3_REGION: minio
  AUTUMN_S3_ENDPOINT: http://revolt-chat-minio:9000
  MINIO_ROOT_USER: revolt${chat_name}minio
  MINIO_ROOT_PASSWORD: password
  AUTUMN_MONGO_URI: mongodb://${chat_name}_chat:password@revolt-${chat_name}-chat-mongo"
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: revolt-${chat_name}-chat-mongodb
  namespace: revolt-${chat_name}-chat
type: Opaque
stringData:
  MONGO_INITDB_ROOT_USERNAME: ${chat_name}_chat
  MONGO_INITDB_ROOT_PASSWORD: password
```

#### vapid

Run the steps here: https://gitlab.insrt.uk/revolt/delta/-/wikis/vapid
Copied below as retrieved on `2023-06-24`

---

VAPID keys are used to ensure that nobody else can communicate with your clients.

You can generate a private VAPID key by running:

    openssl ecparam -name prime256v1 -genkey -noout -out vapid_private.pem

This creates a PEM private key. In order to use this with the server, you must first base64 encode it, then you can pass it in using an environment variable. (Make sure to remove any newlines)

    base64 vapid_private.pem

To convert this to a public key, we run:

    openssl ec -in vapid_private.pem -outform DER|tail -c 65|base64|tr '/+' '_-'|tr -d '\n'

The output of this command is what the clients will be receiving.

---
