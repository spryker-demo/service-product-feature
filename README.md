# Spryker Demo Service Product Feature

## Installation

```
composer require spryker-demo/service-product-feature
```

### Add `SprykerDemo` namespace to configuration

```
$config[KernelConstants::CORE_NAMESPACES] = [
    ...
    'SprykerDemo',
];
```

### Wire the oms command and conditions plugin


### Adjust OMS configuration file


### Add translations


### Import translations

```
console data:import:glossary
```


### Install demo data (optional)

```
composer require spryker-demo/service-product-data-import
```

Demo data for the following entities is provided by the previous composer package:

#### product-attribute-key

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_attribute_key.csv`
* Command: `console data:import:product-attribute-key`

#### product-management-attribute

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_management_attribute.csv`
* Command: `console data:import:product-management-attribute`

#### product-abstract

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_abstract.csv`
* Command: `console data:import:product-abstract`

#### product-concrete

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_concrete.csv`
* Command: `console data:import:product-concrete`

#### product-image

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_image.csv`
* Command: `console data:import:product-image`

#### product-price

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/DE/product_price.csv`
* Command: `console data:import:product-price`

#### product-abstract-store

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/DE/product_abstract_store.csv`
* Command: `console data:import:product-abstract-store`

#### product-stock

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_stock.csv`
* Command: `console data:import:product-stock`

#### merchant-product

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/merchant_product.csv`
* Command: `console data:import:merchant-product`

#### product-approval-status

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/product_abstract_approval_status.csv`
* Command: `console data:import:product-approval-status`

#### product-price-merchant-relationship

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/DE/price_product_merchant_relationship.csv`
* Command: `console data:import:product-price-merchant-relationship`

#### shipment

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/shipment.csv`
* Command: `console data:import:shipment`

#### shipment-method-store

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/DE/shipment_method_store.csv`
* Command: `shipment-method-store`

#### shipment-price

* File: `vendor/spryker/spryker-demo/Bundles/ServiceProductDataImport/data/import/DE/shipment_price.csv`
* Command: `shipment-price`

