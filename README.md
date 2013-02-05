## Configuração CruiseControl.rb

### Configuração ambiente
#### Nginx
1. Configuração http server __/etc/nginx/nginx.conf__

    ```Shell
     http {
       include /etc/nginx/mime.types;
       default_type application/octet-stream;
       .
       .
       include /etc/nginx/enabled/*.conf;
     }
     ```
2. Configuração Upstream __[PROJECT_ROOT]/config/nginx.conf__
     ```Shell
     upstream webrick {
       server localhost:3333 fail_timeout=0;
     }

     server {
       listen 80 default;
       root ~/cruisecontrol.rb/public;
       try_files $uri/index.html $uri @webrick;

       location @webrick {
         proxy_pass http://localhost:3333;
       }
	
        error_page 500 502 503 504 /500.html
     }
     ```

3.  Criar link da configuração do cruisecontrol.rb no nginx
```Shell
# ln -s ~/cruisecontrol.rb/config/nginx.conf /etc/nginx/enabled
```

#### Configurar shell-script build
```Shell
$ touch redu_build
$ chmod +x redu_build
```

1. Build Redu:

    ```Shell
    #!/bin/bash
    echo "configure db";
    cp config/database.yml.example config/database.yml;
    bundle exec rake -q db:migrate;
    bundle exec rake -q db:test:prepare;
        
    echo "run tests"
    bundle exec rake spec;
    ```

2. Adicionar novo projeto ao cruisecontrol.rb

    ```shell
    $ cd [ROOT_CRUISE]
    $ ./cruise add redu -r [PATH_REPOSITORIO] -b [BRANCH]
    ```

3. Configurar banco de dados

#### Configurar autenticação
A autenticação utiliza é simplesmente o http_auth_basic.

```ruby
# config/application.rb

config.admin = {
  :username => 'redu-ci',
  :password => 'password'
}
```

```ruby
# app/controllers/application_controller.rv

before_filter :authenticate

protected
def authenticate
  authenticate_or_request_with_http_basic do |username, password|
    username == CruiseControl::Application.config.admin[:username] && password == CruiseControl::Application.config.admin[:password]
  end
end
```

#### Configurar notificações
##### `cruise_config.rb`
E-mail emissor de notificações configurável via variável (em config/cruise_config.rb) `project.email_notifier.from`. Endereços de e-mail de destinatários são adicionados ao array `project.email_notifier.emails` (no mesmo arquivo de configuração - config/cruise_config.rb).
```ruby
  project.email_notifier.from = 'ci@redu.com.br'
  project.email_notifier.emails = ['guilhermec@redu.com.br',
                                   'jessica@redu.com.br',
                                   'tacsio@redu.com.br',
                                   'juliana@redu.com.br',
                                   'tiago@redu.com.br',
                                   'matheus@redu.com.br']
```

#### Configurações adicionais

##### `configuration.rb`
Dentro da pasta do projeto do servidor de IC.
```ruby
@email_from = 'ci@redu.com.br'
@max_file_display_length = 5000.kilobytes
```

##### `cruise_config.rb`
Setar o intervalo para checar novas atualizações no repositório remoto para o tempo padrão (10s):
```ruby
project.scheduler.polling_interval = 10.seconds
```
Configurar para dar build apeanas quando forem realizadas modificações:
```ruby
project.scheduler.always_build = false
```

## Todo
### Configurar Git keys

