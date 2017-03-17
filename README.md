# Mage2Katas

Work related to the [Mage2Katas](https://www.youtube.com/channel/UCRFDWo7jTlrpEsJxzc7WyPw) series.

## Environment

Copy the `install-config-mysql-php.dist` file and update the database connection details accordingly:
```bash
zone8@zone8-aurora-r5:/var/www/vhosts/magento2.localhost.com$ cp -f dev/tests/integration/etc/install-config-mysql.php.dist dev/tests/integration/etc/install-config-mysql.php
```

## The Module Skeleton Kata

## The Plugin Config Kata

## The Around Interceptor Kata

### Step one

Create the Plugin test:

```php
// File: app/code/Mage2Kata/Interceptor/Test/Unit/Plugin/CustomerRepositoryPluginTest.php
<?php

namespace Mage2Kata\Interceptor\Plugin;

class CustomerRepositoryPluginTest extends \PHPUnit_Framework_TestCase
{
	public function testItCanBeInstantiated()
	{
		new CustomerRepositoryPlugin();
	}
}
```

Now create the plugin class which satisfies the test:
```php
// File: /var/www/vhosts/magento2.localhost.com/app/code/Mage2Kata/Interceptor/Plugin/CustomerRepositoryPlugin.php
<?php

namespace Mage2Kata\Interceptor\Plugin;

class CustomerRepositoryPlugin
{
	
}
```
The test succeeds.

Now let's add the test for the next step: Adding the plugin method itself:
```php
// File: /var/www/vhosts/magento2.localhost.com/app/code/Mage2Kata/Interceptor/Test/Unit/Plugin/CustomerRepositoryPluginTest.php
<?php

namespace Mage2Kata\Interceptor\Plugin;

use Magento\Customer\Api\CustomerRepositoryInterface;
use Magento\Customer\Api\Data\CustomerInterface;

class CustomerRepositoryPluginTest extends \PHPUnit_Framework_TestCase
{
	/** @var  $_customerRepositoryPlugin CustomerRepositoryPlugin */
	protected $_customerRepositoryPlugin;

	/** @var  $_mockCustomerRepository CustomerRepositoryInterface */
	protected $_mockCustomerRepository;

	/**
	 * @var $_mockCustomerToBeSaved CustomerInterface
	 */
	protected $_mockCustomerToBeSaved;

	public function __invoke(CustomerInterface $customer, $passwordHash)
	{
	}

	protected function setUp()
	{
		$this->_mockCustomerRepository      = $this->getMock( CustomerRepositoryInterface::class);
		$this->_mockCustomerToBeSaved       = $this->getMock( CustomerInterface::class);
		$this->_customerRepositoryPlugin    = new CustomerRepositoryPlugin();
	}

	public function testItCanBeInstantiated()
	{
		$this->_customerRepositoryPlugin;
	}

	public function testTheAroundSaveMethodCanBeCalled()
	{
		$subject        = $this->_mockCustomerRepository;
		$proceed        = $this;
		$customer       = $this->_mockCustomerToBeSaved;
		$passwordHash   = null;
		$this->_customerRepositoryPlugin->aroundSave($subject, $proceed, $customer, $passwordHash);
	}
}
```

We've extracted the `customerRepositoryPlugin` object into a member variable and setup the necessary mock objects (`_mockCustomerRepository`, `_mockCustomerToBeSaved`).

We don't need to mock `passwordHash` because it's a simple scalar value (a string).

Note how, for the `proceed` parameter, we make our test class callable by adding the `__invoke` PHP magic method and then assigning a reference to the class `this` to it. This basically makes the test class a mock object in itself.

The test fails. So let's make it pass by adding the `aroundSave` method to the `CustomerRepositoryPlugin` class.

***Pro tip:*** You can auto-generate the method by pressing `Alt+Return` whilst on the method name.

```php
<?php

namespace Mage2Kata\Interceptor\Plugin;

use Magento\Customer\Api\CustomerRepositoryInterface;
use Magento\Customer\Api\Data\CustomerInterface;

class CustomerRepositoryPlugin
{
	public function aroundSave( $subject, $proceed, $customer, $passwordHash )
	{
		
	}
}
```

The `CustomerRepositoryInterface::save()` method returns a `CustomerInterface` object. Let's write a new test to make sure it does.

```php
<?php

namespace Mage2Kata\Interceptor\Plugin;

use Magento\Customer\Api\CustomerRepositoryInterface;
use Magento\Customer\Api\Data\CustomerInterface;

class CustomerRepositoryPluginTest extends \PHPUnit_Framework_TestCase
{
	/** @var  $_customerRepositoryPlugin CustomerRepositoryPlugin */
	protected $_customerRepositoryPlugin;

	/** @var  $_mockCustomerRepository CustomerRepositoryInterface */
	protected $_mockCustomerRepository;

	/** @var $_mockCustomerToBeSaved CustomerInterface */
	protected $_mockCustomerToBeSaved;

	/** @var $_mockSavedCustomer CustomerInterface */
	protected $_mockSavedCustomer;

	public function __invoke(CustomerInterface $customer, $passwordHash)
	{
		return $this->_mockSavedCustomer;
	}

	protected function setUp()
	{
		$this->_mockCustomerRepository      = $this->getMock( CustomerRepositoryInterface::class);
		$this->_mockCustomerToBeSaved       = $this->getMock( CustomerInterface::class);
		$this->_mockSavedCustomer           = $this->getMock( CustomerInterface::class);
		$this->_customerRepositoryPlugin    = new CustomerRepositoryPlugin();
	}

	protected function callAroundSavePlugin()
	{
		$subject      = $this->_mockCustomerRepository;
		$proceed      = $this;
		$customer     = $this->_mockCustomerToBeSaved;
		$passwordHash = null;
		return $this->_customerRepositoryPlugin->aroundSave( $subject, $proceed, $customer, $passwordHash );
	}

	public function testTheAroundSaveMethodCanBeCalled()
	{
		$result = $this->callAroundSavePlugin();
		$this->assertSame( $this->_mockSavedCustomer, $result);
	}
}
```
Here we mock a new object, `_mockSavedCustomer`, then compare it against what the `aroundSave` method actually returns.

The test fails. To make it pass we need to make our plugin return a `CustomerInterface` object. Here is the code to make it pass:

```php
<?php

namespace Mage2Kata\Interceptor\Plugin;

use Magento\Customer\Api\CustomerRepositoryInterface;
use Magento\Customer\Api\Data\CustomerInterface;

class CustomerRepositoryPlugin
{
	public function aroundSave(
		CustomerRepositoryInterface $subject,
		callable $proceed,
		CustomerInterface $customer,
		$passwordHash = null
	)
	{
		return $proceed($customer, $passwordHash);
	}
}
```
The test succeeds again. Of course, this plugin doesn't actually do anything yet.

