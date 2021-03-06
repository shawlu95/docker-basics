# Laravel-PHP

Laravel-PHP stack is non-trivial to install on local machine. Use a multi-container setup

Three applications containers, which stay UP as long as application is running.

1. a container with php interpretor to read from local host source code (port 9000)
2. a container with nginx web server
3. a mysql container

Utility containers:

1. _Composer_: used to install dependencies. just like npm to node
2. _Laravel Artisan_: required by composer
3. _npm_: laravel uses it for some front-end logic

---

## Getting Started

Generate projects

```bash
docker-compose run --rm composer create-project --prefer-dist laravel/laravel .
```

Edit [.env](./src) of generated project:

```
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

Bring up containers (except composer). Should be able to visit port [8000](http://localhost:8000/)

```bash
docker-compose up -d server php mysql
```

Add `depends_on` to the server container. Then can simplify start up command.

```bash
docker-compose up -d server

# force rebuild image if something changed
docker-compose up -d --build server
```

Bring up artisan (would fail if no database conenction)

```bash
docker-compose run --rm artisan migrate
```

---

## Bind Mount

- Do not use bind mount when deploy, this is just for development (a server pod can lives on a different machine than php pod)
- application containers cannot have bind mount. Instead, copy `src` when building image (although update won't be instantly reflected)
- utility containers can have bind mount because they only need to run locally
