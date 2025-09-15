# Symfony-7.2

* Symfony ile bir proje başlatmak için >> 
* **composer create-project symfony/skeleton my-backend "7.2.*"**
* Daha sonra endpoint yazabilmek için **composer require symfony/framework-bundle symfony/http-kernel symfony/http-foundation**
* Routing ( Endpointler için ) **composer require symfony/routing**
* Controller Desteği için **composer require symfony/orm-pack symfony/maker-bundle --dev**
* JSON response kolatylığı için **composer require symfony/serializer**
* Symfony server başlatmak için **symfony server:start** ancak symfony cli yoksa scoop üzerinden **scoop install symfony-cli** daha sonra **symfony server:start**. CLI kurmak istemiyorsan alternatif olarak **php -S localhost:8000 -t public** php'nin kendi serverini kullanabilirsin.
* CORS hatası alırsan **composer require nelmio/cors-bundle** yükle ve config -> packages altında **nelmio_cors.yaml** dosyasını oluştur. İçeriği ->
```
nelmio_cors:
  defaults:
    allow_origin: ['http://localhost:5173']
    allow_methods: ['GET', 'OPTIONS', 'POST', 'PUT', 'DELETE']
    allow_headers: ['Content-Type', 'Authorization']
    expose_headers: ['Link']
    max_age: 3600
  paths:
    '^/api/': ~
```
