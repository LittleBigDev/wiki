# Add a Custom Discount
## 1. Declare a new quote totals item
```xml title="app/code/Vendor/Modulename/etc/sales.xml"
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Sales:etc/sales.xsd">
 <section name="quote">
   <group name="totals">
     <item name="customer_discount" instance="Vendor\Modulename\Model\Total\Quote\Custom" sort_order="420"/>
   </group>
 </section>
</config>
```

## 2. Set the value of the discount
`app/code/Vendor/Modulename/Model/Total/Quote/Custom.php`
``` php title="app/code/Vendor/Modulename/Model/Total/Quote/Custom.php"
<?php
namespace Vendor\Modulename\Model\Total\Quote;

/**
* Class Custom
* @package Vendor\Modulename\Model\Total\Quote
*/
class Custom extends \Magento\Quote\Model\Quote\Address\Total\AbstractTotal
{
   /**
    * @var \Magento\Framework\Pricing\PriceCurrencyInterface
    */
   protected $priceCurrency;
   
   /**
    * Custom constructor.
    * @param \Magento\Framework\Pricing\PriceCurrencyInterface $priceCurrency
    */
   public function __construct(
       \Magento\Framework\Pricing\PriceCurrencyInterface $priceCurrency
   ){
       $this->priceCurrency = $priceCurrency;
   }
   
   /**
    * @param \Magento\Quote\Model\Quote $quote
    * @param \Magento\Quote\Api\Data\ShippingAssignmentInterface $shippingAssignment
    * @param \Magento\Quote\Model\Quote\Address\Total $total
    * @return $this|bool
    */
   public function collect(
       \Magento\Quote\Model\Quote $quote,
       \Magento\Quote\Api\Data\ShippingAssignmentInterface $shippingAssignment,
       \Magento\Quote\Model\Quote\Address\Total $total
   ){
       parent::collect($quote, $shippingAssignment, $total);
           $baseDiscount = 10; // TODO : replace with your custom calculation
           $discount =  $this->priceCurrency->convert($baseDiscount);
           $total->addTotalAmount('customdiscount', -$discount);
           $total->addBaseTotalAmount('customdiscount', -$baseDiscount);
           $total->setBaseGrandTotal($total->getBaseGrandTotal() - $baseDiscount);
           $quote->setCustomDiscount(-$discount);
       return $this;
   }
}
```


## 3. Display the custom discount in front (totals summary)
### Layout file
```xml title="app/code/Vendor/Modulename/view/frontend/layout/checkout_cart_index.xml"
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
       <referenceBlock name="checkout.cart.totals">
           <arguments>
               <argument name="jsLayout" xsi:type="array">
                   <item name="components" xsi:type="array">
                       <item name="block-totals" xsi:type="array">
                           <item name="children" xsi:type="array">
                               <item name="custom_discount" xsi:type="array">
                                   <item name="component"  xsi:type="string">Vendor_Modulename/js/view/checkout/summary/customdiscount</item>
                                   <item name="sortOrder" xsi:type="string">20</item>
                                   <item name="config" xsi:type="array">
                                       <item name="custom_discount" xsi:type="string" translate="true">Custom Discount</item>
                                   </item>
                               </item>
                           </item>
                       </item>
                   </item>
               </argument>
           </arguments>
       </referenceBlock>
   </body>
</page>
```

### View model knockout
```javascript title="app/code/Vendor/Modulename/view/frontend/web/js/view/checkout/summary/customdiscount.js"
define(
   [
       'jquery',
       'Magento_Checkout/js/view/summary/abstract-total'
   ],
   function ($,Component) {
       "use strict";
       return Component.extend({
           defaults: {
               template: 'Vendor_Modulename/checkout/summary/customdiscount'
           },
           isDisplayedCustomdiscount : function(){
               return true;
           },
           getCustomDiscount : function(){
               return '$10'; // TODO : replace with your custom calculation
           }
       });
   }
);
```

### Template knockout
```html title="app/code/Vendor/Modulename/view/frontend/web/template/checkout/summary/customdiscount.html"
<!-- ko if: isDisplayedCustomdiscount() -->
<tr class="totals customdiscount excl">
   <th class="mark" colspan="1" scope="row" data-bind="text: custom_discount"></th>
   <td class="amount">
       <span class="price" data-bind="text: getCustomDiscount(), attr: {'data-th': custom_discount}"></span>
   </td>
</tr>
<!-- /ko -->
```