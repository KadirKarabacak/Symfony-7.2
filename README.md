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
## Yeni Endpoint Oluşturma 🟢
* php bin/console make:controller ÜrünController
* Örnek bir CRUD operation'a sahip controller ->
```
<?php

namespace App\Controller;

use App\Entity\Product;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Attribute\Route;

class ProductController extends AbstractController
{
    # PRODUCT LIST
    #[Route('/api/products', name: 'app_products', methods: ['GET'])]
    public function  index(EntityManagerInterface $em): JsonResponse
    {
        $products = $em->getRepository(Product::class)->findAll();

        $data =array_map(fn(Product $p)=> [
            'id'=> $p->getId(),
            'name'=> $p->getName(),
            'price'=> $p->getPrice()
        ], $products);

        return new JsonResponse($data);
    }

    # PRODUCT SHOW
    #[Route('/api/products/{id}', name: 'app_product_show', methods: ['GET'])]
        public function show(EntityManagerInterface $em, int $id):JsonResponse
    {
        $product = $em->getRepository(Product::class)->find($id);

        if(!$product){
            return new JsonResponse(['error'=> 'Ürün bulunamadı'], 404);
        }

        return new JsonResponse([
            'id' => $product->getId(),
            'name' => $product->getName(),
            'price' => $product->getPrice(),
        ]);
    }

    # CREATE PRODUCT
    #[Route('/api/products', name:'app_product_create', methods: ['POST'])]
    public function create(Request $request,EntityManagerInterface $em):JsonResponse
        {
            $data =json_decode($request->getContent(), true);
            $product = new Product();
            $product->setName($data['name']);
            $product->setPrice($data['price'] );
            $em->persist($product);
            $em->flush();

            if(!$data || !isset($data['name'], $data['price'])){
                return  new JsonResponse([
                    'error'=> 'Eksik parametre. "name" ve "price" gerekli.'
                ], 400);
            }

            return new JsonResponse([
                'id' => $product->getId(),
                'name' => $product->getName(),
                'price' => $product->getPrice(),
            ], 201);

        }

        # DELETE PRODUCT
    #[Route('/api/products/{id}', name:'app_product_delete', methods: ['DELETE'])]
    public function delete(EntityManagerInterface $em, $id):JsonResponse
    {
       $product =$em->getRepository(Product::class)->find($id);

            if (!$product) {
                return new JsonResponse(['error' => 'Ürün bulunamadı'], 404);
            }

            $em->remove($product);
            $em->flush();

        return new JsonResponse(['message' => 'Ürün silindi'], 200);

    }
}


```
* Şimdi burada ürün oluşturdukça bir data-base'e kaydetmek için config -> doctrine.yaml dosyasına gidiyoruz. Url kısmına bilgilerimizi giriyoruz. Şimdilik proje içerisine kaydedilen sqlite kullanabiliriz. ( var -> altına data.db olarak açıyor. ) url'imiz **url: 'sqlite:///%kernel.project_dir%/var/data.db'**
* Daha sonra bir entity oluşturuyoruz. Bizim konumumuzda **php bin/console make:entity Product**
* Sonrasında alanlarımızı, tiplerini ve boş olup olamayacaklarını belirtiyoruz.
* Alanlar eklendikten sonra migration yapmamız gerek ( **php bin/console make:migration ve devamında php bin/console doctrine:migrations:migrate** )
### VALIDATION
* Validasyon için **composer require symfony/validator** yüklüyoruz. Sonrasında ( başka şekilde de yapılabilir ) controller içinde aşağıdaki örnekteki gibi **constraints** oluşturup endpoint işlemini gerçekleştirmeden önce gelen veriyi validate edebiliriz.
```
# CREATE PRODUCT
    #[Route('/api/products', name:'app_product_create', methods: ['POST'])]
    public function create(Request $request,EntityManagerInterface $em, ValidatorInterface $validator):JsonResponse
        {
            $data =json_decode($request->getContent(), true);

            $constraints = new Assert\Collection([
                'name'  => [
                    new Assert\NotBlank(message: 'Name is required'),
                    new Assert\Length(max: 255, maxMessage: 'Name cannot exceed 255 chars')
                ],
                'price' => [
                    new Assert\NotBlank(message: 'Price is required'),
                    new Assert\Type(type: 'numeric', message: 'Price must be numeric'),
                    new Assert\Positive(message: 'Price must be positive')
                ],
            ]);

            // 2) Ham data’yı validate et
            $errors = $validator->validate($data, $constraints);

            if (count($errors) > 0) {
                $errorMessages = [];
                foreach ($errors as $error) {
                    $errorMessages[$error->getPropertyPath()] = $error->getMessage();
                }

                return new JsonResponse(['errors' => $errorMessages], 400);
            }

            $product = new Product();
            $product->setName($data['name']);
            $product->setPrice($data['price'] );

            $em->persist($product);
            $em->flush();

            return new JsonResponse([
                'id' => $product->getId(),
                'name' => $product->getName(),
                'price' => $product->getPrice(),
            ], 201);

        }
```
# Relations ve Repository sorguları
* Sonradan ikinci bir Entity ile varolan bir entity için ilişki oluşturursan foreign key hatası alırsın. Bunu düzeltmek için **php bin/console doctrine:database:drop --force** database'i komple sil. Sonra tekrar **php bin/console doctrine:database:create** ve migration **php bin/console doctrine:migrations:migrate** yap. Böylece devam edebilirsin. ✅
* PUT / PATCH / DELETE endpointleri ekle ❌
* Services ve EventSubscriber ile controller logic’i soyutla ❌
* API token veya JWT authentication ekle ❌
* Pagination ve filtering ekle ❌
* Frontend ile tam entegrasyon yap ❌
* Test yaz ve migration yönetimini öğren ❌

