# readme

## translations.sql

- content = table of final entities content
- translation = table with translations for entities
- log = table for logging the processess under the entity
- language = table with language entity with parent-child relationship, which is used for missing final translations
- product = table with entity type product

### columns

- entity_id, entity_type - define the main entity like product, language, category, etc
- version - version of the entity, it's and integer, which is incremented
- data - json data of the entity, data which are not used in SQL queries
- deleted - soft delete of the entity

## app process

### import data

1. new product
- insert data source to tables product, content, translation

2. update product
- upsert data source to tables product, content, translation
- increment the version for product if has changes

3. the is no defined proccess, for update the content from trnaslation, but it can be done by the application / omporting proccess, self-service
- if is translation published, then update the content by the translation
- comparations of the content with translations is by columns logic content.version, translation.content and content.translation_id, translation.id

### exmaple of usage

``` sql
-- first use
INSERT INTO `language` VALUES (1,NULL,'cz','{\"key\": \"value\"}','2024-02-02 18:20:00','2024-02-02 18:50:00',0);
INSERT INTO `product` VALUES (1,NULL,'type1','updatedCode','supplier1','manufacturer1','international1','5','{\"key\": \"value\"}','2024-02-02 18:35:00','2024-02-02 19:00:00',0);
INSERT INTO `content` VALUES (1,1,1,NULL,'product','Updated Content Title','Content Description','Updated Content Text','{\"key\": \"value\"}',2,'2024-02-02 18:10:00','2024-02-02 18:40:00');
INSERT INTO `translation` VALUES (1,1,'product',1,'manual','approved','Updated Title','Description','Text content',2,'2024-02-02 18:00:00','2024-02-02 18:30:00');
INSERT INTO `log` VALUES (1,1,'product','info','Log Description','{\"key\": \"value\"}','2024-02-02 18:25:00');
```

``` sql
-- get the language from user input or set default, : string (iso code), use the parent as sibling of main model
-- php user->getPreferLanguages() // from browser, by ip etc returns [array of language codes]
-- dont forget to check the input with output
-- select language as row, and then fill the language object (first row lang is the main language)
SELECT l.id, l.code, l.parent_id
FROM language l
WHERE l.code IN ('sk', 'cz');
ORDER BY field(l.code, 'sk', 'cz');
```

## product content

``` sql
-- select product with final content from language object
-- i am not using the whole logic of e-commerce, like stock, price, visibility etc
SELECT p.id, p.code, c.title, c.content
FROM product p
LEFT JOIN content c ON p.id = c.product_id AND c.language_id = 1 AND c.entity_type = 'product'
WHERE p.id = 1;
-- if the content is not found, then select by the codes only the content for product and then fill the content object by the application
```
