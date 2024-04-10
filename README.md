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

### Wire Shipment methood filter plugin

```
# src/Pyz/Zed/Shipment/ShipmentDependencyProvider.php

use SprykerDemo\Zed\ServiceProduct\Communication\Plugin\Shipment\ServiceProductShipmentGroupMethodFilterPlugin;

// ...

    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return array<\Spryker\Zed\ShipmentExtension\Dependency\Plugin\ShipmentMethodFilterPluginInterface>
     */
    protected function getMethodFilterPlugins(Container $container): array
    {
        return [
            new ServiceProductShipmentGroupMethodFilterPlugin(),
        ];
    }
```

### Wire the Merchant OMS condition plugin

```
# src/Pyz/Zed/MerchantOms/MerchantOmsDependencyProvider.php

use SprykerDemo\Zed\ServiceProduct\Communication\Plugin\StateMachine\Condition\IsServiceProductConditionPlugin;

// ...

    /**
     * @return array<\Spryker\Zed\StateMachine\Dependency\Plugin\ConditionPluginInterface>
     */
    protected function getStateMachineConditionPlugins(): array
    {
        return [
            'MarketplaceServiceProduct' => new IsServiceProductConditionPlugin(),
        ];
    }
```

### Wire the OMS condition plugin

```
# src/Pyz/Zed/Oms/OmsDependencyProvider.php

SprykerDemo\Zed\ServiceProduct\Communication\Plugin\Oms\Condition\IsServiceProductConditionPlugin;

// ...

    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return \Spryker\Zed\Kernel\Container
     */
    protected function extendConditionPlugins(Container $container): Container
    {
        $container->extend(self::CONDITION_PLUGINS, function (ConditionCollectionInterface $conditionCollection) {
            // ...
            $conditionCollection->add(new IsServiceProductConditionPlugin(), 'Service/IsServiceProduct');

            return $conditionCollection;
        });

        return $container;
    }
```

### Create new oms subprocess xml file

```
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:oms-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:oms-01 http://static.spryker.com/oms-01.xsd"
>
    <process name="Service">
        <states>
            <state name="waiting for delivery by merchant"/>
        </states>

        <events>
            <event name="check-is-service" onEnter="true"/>
        </events>
    </process>
</statemachine>

```

### Create new state machine subprocess xml file

```
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:oms-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:oms-01 http://static.spryker.com/oms-01.xsd">

    <process name="MerchantServiceProduct01">
        <states>
            <state name="service product purchased"/>
            <state name="ready for delivering service"/>
            <state name="service started"/>
            <state name="service delivered"/>
        </states>

        <transitions>
            <transition happy="true">
                <source>service product purchased</source>
                <target>ready for delivering service</target>
                <event>activate service</event>
            </transition>

            <transition happy="true">
                <source>ready for delivering service</source>
                <target>service started</target>
                <event>start service</event>
            </transition>

            <transition happy="true">
                <source>service started</source>
                <target>service delivered</target>
                <event>deliver service</event>
            </transition>

            <transition happy="true">
                <source>service delivered</source>
                <target>delivered</target>
                <event>complete service</event>
            </transition>
        </transitions>

        <events>
            <event name="activate service" onEnter="true"/>
            <event name="start service" onEnter="true"/>
            <event name="deliver service" onEnter="true"/>
            <event name="complete service" onEnter="true"/>
        </events>
    </process>
</statemachine>

```

### Include service subprocess into OMS.
```
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:oms-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:oms-01 http://static.spryker.com/oms-01.xsd"
>

    <process name="MarketplacePayment01" main="true">

        <states>
            <!--     ...    -->
            <state name="ready for shipment" display="oms.state.ready-for-shipment"/>
            <state name="ready for shipment by merchant" reserved="true" display="oms.state.ready-for-shipment-by-merchant"/>
            <state name="delivered" reserved="true" display="oms.state.delivered"/>
        </states>

        <transitions>
            <!--     ...    -->
            <transition happy="true">
                <source>ready for shipment</source>
                <target>ready for shipment by merchant</target>
                <event>check-is-service</event>
            </transition>

            <transition happy="true" condition="Service/IsServiceProduct">
                <source>ready for shipment</source>
                <target>waiting for delivery by merchant</target>
                <event>check-is-service</event>
            </transition>

            <transition happy="true">
                <source>waiting for delivery by merchant</source>
                <target>delivered</target>
                <event>deliver</event>
            </transition>

            <!--    ...      -->
        </transitions>

        <events>
            <event name="deliver" manual="true"/>
        </events>

        <subprocesses>
            <process>Service</process>
        </subprocesses>
    </process>

    <process name="Service" file="ServiceSubprocess/DummyService01.xml"/>
</statemachine>

```


### Integrate Merchant OMS State machine with new subprocess.

```
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:state-machine-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:state-machine-01 http://static.spryker.com/state-machine-01.xsd"
>

    <process name="MerchantDefaultStateMachine" main="true">

        <states>
            <state name="created"/>
            <state name="new"/>
            <state name="canceled by merchant"/>
            <state name="shipped"/>
            <state name="delivered"/>
            <state name="closed"/>
        </states>

        <transitions>
            <transition happy="true">
                <source>created</source>
                <target>new</target>
                <event>initiate</event>
            </transition>

            <transition happy="true" condition="MarketplaceOrder/IsServiceProduct">
                <source>new</source>
                <target>service product purchased</target>
                <event>check service product purchase</event>
            </transition>

            <transition happy="true">
                <source>new</source>
                <target>shipped</target>
                <event>ship</event>
            </transition>

            <transition>
                <source>new</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition>
                <source>new</source>
                <target>canceled by merchant</target>
                <event>cancel by merchant</event>
            </transition>

            <transition>
                <source>canceled by merchant</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition happy="true">
                <source>shipped</source>
                <target>delivered</target>
                <event>deliver</event>
            </transition>

            <transition happy="true">
                <source>delivered</source>
                <target>closed</target>
                <event>close</event>
            </transition>
        </transitions>

        <events>
            <event name="initiate" onEnter="true"/>
            <event name="check service product purchase" onEnter="true" />
            <event name="ship" manual="true" command="MarketplaceOrder/ShipOrderItem"/>
            <event name="deliver" manual="true" command="MarketplaceOrder/DeliverOrderItem"/>
            <event name="close"/>
            <event name="cancel by merchant" manual="true" command="MarketplaceOrder/CancelOrderItem"/>
        </events>

        <subprocesses>
            <process>MerchantReturn</process>
            <process>MerchantRefund</process>
            <process>MerchantServiceProduct01</process>
        </subprocesses>

    </process>

    <process name="MerchantReturn" file="Subprocess/MerchantReturn"/>
    <process name="MerchantRefund" file="Subprocess/MerchantRefund"/>
    <process name="MerchantServiceProduct01" file="Subprocess/MerchantServiceProduct01"/>


</statemachine>

```

### Install demo data

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

