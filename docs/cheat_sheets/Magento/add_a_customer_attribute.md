# Add a Customer Attribute
## The setup script
In your module, in the `InstallData.php` or `UpgradeData.php` script, create the new customer attribute. In the bellow example, this is InstallData.

```php
<?php
namespace Vendor\Modulename\Setup;

use Magento\Customer\Model\Customer;
use Magento\Eav\Model\Config;
use Magento\Eav\Setup\EavSetup;
use Magento\Eav\Model\Entity\Attribute\SetFactory as AttributeSetFactory;
use Magento\Eav\Setup\EavSetupFactory;
use Magento\Framework\Setup\InstallDataInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;

class InstallData implements InstallDataInterface
{
    private $eavSetupFactory;
    private $eavConfig;
    private $attributeSetFactory;
    
    public function __construct(
        EavSetupFactory $eavSetupFactory,
        Config $eavConfig,
        AttributeSetFactory $attributeSetFactory
    ) {
	    $this->eavSetupFactory     = $eavSetupFactory;
		$this->eavConfig           = $eavConfig;
		$this->attributeSetFactory = $attributeSetFactory;
	}

    public function install(
        ModuleDataSetupInterface $setup, 
        ModuleContextInterface $context
    ) {
        $eavSetup = $this->eavSetupFactory->create(['setup' => $setup]);
        $eavSetup->addAttribute(
            Customer::ENTITY,
            'sample_attribute',
            [
                'type'         => 'varchar',
                'label'        => 'Sample Attribute',
                'input'        => 'text',
                'required'     => false,
                'visible'      => true,
                'user_defined' => true,
                'position'     => 999,
                'system'       => 0,
            ]
        );

        $sampleAttribute = $this->eavConfig->getAttribute(
            Customer::ENTITY,
            'sample_attribute'
        );

        $attributeSet     = $this->attributeSetFactory->create();
        $attributeGroupId = $attributeSet->getDefaultGroupId($attributeSetId);

        // more used_in_forms ['adminhtml_checkout', 'adminhtml_customer',
        // 'adminhtml_customer_address', 'customer_account_edit',
        // 'customer_address_edit', 'customer_register_address']
        $sampleAttribute->addData([
            'attribute_set_id' => $attributeSetId,
            'attribute_group_id' => $attributeGroupId,
            'used_in_forms' => ['adminhtml_customer'],
		]);

        $sampleAttribute->save();
    }
}
```

## Apply the changes
```bash
bin/magento setup:upgrade
bin/magento setup:static-content:deploy
bin/magento c:f
```
