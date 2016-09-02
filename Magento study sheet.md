#2. Request flow
####Application initialization
######Describe the steps for application initialization
1. `index.php`, which includes `Mage`
2. `Mage::run`, which instantiates `Mage_Core_Model_App`
3. `Mage_Core_Model_App->run()` which initializes the configuration, caches, and hands processing to the front controller

######How and when does Magento load the base configuration, the module configuration, and the database configuration?

loading base configuration: the first step in the `Mage_Core_Model_App->run()` method by grabbing and parsing all the XML files in the app/etc folder.
module configuration: loaded next, with the `_initModules` method which grabs and parses all the module declaration `*.xml` files in `app/etc/modules`. It then loads the config.xml files from the etc directories. local.xml is then reloaded
database configuration: loaded with the config->loadDb method, loads the configuration values stored in the DB into the XML tree.

######How and when are the two main types of setup script executed?

Install/update scripts: run in the `Mage_Core_Model_Resource_Setup::applyAllUpdates()` function after the `local.xml` is loaded before DB config is loaded.

Data scripts: run after all config is loaded: `Mage_Core_Model_Resource_Setup::applyAllDataUpdates()`.

###### When does Magento decide which store view to use, and when is the current locale set?

The store view is decided in the `Mage_Core_Model_App->_initCurrentStore()` function. The locale is set when system configuration is loaded in.

######Which ways exist in Magento to specify the current store view?

1. Environment variables (`$_SERVER['MAGE_RUN_CODE']`, set in index.php)
2. Cookies (the store cookie and the `Mage_Core_Model_App->_checkCookieStore()` function)
3. The HTTP GET parameter `__store` (`Mage_Core_Model_App->_checkGetStore()`)

######When are the request and response objects initialized?

The request object is initialized in the `Mage_Core_Model_App->_initRequest()` function, through the `Mage_Core_Model_App->getRequest()` function, which instantiates the object if it does not exist yet, and returns it.

The response object is initialized in the Mage_Core_Model_App->getResponse() function, like the getRequest function. It is initially called in the Mage_Core_Controller_Varien_Front->getResponse function, as a parameter of instantiating the controller class.

####Front Controller
######Describe the role of the front controller

The role of the front controller (`Mage_Core_Controller_Varien_Front`) is to initialize the routers for the different application areas, match the request to the routes, and send the response.

######Identify uses for events fired in the front controller

Here are some possible uses for events fired in the front controller:

- Modify the request
- Modify the response (output)
- Add another router
- Set cookies

######Which ways exist in Magento to add router classes?

1. In config.xml

```
<config>
  <default>
    <web>
      <routers>
        <routername>
          <area>areahere</area>
          <class>Your Router Class Name Here</class>
        </routername>
      </routers>
    </web>
  </default>
</config>
```

2. An event listener on the `controller_front_init_routers` or `controller_front_init_before` event, and calling the addRouter() method
3. Through the registry: Mage::registry('controller')->addRouter()

######What are the differences between the various ways to add routers?

The event listener method would load the router after the config method. The registry method would be very difficult and would have to be run through an observer.

######Think of possible uses for each of the events fired in the front controller

Here are the events fired in the front controller:

- `controller_front_send_response_before` - could be used to modify output before it is sent, or to set cookies, etc. (Commercebug uses this to add itself)
- `controller_front_send_response_after` - potentially useful to use like a destructor, for clean-up related tasks. I.E., you want to increment the number of visitors on the site, etc.
- `controller_front_init_before` - useful to modify the request/input
- `controller_front_init_routers` - useful to add new routers

####URL rewrites
######Describe URL structure/processing in Magento

Magento URL's are made up of the followup structure: `www.baseurl.mag/frontName/controller/action/params/1`. A front name is set by a module and identifies a URL to a module.

Front names are set in config.xml:

```
<config>
  <frontend>
    <routers>
      <moduleName>
        <use>standard</use>
        <args>
          <module>Namespace_Modulename</module>
          <frontName>frontName</frontName>
        </args>
      </moduleName>
    </routers>
  </frontend>
</config>
```

The front name is then followed by the prefix of the controller you are trying to access (i.e., IndexController = index), and then the prefix of the action you are attempting to run (e.g., viewAction() = view). This is then followed by any parameters in the URL.

######Describe the URL rewrite process

The Magento URL rewrite process allows Magento's API-style urls to become SEO-friendly URLs. The rewrite process involves these steps:

- The `$this->_getRequestRewriteController()->rewrite` method is called in `Mage_Core_Controller_Varien_Front`.
- The `_getRequestRewriteController` method instantiates the `Mage_Core_Model_Url_Rewrite_Request` class.
- The `Mage_Core_Model_Url_Rewrite_Request->rewrite()` method is called. It applies the rewrites from the database and from the config.

######What is the purpose of each of the fields in the core_url_rewrite
table?

- The id_path field - ?
- The request_path field specifies an SEO-friendly URL that can be used
- The target_path field specifies the path to resolve the request_path to.
- is_system specifies if the URL is rewritten by the system, or is a custom rewrite.
- options
- description - custom description
- store_id - store key
- category_id
- product_id

######When does Magento created the rewrite records for categories and
products?

???Catalog_Model_Url, indexer_url

######How and where does Magento find a matching record for the current request?

The request is matched to the rewrite in `Mage_Core_Model_Url_Rewrite_Request->_rewriteDb`. This method calls the `Mage_Core_Model_Url_Rewrite->loadByRequestPath` method which looks for the `request_path` to match. If they match, the system assigns the rewritten url to the `_aliases` property in the `Zend_Controller_Request_Http` object.

####Request routing

######Describe request routing/request flow in Magento

1. The routers are initialized in Mage_Core_Controller_Varien_Front->init()
2. The various front names are collected in the init function with the Mage_Core_Controller_Varien_Router_Standard->collectRoutes function
3. The match function is called on each of the routers. If a match is found, the controller and action names are passed, and the action is dispatched.
4. The output is sent with the Zend_Controller_Request_Http->sendResponse() function.

######Describe how Magento determines which controller to use and how to customize route-to-controller resolution

Magento determines which controller to use by matching the frontname to your module, and matches the controller name from the URL. The route-to-controller resolution process can be customized with a custom router, which is usually stored in the Controller/ folder.

######Which routers exist in a native Magento implementation?

There are five routers in a native Magento implementation: the admin router (Mage_Core_Controller_Varien_Router_Admin), the standard (or frontend) router (Mage_Core_Controller_Varien_Router_Standard), the installation router (Mage_Install_Controller_Router_Install), the CMS router (Mage_Cms_Controller_Router), and the default router (Mage_Core_Controller_Varien_Router_Default).

######How does the standard router map a request to a controller class?

By getting the front name and controller name from the request, and then getting the module associated with that front name. The controller name is then retrieved from the request, and the system merges it all into a path.

######How does the standard router build the filesystem path to a file that might contain a matching action controller?

See above. The `Mage_Core_Controller_Varien_Router_Standard->getControllerFileName()` method does this. The `Namespace_Modulename` and the controller name are passed in. The `Namespace_Modulename` is split on the `_` character, and is rejoined into an array with only the first two elements remaining. The `controllers` directory is retreived with Mage::getModuleDir('controllers', $moduleName). If there are additional parts (i.e., directories) in the original moduleName, they are appended into the path. The name of the controller is CamelCased and joined with `Controller`. Voila! Magento has built the filesystem path to a file that might contain a matching action controller.

######How does Magento process requests that cannot be mapped?

If a request cannot be matched, Magento will initialize the modules IndexController and call the noRoute action. If there is no module, Magento will by default use the Mage_Cms_Controllers_IndexController class and call the noRoute function.

######After a matching action controller is found, what steps occur before the action method is executed?

- The controller class is instantiated, and it is checked to make sure that the actionMethod is defined.
- The request's module name, action name, controller name, and controller module are all set.
- The request parameters are parsed out, and the request is set to dispatched.
- The dispatch function is called, and the full action method name is retreived. It is then checked again to ensure that the method exists.
- The controller's preDispatch function is called, and the action method is called, after ensuring that the preDispatch function hasn't changed anything (i.e., set the request to dispatched = false, or the FLAG_NO_DISPATCH is not set).

####Module initialization
######Describe the steps needed to create and register a new module

To create and register a new module, you need to add a `Namespace_Modulename.xml` file to app/etc/modules. Inside, it should look something like this:

```
<config>
    <modules>
        <Namespace_Modulename>
            <active>true</active>
            <codePool>local</codePool>
            <depends>
                <Mage_Core />
            </depends>
        </Namespace_Modulename>
    </modules>
</config>
```

The module is created. However, to make it do anything, you need to create a Modulename folder inside your Namespace directory inside your codepool. Inside the Modulename folder, you need to create an etc dir and a config.xml. Now you can wire up helpers, blocks, models, etc.

######Describe the effect of module dependencies

A module dependency causes the module's configuration to be loaded before the depending module's configuration.

######Describe different types of configuration files and the priorities of their loading

There are two main configuration file types in Magento: the `Namespace_Modulename.xml` configuration files in the app/etc/modules directory, and the configuration files in the Namespace/Modulename/etc/ directory. The `Namespace_Modulename.xml` file registers the module with some rudimentary config (codePool, dependencies).

Inside the Namespace/Modulename/etc directory, things are a little more complicated. The config.xml file contains module-level configuration, such as blocks, helpers, and events. It is loaded in every request. There can also be an adminhtml.xml file, which specifies configuration for the admin application. It is loaded after the config.xml in requests dealing with the admin store. There is an system.xml file, which configures the interface for the system config application. It is loaded when system config is being initialized. There are also several others: install.xml, wsi.xml, wsdi.xml, api2.xml, compilation.xml, api.xml, jstranslator.xml and widget.xml. They are used and loaded for various tasks in the system.

######What does "Magento loads modules" mean?

Magento loads modules by getting the module declaration files out of app/etc/modules.

######In which order are Magento modules loaded?

Magento modules are loaded in the following order:

- Modules in the Mage_All.xml declaration file
- Modules with the Mage_ prefix (i.e., namespace)
- Then custom modules are loaded

######Which core class loads modules?

`Mage_Core_Model_Config->_loadDeclaredModules`

######What are the consequences of one module depending on another module?

If one module is dependent on another, the dependency module is loaded before the dependent module.

######During the initialization of Magento, when are modules loaded in?

Modules are loaded in after the basic configuration is initialized and the application parameters are registered.

######Why is the load order important?

Load order is important because layouts, events can be dependent on one another.
######What is the difference regarding module loading between Mage::run() and Mage::app()?

Appears to be no difference.

####Design and layout initialization

######Identify the steps in the request flow in which:

– Design data is populated
  - Design data is popluated in the `Mage_Core_Controller_Varien_Action->addActionLayoutHandles()` function, which is called as part of the `Mage_Core_Controller_Varien_Action->loadLayout()` method.

– Layout configuration files are parsed
  - Layout configuration files are parsed in `Mage_Core_Model_Layout_Action->merge` function.

– Layout is compiled
  - Layout is compiled as part of the `Mage_Core_Controller_Varien_Action->generateLayoutXml()` function, which is called in `loadLayout()`.
– Output is rendered
  - Output is rendered with the `Mage_Core_Controller_Varien_Action->renderLayout()` method.

######Which ways exist to specify the layout update handles that will be processed during a request?

- `Mage_Core_Controller_Varien_Action->getLayout->getUpdate->addHandle();`
- The store handle is added by default (`'STORE_'.Mage::app()->getStore()->getCode()`)
- The theme handle is loaded by default (`'THEME_'.$package->getArea().'_'.$package->getPackageName().'_'.$package->getTheme('layout')`)
- The controllers action name is loaded by default
- `Mage_Core_Controller_Varien_Action->getLayout()->getUpdate()->load([handles])`
- Through the update node in a layout xml file. Nest them.
- loadLayout(handle) will not load default handles, perhaps.

######Which classes are responsible for the layout being loaded?

`Mage_Core_Model_Layout_Update` and `Mage_Core_Controller_Varien_Action`

######How are layout xml directives processed?

Layout xml directives are processed by loading all the layout update files into one big layout XML object, similar to how config is processed.

######Which configuration adds a file containing layout xml updates to a module?

In config.xml:
```
<area>
 <layout>
  <updates>
   <unique_identifier>
    <file>file.xml</file>
   </unique_identifier>
  </updates>
 </layout>
</area>
```

######Why is the load order of layout xml files important?

One reason is that layout is loaded in a last-in, last-applied method.

####Flushing data (output)
######Describe how and when Magento renders content to the browser

the `Mage_Core_Controller_Response_Http->sendResponse` method is called. It sends the headers and the response body is directly echoed to the browser.

######Which events are associated with sending output?

- `controller_front_send_response_before`
- `http_response_send_before`
- `controller_front_send_response_after`

######Which class is responsible for sending output?

`Mage_Core_Controller_Response_Http`

######How are redirects handled?

Redirects are made with the `Mage_Core_Controller_Varien_Action->_redirect($path)` method. The redirect headers are set and then, with `Mage_Core_Controller_Response_Http->sendResponse()` are echoed out.

--

####Important Magento xpaths:

- class rewrite: config/global/type/module/rewrite/classidentifier+rewriteinside
- cron: config/crontab/jobs/job_name/schedule/cron_expr, ../job_name/run/model
- event: config/area/events/eventname/observers/modulename/class-method-type
- layout: config/area/layout/updates/modulename/file
- add a new frontname: config/frontend/routers/modulename/use, ../modulename/args/module, frontName
- add a backend controller: config/admin/routers/adminhtml/args/module before="Mage_Adminhtml"
- create a product type: config/global/catalog/product/type/typename/label-model-etc
- add an indexer: config/global/index/indexer/indexer_identifier/model
- configure a DB connection: config/global/resources/identifier/connection
- totals: config/global/sales/quote/totals/total_identifier
- add a payment method: config/default/payment/method_identifier

##Magento products
####Magento product types
- Configurable: `Mage_Catalog_Model_Product_Type_Configurable`
- Simple: `Mage_Catalog_Model_Product_Type_Simple`
- Virtual: `Mage_Catalog_Model_Product_Type_Virtual`
- Grouped: `Mage_Catalog_Model_Product_Type_Grouped`
- Bundle: `Mage_Bundle_Model_Product_Type`
- Downloadable: `Mage_Downloadable_Model_Product_Type`

####Mageto Product Price Models
- Simple, virtual: uses `Mage_Catalog_Model_Product_Type_Price`
- Configurable: `Mage_Catalog_Model_Product_Type_Price_Configurable`
- Grouped: `Mage_Catalog_Model_Product_Type_Price_Grouped`
- Bundle: `Mage_Bundle_Model_Product_Price`
- Downloadable: uses `Mage_Downloadable_Model_Product_Price`

Product pricing models are set in the types `price_model` field. Product prices are indexed through the `price_indexer` node.

Prices can be modified through the `catalog_product_get_final_price` event. The `getFinalPrice` method is called on the product.

The lowest available price is always shown to the customer.

####Catalog product relationships
- Bundle, configurable, and grouped products have relationships, either one to one, one to many, or many to one. The `catalog_product_relation` table is used to tie grouped products, bundle products, and configurable products to their simple children. The `catalog_product_super_link` table is used to tie configurable products to children.

####Magento categories

Category relationships are stored in the `catalog_category_entity` table. If a category's `parent_id` is 0, the category is the root category for the store. The category path is set by concatenating the parent id's of the category together.

####Magento Catalog Price Rules

Catalog price rules apply discounts to products based on the date, product, website and customer group.

When `getFinalPrice()` is called on a product, the event `catalog_product_get_final_price` is fired. This is observed by Mage_CatalogRule_Model_Observer which will then look for any catalog price rule that applies to the product. If applicable, it then looks at the database price table and writes the price back to the product model as a Varien data field final_price.

Within the database, the catalogrule table describes rules, their conditions and their actions. catalogrule_product contains the matched products and some rule information. Meanwhile catalogrule_product_price contains the price after the rule has been applied.

####Magento layered navigation

The layered navigation is configured in the `Mage/Catalog/Block/Layer/*` blocks. It pulls out attributes that are marked `is_filterable`, and loads their filter blocks. Layered navigation is loaded with the `catalog_category_layered` layout handle.

####Magento Product options

Used to specify options or addons for products. Processed with the product type's `_prepareProduct` method. Stored in a seperate table for the quote item, stored on the order item.

####Mage tax factors

- customer group
- location
- tax calculation rate
- product tax class
- tax calculation rule

##Mage DB
Magento DB connection defaults: `core_read`, `core_write`, `default_read`, `default_write`, `core_setup`, `default_setup`

Magento base resource model: `Mage_Core_Model_Resource_Db_Abstract`
Mage base collection model: `Mage_Core_Model_Resource_Db_Collection_Abstract`, inherits from `Varien_Data_Collection_Db` which inherits from `Varien_Data_Collection`

DB collection:
To filter: `Mage::getResourceModel('my/model_collection')->addFieldToFilter('field_name', 0|['eq' => 0])`
To sort: `Mage::getResourceModel('my/model_collection')->setOrder('field_name', 'ASC'|'DESC', )`
To group: `Mage::getResourceModel('my/model_collection')->getSelect()->group('column_to_group_by')`

##Mage EAV
####Mage EAV installer

Installer class: `Mage_Eav_Model_Entity_Setup`
Installer methods: `addEntityType($entityType, ['entityModel', 'table'])`, `createEntityTables($tableName)`, `addAttribute($entityType, 'attribute_code', ['type', 'label', 'input', 'class', 'frontend', 'backend', 'source', 'required', 'user_defined', 'default', 'unique']);`
Data types: char, datetime, decimal, int, text, varchar, entity
