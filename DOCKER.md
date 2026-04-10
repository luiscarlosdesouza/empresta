# Rodando o Empresta com Docker

Este guia fornece instruções para configurar e gerenciar o sistema **Empresta** utilizando Docker.

## Requisitos

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Instalação Rápida

1.  **Clone o repositório:**
    ```bash
    git clone https://github.com/luiscarlosdesouza/empresta.git
    cd empresta
    ```

2.  **Configure o arquivo .env:**
    Crie uma cópia do exemplo e preencha as credenciais necessárias (Senha Única, Replicado, SMTP):
    ```bash
    cp .env.example .env
    ```

3.  **Suba os containers:**
    ```bash
    docker-compose up -d --build
    ```

4.  **Finalize a instalação:**
    Execute estes comandos para preparar o banco de dados e as dependências:
    ```bash
    # Instalar dependências PHP
    docker exec -it empresta-web composer install

    # Instalar dependências de assets e compilar
    docker exec -it empresta-web npm install
    docker exec -it empresta-web npm run prod

    # Gerar chave e rodar migrações
    docker exec -it empresta-web php artisan key:generate
    docker exec -it empresta-web php artisan migrate
    docker exec -it empresta-web php artisan storage:link
    ```

O sistema estará disponível em [http://localhost:5003](http://localhost:5003).

## Comandos Úteis

### Gerenciamento de Containers
- **Parar os containers:** `docker-compose stop`
- **Iniciar os containers:** `docker-compose start`
- **Remover os containers:** `docker-compose down`
- **Logs em tempo real:** `docker-compose logs -f`

### Comandos Artisan (Laravel)
Qualquer comando `php artisan` deve ser executado através do container `empresta-web`:
```bash
docker exec -it empresta-web php artisan <comando>
```

### Banco de Dados Maker (Faker)
Para popular o banco com dados de teste:
```bash
docker exec -it empresta-web php artisan db:seed
```

## Deploy em Produção

Ao realizar o deploy em um servidor de produção, atente-se aos seguintes pontos:

1.  **APP_ENV**: Mude para `production` no `.env`.
2.  **APP_DEBUG**: Mude para `false` no `.env`.
3.  **Proxy Reverso**: Se utilizar HTTPS via proxy (Nginx/Apache), configure `FORCE_HTTPS=true` no `.env`. O sistema já está configurado para confiar em cabeçalhos de proxy.
4.  **Permissões**: O Dockerfile já configura as permissões de `storage` e `bootstrap/cache`, mas se notar problemas, rode:
    ```bash
    docker exec -it empresta-web chown -R www-data:www-data storage bootstrap/cache
    ```

## Solução de Problemas

### Erro de Build de Assets (Webpack)
Se o comando `npm run prod` falhar com erro de `ProgressPlugin`, certifique-se de que rodou o `npm install` primeiro. Existe um script de `postinstall` configurado para corrigir automaticamente esse erro.
