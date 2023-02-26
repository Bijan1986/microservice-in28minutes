# Table of contents
1. [Introduction](#introduction)
   1. [Create Product](#createProduct)
   2. [Get All Product](#getAllProducts)
2. [Some paragraph](#paragraph1)
    1. [Sub paragraph](#subparagraph1)
3. [Another paragraph](#paragraph2)

## Product service <a name="introduction"></a>

<img src="images\productServiceDiagram.png"/>

### Create Product <a name="createProduct"></a>

<img src="images\createProductPostman.png"/>

#### Request body:

```json
{
  "name":"ipad pro",
  "description":"ipad pro X",
  "price":1200
}
```

#### Model:

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.math.BigDecimal;

@Document(value = "product")
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Data
public class Product {
    @Id
    private String id;
    private String name;
    private String description;
    private BigDecimal price;

}
```

#### Repository

```java
@Repository
public interface ProductRepository extends MongoRepository<Product,String> {

}
```

#### Service

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {

    private final ProductRepository productRepository;

    public void createProduct(ProductRequest productRequest) {
        Product product = Product.builder()
                .name(productRequest.getName())
                .description(productRequest.getDescription())
                .price(productRequest.getPrice())
                .build();
        productRepository.save(product);
        log.info("Product {} is saved",product.getId());
    }
    
}

```

#### Controller

```java
@RestController
@RequestMapping("/api/product")
@RequiredArgsConstructor
public class ProductsController {

    private final ProductService productService;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void createProduct(@RequestBody ProductRequest productRequest) {
        productService.createProduct(productRequest);
    }

}

```

#### Application.properties file

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/product-service

```

### Get all products <a name="getAllProducts"></a>


<img src="images\getAllProductsPostman.png"/>


#### Service

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {

    public List<ProductResponse> getAllProducts() {
        List<Product> products = productRepository.findAll();
        return products.stream().map(this::mapToProductResponse).toList();
    }

    private ProductResponse mapToProductResponse(Product product) {
        return ProductResponse.builder()
                .id(product.getId())
                .name(product.getName())
                .description(product.getDescription())
                .price(product.getPrice())
                .build();
    }
    
}

```

#### Controller

```java
@RestController
@RequestMapping("/api/product")
@RequiredArgsConstructor
public class ProductsController {

    private final ProductService productService;

    @GetMapping
    @ResponseStatus(HttpStatus.OK)
    public List<ProductResponse> getAllProducts() {
        return productService.getAllProducts();
    }

}
```


