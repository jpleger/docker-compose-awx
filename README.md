# docker-compose-awx
Docker compose file for AWX

The [awx installer](https://github.com/ansible/awx/blob/devel/installer/roles/local_docker/templates/docker-compose.yml.j2) has its own docker-compose thats run through ansible. However if you don't want to use the offical method of bootstrapping, here is a compose file.

# How to use:

## Write the /docker/awx/credentials.py file
```
DATABASES = {
    'default': {
        'ATOMIC_REQUESTS': True,
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': "awx",
        'USER': "awx",
        'PASSWORD': "Ch4ng3m3!",
        'HOST': "postgres",
        'PORT': "5432",
    }
}
BROKER_URL = 'amqp://{}:{}@{}:{}/{}'.format(
    "guest",
    "guest",
    "rabbitmq",
    "5672",
    "awx")

CHANNEL_LAYERS = {
    'default': {'BACKEND': 'asgi_amqp.AMQPChannelLayer',
                'ROUTING': 'awx.main.routing.channel_routing',
                'CONFIG': {'url': BROKER_URL}}
}
```

## Write the secret key
 > echo "SUPERs3cr3tPassw0rd" > /docker/awx/SECRET_KEY


## Setup your /docker/awx/environment.sh file:
```
DATABASE_USER=awx
DATABASE_NAME=awx
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_PASSWORD="Ch4ng3m3!"
MEMCACHED_HOST=memcached
RABBITMQ_HOST=rabbitmq
AWX_ADMIN_USER=guest
AWX_ADMIN_PASSWORD=guest
```
## Create the CA items:
  > docker exec awx_web '/usr/bin/update-ca-trust'

# Other stuff

When the ansible awx starts initially, you might see an exception like noted below, it is normal and the awx_task container will create the schema and then the awx_web will connect afterwards.

```
awx_web      |   File "/var/lib/awx/venv/awx/lib64/python3.6/site-packages/django/utils/six.py", line 685, in reraise
awx_web      |     raise value.with_traceback(tb)
awx_web      |   File "/var/lib/awx/venv/awx/lib64/python3.6/site-packages/django/db/backends/utils.py", line 64, in execute
awx_web      |     return self.cursor.execute(sql, params)
awx_web      | ProgrammingError: relation "conf_setting" does not exist
awx_web      | LINE 1: ...f_setting"."value", "conf_setting"."user_id" FROM "conf_sett...
awx_web      |                                                              ^
awx_web      | 
awx_web      | WSGI app 0 (mountpoint='') ready in 3 seconds on interpreter 0x2436290 pid: 127 (default app)
awx_web      | No previous hash foundRESULT 2
awx_web      | OKREADY
awx_task     |   Applying main.0001_initial... OK

```