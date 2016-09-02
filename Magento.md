#1. Basics
####High-level Magento architecture
######Describe Magento codepools

In Magento, most code is stored in the `app/code` folder. Within `app/code`, there are three different "code pools": `local`, `community` and `core`. Site-specific modules and customizations are stored in the `local` code pool. Community-built modules or pre-written modules purchased from a developer would usually go into the community module. All Magento Core code belongs in the `core` code pool. As a general rule, you should never edit anything in the `core` code pool. Those files should be considered "untouchable".

When Magento is autoloading a class, it first looks in the `local` code pool. If the file cannot be located in the `local` pool, Magento will next look in the `community` pool and finally in the `core` pool.

######Describe typical Magento module structure

In Magento, all code is stored in a module folder. Modules typically have six folders associated, but there can be as many as one wanted. The six conventional folders are:

 - Model - this directory stores all data-related classes
 - Block - this directory stores blocks, which provide functionality for templates
 - controllers - this directory houses controllers
 - Helper - this directory houses helper classes, and must have a `Data.php` helper
 - etc - stores XML configuration
 - sql - stores install and database scripts

Other commonly-used folders include `data` and `Controller`. Every module also has a `module.xml` file in the `app/etc` directory. This file tells Magento what code pool the module is in, sets the status, and provides basic information.

######Describe Magento templates and layout files location

Magento stores templates and layout files in the `app/design` directory. There are two difference directories in the `app/design` folder: `adminhtml` and `frontend`. Inside each of those, there are directories for each package. Inside the package folder, there is a folder for each theme, and then a `layout` and `template` folder. Layout files are stored in the `layout` folder, and template files are stored in the `template` folder.

######Describe Magento skin and JavaScript files location

Magento stores theme files (CSS, Images, and theme-related JavaScript) in a `skin` folder at the top level of the Magento directory. Javascript libraries are stored in the `js` folder in the same place. Javascript files are stored in a directory named after their their namespace.

The `skin` folder is organized a little bit differently. Inside the skin folder, there are typically two folders representing the two main areas of the Magento application: `frontend` and `adminhtml`. Inside those folders, there is a directory for each theme and then for each sub-theme. The files themselves are then stored in directories by kind: CSS files belong in `css/`, JavaScript files in `js/` and image assets in `images`.

######Identify and explain the main Magento design areas (adminhtml and frontend)

There are two main areas to the look of the Magento application: `adminhtml` and `frontend`. The `adminhtml` area refers to the Admin panel, and the `frontend` area is what customers should see (and if they don't, there's a problem) when they visit a store. Magento differentiates between themes on the two areas, and so Admin theme-related files (Blocks, CSS, some layout files and templates) are typically stored in an `adminhtml` directory wherever they are. Front-end theme-related files, except blocks, are stored in a `frontend` directory. Front-end blocks are stored in the `Block` directory.

######Explain class naming conventions and their relationship with the autoloader

In Magento, class names use the following convention: `Namespace_Modulename_Directoryname_Classname.php`. The autoloader (`lib/Varien/Autoload.php`) interprets the underscores as seperating directories in the class name. The autoloader always looks first in the `local` code pool, then the `community` pool, and then `core` pool. If it cannot be found in the Magento code pools, it will look for the class in the `lib` directory. Only then will it throw an error.

######Describe methods for resolving module conflicts

There are several things that can assist in resolving module conflicts. Module conflicts can be caused by configuration conflicts, rewrite conflicts, and theme conflicts. A configuration conflict may involved two modules extending the same class or making changes to config - solve by explicitly making one `<depend>` on another, thereby loading it first. A rewrite conflict may involve two modules rewriting the same class. Solution? Make one depend on the other and extend its rewrite. Finally, a theme conflict may be caused by a module making a change to a block that another module is looking for. Identify what is going wrong and resolve accordingly. Write code that doesn't majorly mess up layout, if possible.

####Magento configuration

######Explain how Magento loads and manipulates configuration information

Magento loads and manipulates configuration information from XML documents. Magento globs all of the XML into a single document, which allows modules to rewrite or add new information to existing nodes.

######Describe class group configuration and use in factory methods

**Class group configuration**

You configure your classes in your modules `config.xml` file. To configure your models, add a `<models>` node to the `<global>` node. It should look like this:

```
<global>
  <models>
    <Namespace_Modulename>
      <class>Namespace_Modulename_Directoryname</class> <!--Directoryname will usually be "Model"-->
      <resourceModel>typically yourmodule_resource</resourceModel>
    </Namespace_Modulename>
    <whatever_you_put_in_the_resourceModel_node> <!--here you will configure your resource models-->
      <class>Same as above, but this time, add the directory that your resource models will be stored in (typically Resource) at the end.</class>
      <entities>
        <table_name>
          <table>table_name</table>
        </table_name>
      </entities>
    </whatever_you_put_in_the_resourceModel_node>
  </models>
  <resources>
    <Namespace_Modulename>
      <setup>
        <module>Namespace_Modulename</module>
      </setup>
    </Namespace_Modulename>
  </resources>
  <blocks>
    <Namespace_Modulename>
      <class>Directory that Blocks are stored in</class>
    </Namespace_Modulename>
  </blocks>
  <helpers>
    <Namespace_Modulename>
      <class>Director that Helpers are in</class>
    </Namespace_Modulename>
  </helpers>
</global>
```

**Factory methods**

Magento makes heavy use of factory methods. You can load `helper` classes with `Mage::helper('YourNamespace_YourModule/helper')`. By Magento convention, every module has a `data` helper, and it can be referred to with just the namespace and the module name. Models (and resource models) can be loaded with `Mage::getModel('YourNamespace_YourModule/ModelName')` (or `Mage::getResourceModel...`). You can also load models and resource models as singletons, which means that they will be instantiated once per request cycle. After that, the already-instantiated object will be returned. To load a model as a singleton, you use `Mage::getSingleton` (or `Mage::getResourceSingleton`). Blocks are referred to in XML documents or in PHP code the same way: `YourNamespace_YourModule/Your_Block_Name`.

######Describe the process and configuration of class overrides in Magento

In Magento, you can override a class by adding a `<rewrite>` node to the node that corresponds to the type of class you are overriding (model, helper, block).

It should look like this:

```
<model>
  <moduleIdentifier>
    <rewrite>
      <class_identifier>Your_Overriding_Class</class>
    </rewrite>
  </moduleIdentifier>
</model>
```

######Register an Observer

The Event/Observer pattern is utilized to a tremendous extent in Magento. You can register an event like this:

```
<area> <!--can be global, frontend, or admin. If global, the event will be listened for everywhere, but otherwise, only in the area you choose-->
  <events>
    <event_name>
      <observers>
        <Module_Identifier>
          <class>Your_Module_Model_Observer</class>
          <method>awesomeMethod</method>
        </Module_Identifier>
      </observers>
    </event_name>
  <events>
</area>
```

######Identify the function and proper use of automatically available events, including `*_load_after`, etc.

These events can be used to "hook" into different actions in the system and modify results without editing core code. The `*_load_after` event is fired every time a model is loaded, and it is prefixed with the `_eventPrefix` property of the model. This can be set in each model. It provides a way to modify data easily.

#######Setup a cron job

Cron jobs can be set up in Magento inside a modules config.xml file.

```
<config>
  <crontab>
    <jobs>
      <job_name>
        <schedule>
          <cron_expr>Takes cron expressions (ex., 9 10 * * 2017 - this would run every day at 10:09am in 2017)</cron_expr>
        </schedule>
        <run>
          <model>Yourmodule_Identifier/Cron::yourMethodName</model>
        </run
      </job_name>
    </jobs>
  </crontab>
</config>
```

####XML-DOM

######How does the framework discover active modules and their configuration?

* See `Mage_Core_Model_Config`.

Active modules and their configuration are discovered by going into the app/etc/modules/ directory and grabbing every XML file in the directory. The files are then looped through and checked to ensure that the modules are active. The framework then loads the module config files from each modules etc/ directory (Mage_Core_Model_Config->loadModulesConfiguration()) and adds them to the mammoth config object.

######What are the common methods with which the framework accesses its configuration values and areas?

`Mage::getStoreConfig()`, `Mage_Core_Model_Config (through Mage::app()->getConfig()->getNode() or Mage::getConfig->getNode()` and `Mage::getStoreConfigFlag()`

######How are per-store configuration values established in the XML DOM?

Per-store configuration values are read from the database (takes place in `Mage_Core_Model_Config->_loadDb`) or from the module config files (in the Mage_Core_Model_Config->loadModulesConfiguration function). They are stored in `Mage_Core_Model_Config->_xml->stores` property, and can be accessed with `Mage::getConfig()->getNode()` or `Mage::getStoreConfig()`.

######By what process do the factory methods and autoloader enable class instantiation?

The factory methods, depending on which is being called, either look in the registry for an existing object and then instantiate if not found (singleton and helper), or just instantiate a new object (model and block), by translating the name into a string like this: `Namespace_Modulename_Model_Class`. The framework first looks for the class by checking if it's path is registered in the config tree. The autoloader then replaces the underscores with directory separators and includes the file path. If the class is not found, an error is thrown. The factory method then instantiates the class, and returns it.

######Which class types have configured prefixes, and how does this relate to class overrides?

Models, Resource Models, Helpers, and Blocks all have configured prefixes. Classes can be overridden in the module’s etc/config.xml file. Inside the models, helpers, or blocks node add a module identifier node, followed by the rewrite node. Inside the rewrite node, add a node for the class name that you want to override. It’s value should be set to your overriding class name.

######Which class types and files have explicit paths?

Templates, layout files, and controllers all have explicit paths.

######What are the interface and configuration options for automatically fired
events?

Magento fires off different default events in the system, often related to the loading of data. These events enable you to edit data or otherwise add functionality without overriding code.

######What configuration parameters are available for event observers?

- class, which specifies the class to use
- method, which sets the method to call
- type, which sets how to retrieve the class (e.g., model, singleton, etc.)

######What is the structure of event observers, and how are properties accessed therein?

Event observers are instances of Varien_Event_Observer. Varien_Event_Observer extends from Varien_Object, so properties in an observer can be retrieved/assigned with the “magic getters and setters.” Varien_Event_Observer also has a number of predefined getters and setters, such as get/setCallback, get/setName, and others, so they can also be used.

######What configuration parameters are available for cron jobs?

The following configuration parameters are available for cron jobs: `<schedule><cron_expr></cron_expr></schedule>` and `<run><model></model></run>`, which specifies which model/method (separated by a double colon) to run.

####Internationalization

#######Describe how to plan for internationalization of a Magento site

Internationalization is fairly easy in Magento. You will have a unique store view for each language you are using. You can then assign the different store views to sub-directories or sub-domains in Magento. Store views can be created in the admin panel in System->Manage Stores. Once you create a store view, you will need to download the language packs from Magento and configure the store view's locale information appropriately (this is in system config).

#######Describe the use of Magento translate classes and translate files

Magento translate files can be obtained from Magento Connect and are placed in the app/locale folder. They are comma-separated strings with english strings and foreign language equivalents. The Mage_Core_Model_Translate class loads the translation files from the translate/modules node in the config tree, the theme translations in the theme's `locale/[languagecode]_[countrycode]/translate.csv` and the translation rows from the core_translate table. The data is then pulled out in pairs and stored in Mage_Core_Model_Translate's `_data` property. When a helper/block's `__()` method is called, the class looks for the string in the config tree. If found, it returns the foreign-language equivalent.

######Describe the advantages and disadvantages of using subdomains and subdirectories in internationalization

Subdomains are better because they provide a more "finished" look and national brand. They are also easier, slightly, to link content with, and can be hosted on separate servers which allows for localizing boxes by country or region.  However, subdirectories are better because they increase search engine ratings for the main site. Subdirectories increase name recognition for the main brand. All in all, both are viable options.

######Which method is used for translating strings, and on which types of objects is it generally available?

The `__('Your string here')` method is used for translating strings. It is generally available on blocks and helpers.

######In what way does the developer mode influence how Magento handles
translations?

When the developer mode is set, all translations not related to a module are removed. Only module level translations are allowed.

######How many options exist to add a custom translation for any given string?

There are three options:

1. The core_translate table in the database
2. The theme's translations files (`app/design/[area]/[package]/[theme]/locale/[languagecode][countrycode]/translate.csv`)
3. Module-level translation files (registered in the config.xml file and stored in `app/locale/[languagecode]_[countrycode]/[namespace]_[module].csv`)

You register module-level translation files in config.xml like this:

```
<config>
  <area>
    <translate>
      <modules>
        <Namespace_Modulename>
          <files>
            <default>Your_File</default>
          </files>
        </Namespace_Modulename>
      </modules>
    </translate>
  </area>
</config>
```

######What is the priority of translation options?

1. Translations from the core_translate table
2. Translations from the theme's translations files
3. Module-level translations

######How are translation conflicts (when two modules translate the same string) processed by Magento?

Magento handles translation conflicts by prefixing the duplicated string with it's Namespace_Modulename. You can specify to use Namespace_Modulename:String in your code.

#2. Request flow
####Application initialization
######Describe the steps for application initialization
1. Magento routes all requests through index.php, which checks PHP versions, includes the compiler (if so configured), and includes the Mage class. It does some processing as well to set if Magento is running a store or a website.
2. The Mage class then takes over, with the method `run` being called. The run method sets the root directory for Magento and instantiates the Mage_Core_Model_App class, which is the Magento application. It also creates an event collection at this point.
3. The Mage_Core_Model_App->run method is shortly called, and it initializes the PHP environment and the basic configuration (app/etc/local.xml, app/etc/config.xml) and the cache. After initializing these, the run method checks to see it the appropriate response is cached. If so, it returns it. If not, the modules are initialized and the area is loaded. The method ensures that the local config (DB connection, etc.) is loaded, and then passes the request to the front controller. The application is now initialized.

######Describe the role of the system entrypoint, index.php

The index.php file has several important roles.
- It checks the PHP version for compatibility.
- It checks and if found, includes, the compiler configuration.
- It checks for a maintenance file, and includes that if found.
- Index.php includes the Mage class and the bootstrap file.
- It determines if the request pertains to a store or a website, and then it calls the all-important `Mage::run()` method.

######How and when is the include path set up and the auto loader registered?

The include path is set up in the beginning of the Mage.php file, before the Mage class definition, by loading the different code pools into one include path and combining it with the original include path. The autoloader is included after the include path is set up, and the auto loader registers itself with `spl_autoload_register` in it's register method.

######How and when does Magento load the base configuration, the module configuration, and the database configuration?

Magento loads the base configuration as the first step in the Mage_Core_Model_App->run() method by grabbing and parsing all the XML files in the app/etc folder. The module configuration is loaded next, with the `_initModules` method. It (and it's called functions) grab and parse all the module declaration `*.xml` files in `app/etc/modules`. It then loads the config.xml files from the etc directories. The database configuration is then loaded with the config->loadDb method. It loads the configuration values stored in the DB into the XML tree.

######How and when are the two main types of setup script executed?

Install/update scripts are run in the Mage_Core_Model_Resource_Setup::applyAllUpdates() function after the local.xml file is loaded. The function runs any necessary updates (determining any necessary upgrades by comparing the version stored in the database to the version in the configuration file).

Data scripts are run after the local configuration is loaded and the store and request are initiated, by the Mage_Core_Model_Resource_Setup::applyAllDataUpdates(). Its method of actions is fairly similar to the updates function.

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

######What is the purpose of each of the fields in the core_url_rewrite table?

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

######Which ways exist to specify the layout update handles that will be
processed during a request?

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

In the `Mage_Core_Controller_Varien_Action->renderLayout` method, the appropriate blocks are output and their output is added to the response (`Mage_Core_Controller_Response_Http`). The controller action is then finished up, and the `Mage_Core_Controller_Response_Http->sendResponse` method is called. It sends the headers and the response body is directly echoed to the browser.

######Describe how and when Magento flushes output variables using the Front controller

Magento flushes output variables using the Front controller after ???

######Which events are associated with sending output?

- `controller_front_send_response_before`
- `http_response_send_before`
- `controller_front_send_response_after`

######Which class is responsible for sending output?

`Zend_Controller_Response_Abstract` - called in it's child class `Mage_Core_Controller_Response_Http`

######What are possible issues when this output is not sent to the browser
using the typical output mechanism, but is instead sent to the browser
directly?

The correct headers may not be sent.

######How are redirects handled?

Redirects are made with the `Mage_Core_Controller_Varien_Action->_redirect($path)` method. The redirect headers are set and then, with `Mage_Core_Controller_Response_Http->sendResponse()` are echoed out.

##3. Rendering
##Themes in Magento
######How you can use themes to customize core functionality?
Themes should not be used to truly _customize_ core functionality; if you want to really customize core functionality you should employ a custom module. Themes can be used to customize layouts and templates, however. Whether or not those are "functionality" is debatable.

######How can you implement different designs for different stores using Magento themes?

These are set in the `design_change` table. You can set themes or packages on a store-by-store basis.

If you are looking for small changes from store to store, use different themes from the same package. If you are looking for major changes from store to store, use different packages.

######In which two ways you can register custom theme?

- Through the admin panel, in System -> Configuration -> Design.
- Date-based changes can be made in System -> Design.

####Define and describe the use of design packages
######What is the difference between package and theme?

A package is a collection of themes, with at least one (called default) defined. A theme is a collection of related layout, translation, template, image, JavaScript, and CSS files.

######What happens if the requested file is missed in your theme/package?

Magento uses a system called fallbacks to look for missing theme/package files. Fallbacks can also provide a powerful mechanism for theme-inheritance.

Files will be searched for in the currently specified theme. The fallback mechanism will look next in the store's default theme. Then the file will be searched for in the theme named default. Finally, the file is searched for in the base package and the base theme.

There is also a newer theme inheritance mechanism that looks at the theme's etc/theme.xml file's `theme/parent` node for an theme to inherit from. It does not appear to be heavily used, and the only difference is that it will first search for the file based off of that, and then go to the base package.

####Describe the process of defining template file paths
######Which kind of paths (absolute or relative) does Magento use for templates and layout files?

Relative paths are used for templates, and absolute paths are used for layout files. These are the default options, they could be changed...

######How exactly can Magento define which physical file correspond to certain template/layout to use?

Magento uses the `Mage_Core_Model_Design_Package->getFilename()` method to find all filenames. The `$params` are updated and then used to concatenate the correct file path. The `_type` key is particularly useful - here, Magento checks to determine which file type it is looking for, and then changes the base dir accordingly.

######Which classes and methods need to be rewritten in order to add additional directories to the fallback list?

- `Mage_Core_Model_Design_Fallback->_getFallbackScheme(), ->_getLegacyFallbackScheme(), ->getFallbackScheme`.

##Blocks
####Describe the programmatic structure of blocks
######What are blocks used for in Magento?

Magento blocks are chunks of user interface and related logic. They are the core of Magento's `view` system.

######What is the parent block for all Magento blocks?

`Mage_Core_Block_Abstract`

######Which class does each block that uses a template extend?

`Mage_Core_Block_Template`

######In which way does a template block store information about its template file? Does it store an absolute or a relative path to the template?

A template block stores information about its template in the `protected $_template` property. The path is stored relative to the template directory, and can be assigned or retrieved through the `getTemplate()` or `setTemplate()` methods, respectively.

######What is the role of the Mage_Core_Block_Abstract class?

All blocks in Magento inherit from the `Mage_Core_Block_Abstract` class, and it provides much of the base functionality to a block (i.e., access to the request, to layout, setting children/sibling blocks, getting output, caching, etc.).

####Describe the relationship between templates and blocks
######Can any block in Magento use a template file?

No. Only blocks that inherit from `Mage_Core_Block_Template` can use template files.

######How does the $this variable work inside the template file?

The template file is executed inside the context of the block that uses it; therefore, the `$this` variable refers to the block.

######Is it possible to render a template without a block in Magento?

It is theoretically possible, as you could just `include` the template file.

######Is it possible to have a block without a template in Magento?

Yes. `Mage_Core_Block_Text`, for example, does not have a template associated with it. It renders a text node.

####Describe the stages in the lifecycle of a block:
######Which class is responsible for creating an instance of the block?

The `Mage_Core_Model_Layout` template is responsible for creating an instance of a block, through its `_getBlockInstance()` method.

######Which class is responsible for figuring out which blocks should be created for certain pages?

The `Mage_Core_Model_Layout_Update` class is responsible for figuring this out, through the `load()` method.

######How is the tree of blocks typically rendered?

The tree of blocks is typically rendered with the following process:

The `Mage_Core_Controller_Varien_Action::loadLayout()` method is called. It loads the appropriate layout handles, loads the layout xml updates from the files, and marks all ignored/without-adaquate-permissions blocks, then setting the appropriate xml tree. The `Mage_Core_Controller_Varien_Action::generateLayoutUpdates()` method is then called, which generates the block tree by looping through the layout XML tree and expanding it into a tree of block classes with their output method specified. (Initiating the various classes, setting/calling the actions, and modifying the references.)

######Is it possible to create an instance of the block and render it on the page without using the Magento layout?

Yes. You can instantiate the class and call its `toHtml` method.

######Is it possible to create an instance of the block and add it to the current layout manually?

A block can be instantiated manually, with `new BlockName();`. It can be added to the current layout manually with `$this->getLayout()->addBlock(BlockName, $name)`.

######How are a block’s children rendered? Once you added a child to the block, can you expect it will be rendered automatically?

It depends on which block your block inherits from. `Mage_Core_Block_Template`, `Mage_Core_Block_Abstract`, `Mage_Core_Block_Text` do not render their children automatically - you need to manually call the child block with `$this->getChildHtml()`. On the other hand, `Mage_Core_Block_Text_List` automatically renders its children.

######What is a difference in rendering process for different types of blocks?

The main difference in rendering processes for different types of blocks is in rendering children. Some blocks render children automatically, while others require the rendering of child blocks to be specified.

####Describe events fired in blocks
######How can block output be caught using an observer?

`Mage::addObserver('core_block_abstract_to_html_after', function ($observer) { $observer['transport']->getHtml() });`

######What events do Mage_Core_Block_Abstract and Mage_Core_Block_Template fire?

- `Mage_Core_Block_Abstract`:
- - `core_block_abstract_prepare_layout_before` - fired before the `_prepareLayout()` method is called
- - `core_block_abstract_prepare_layout_after` - fired after the `_prepareLayout()` method
- - `core_block_abstract_to_html_before` - fired before the `_toHtml()` method is called.
- - `core_block_abstract_to_html_after` - fired after the `_toHtml()` method is called and useful for modifying output.
- `Mage_Core_Block_Template`:
- - same as above.

####Identify different types of blocks:
######What is the purpose of each of the following block types?
- `Mage_Core_Block_Template` - this is a block that renders a template associated with itself.
- `Mage_Core_Block_Text_List` - this block is useful when you need a block that automatically renders all of its children.
- `Mage_Core_Block_Text` - This block is the generic magento block. It simply renders a text fragment (set with `setText()`.

######Which block type renders its children automatically?

`Mage_Core_Block_Text_List` renders its children automatically.

######Which block type is usually used for a “content” block on Magento pages?

`Mage_Page_Block_Html` appears to be used as the root block. `Mage_Core_Block_Text_List` is used as often as a content block.

####Describe block instantiation
######The block instance can be accessed with the `$this` variable. Other block instances can be accessed with the `Mage::app()->getLayout()->getBlock('block_name')`. A template should not access another block; that is better done in the block or controller.

######How can block instances be accessed from the controller?

`$this->getLayout()->getBlock('block_name')`

######How can block instances be accessed inside install scripts or other model class instances?

`Mage::app()->getLayout()->getBlock('block_name')`

####Explain different mechanisms for disabling block output
######In which ways can block output be disabled in Magento?

You can add the attribute `ignore='1'` to the block. You can `<remove name"nameOfTheBlock" />`.

######Which method can be overridden to control block output?
`_toHtml`

####Describe how a typical block is rendered
######Which class performs rendering of the template?

`Mage_Core_Block_Template`

######Which classes are responsible for figuring out the absolute path for including the template file?

`Mage_Core_Block_Template` and `Mage_Core_Model_Design_Package`

######In which method are templates rendered?

`Mage_Core_Block_Template->fetchView()`

######How can output buffering be enabled/disabled when templates are rendered?

- `$this->getLayout()->setDirectOutput(false)` to enable output buffering
- `$this->getLayout()->setDirectOutput(true)` to disable output buffering

####Design layout, XML schema, and CMS content directives
#####Describe the elements of Magento's layout XML schema, including the major layout directives

Magento uses an XML-based schema for layout. In addition to the ones below, there are two other directives: `<remove name="block_name" />` and `<reference name=""></reference>` which allows you to perform an action on a block specified in another place.

######How are <update />, <block />, and <action /> used in Magento layout?

- `<update />`: The update node updates layouts; it tells Magento: "Merge the specified `handle` and what I put inside the update node."
- `<block />`: The block node is used to add a block to layout. It has several attributes:
- - `type`: used to specify which block class to render
- - `name`: this is the way to refer to the block in layout, in templates, and in code. Must be unique on the page
- - `alias`: provides a shorthand way to refer to the block inside the parent block. Must be unique within the parent block
- - `template`: the template for the block to render
- `<action />`: provides a way to call a PHP method on a block. You specify the method (e.g., method="setText") and specify parameters with `<paramName>value</paramName>`

######Which classes and methods determine which nodes from layout XML correspond to certain URLs?
`Mage_Core_Controller_Varien_Action::loadLayout()` which in turn calls `Mage_Core_Controller_Varien_Action::addActionLayoutHandles()`.

#####Register layout XML files
######How can layout XML files be registered for the frontend and adminhtml areas?

Frontend:

```
<?xml version="1.0">
<config>
 <adminhtml (or frontend)>
  <layout>
   <updates>
    <updates>
     <file>path/to/file.xml</file> <!--relative to the theme's layout directory-->
    </updates>
   </updates>
  </layout>
 </adminhtml>
</config>
```

#####Create and add code to pages:
######How can code be modified or added to Magento pages using the following methods, and in which circumstances are each the above methods more or less appropriate than others?

- Template customizations - you can add javascript, css, and pretty much any PHP you desire to a template. Adding complex PHP to a template is a code smell and the complex code moved to the block. Templates are appropriate to add presentation logic to.
- Layout customizations - you can add blocks, set templates, and call block methods in layout. Layout is appropriate for setting up block parent/child structure and configuring blocks.
- Overriding block classes - you can do pretty much anything, by overriding the class (`<block><identifier><rewrite><block_name_after_id>New_Block_Name</block_name_after_id>`. I'm of the opinion that says you shouldn't overwrite core code without a really good reason to do so - and should never do it, if at all possible, in a distributed extension. Let the store developer do that.
- Registering observers on general block events - again, you can do almost anything by listening to `core_block_abstract_to_html_after` and calling `$observer->getBlock()` in your observer method. This is a good idea most of the time, to avoid rewrites and to avoid abusing layout/templates. Templates should handle presentation logic, however. Be careful of performance, general block events are always fired.

#####Explain how variables can be passed to block instances via layout XML
######How can variables be passed to the block using the following methods, and in which circumstances are each of the above methods more or less appropriate than others?
- From layout xml file: can use this with `<action method="setWhatever"><param1>1</param1></action>`. This is appropriate for block configuration. Not apropriate for very complicated things, and really, limited in capability.
- From controller - `$this->getLayout()->getBlock()->doWhatever($passValue)`. Often appropriate, if necessary, however, it is nice to avoid direct controller interaction with blocks, if possible.
- From one block to another - `$this->getLayout()->getBlock()->doWhatever($passValue)`. Also a fine choice, but I would personally try to use layout first and then the controller before this.
- From an arbitrary location (for example, install/upgrade scripts, models) - `Mage::getSingleton('core/layout')->getBlock()->doWhatever($passvalue);`. I would avoid this way if at all possible. Haven't really seen this done much.

#####Describe various ways to add and customize JavaScript to specific request scopes
######Which block is responsible for rendering JavaScript in Magento?

`Mage_Page_Block_Html_Head`. You can use the `addItem($type, $fileName, $params, $if, $cond)` method.

######Which modes of including JavaScript does Magento support?

You can include it on the page or in the head. Magento doesn't support adding Javascript after the body out of the box.

To include it, you can use layout action:

```
<reference name="head">
 <action method="addJs"><name>Path/from.jsdirectory</name></action>
</reference>
```

To include javascript in a block, you need to ensure that the path is correct, including the JS part of the directory. To help with this, you can use `$this->getJsUrl("normal path relative to js")`, `$this->baseUrl()/your/file.js`, or just specify the path yourself.

######Which classes and files should be checked if a link to a custom JavaScript file isn’t being rendered on a page?

`Mage_Page_Block_Html_Head::getCssJsHtml()`, or anything related to layout to ensure that the nodes are being picked up.

#4. Working with Databases in Magento
###Models, resource models, and collections
####Describe the basic concepts of models, resource models, and collections, and the relationship they have to one another

Models are objects that represent data, often from the database. Resource models interact with the database and the model, reading from the database to the model and writing from the model to the database. Collections are collections of models of the same type.

####Configure a database connection

Database connections are configured in the app/etc/local.xml file:

```
<?xml version="1.0"?>
 <config>
  <global>
   <resources>
    <default_setup>
     <connection>
      <host><![CDATA[hostaddre.ss]]></host>
      <username><![CDATA[user]]></username>
      <password><![CDATA[ReAl1y-$ecur#-pa$sWo9D]]></password>
      <dbname><![CDATA[database-name]]></dbname>
      <initStatements><![CDATA[INIT STATEMENTS]]></initStatements>
      <model><![CDATA[mysql4]]></model>
      <type><![CDATA[pdo_mysql]]></type>
      <pdoType><![CDATA[pdoType]]></pdoType>
      <active>1</active>
     </connection>
    </default_setup>
   </resources>
  </global>
 </config>
 ```

 The connections to use for `core_read`, `core_write`, `default_read`, and `default_write`, `default_setup` and `core_setup` are configured in the config.xml file:

 ```
 <config>
  <global>
   <resources>
    <identifier>
     <connection>
      either <use>identifier</use> or details, as above
 ```
 ####Describe how Magento works with database tables

 Magento interacts with database tables using resource models. In a resource model's `_construct` method, this `init` method is typically called, passing in the main_table identifier and the id field name as parameters. The main table identifier should correspond to an entity:

 ```
 <config>
  <global>
   <model>
    <id_resource>
     <entities>
      <entityName>
       <table>tableName</table>
 ```

 Magento uses these entity names to refer to tables. A tablename can be retreived with `Mage_Core_Model_Resource_Db_Abstract::getTable("entity_name"`.

 ####Describe the load-and-save process for a regular entity

 When the load method is called on a model, the `_beforeLoad` method is called. Next, the resource model is retrieved, and its load method is called. The load method in the resource model executes a query looking for the first row to match the specified field (if none is specified, it looks for the id field). The data is then set to the model instance. The resource model `_afterLoad()` method is called, and then the models after load method is called. Finally the original data is set, and the model is returned.

 When the save method is called, there is a check to ensure that the item has data changes. If it does, a transaction is begun on the database, and the `_beforeSave` method is called, followed by the resource model's `_save` method. This method serializes any specified fields, checks to make sure that the object is unique, and inserts/updates it into the database. Fields are then unserialized, and the `_afterSave` methods are called. The transaction is committed if this is all successful, and the transaction is rolled back if there is an error. The `_afterSaveCommit` method is then called, and `$this` is returned.

 ####Describe group save operations

 Calling the `save` method on a `Mage_Core_Model_Resource_Db_Collection_Abstract` collection model merely loops through the items and calls the save method on each of them.

 ####Describe the role of Zend_Db_Select in Magento

 `Zend_Db_Select` is essentially Magento's alternative to a sql select statement. The various parts of the statement are stored in the `_parts` property. Calling the `Zend_Db_Select::__toString` method generates the parts into the SQL query string.

 ####Describe the collection interface (filtering/sorting/grouping)

 To filter: `Mage::getResourceModel('my/model_collection')->addFieldToFilter('field_name', 0|['eq' => 0])`
 To sort: `Mage::getResourceModel('my/model_collection')->setOrder('field_name', 'ASC'|'DESC', )`
 To group: `Mage::getResourceModel('my/model_collection')->getSelect()->group('column_to_group_by')`

 TALK TO JOSEPH ABOUT THE GROUP BY!

 ####Describe the hierarchy of database-related classes in Magento

 Most resource models extend `Mage_Core_Model_Resource_Db_Abstract`, which extends `Mage_Core_Model_Resource_Abstract`.

 Most collection models extend `Mage_Core_Model_Resource_Db_Collection_Abstract`, which extends `Varien_Data_Collection_Db`, which extends `Varien_Data_Collection`.

 ####Describe the role and hierarchy of setup objects in Magento

 All setup objects extend `Mage_Core_Model_Resource_Setup`. The role of a setup object is to create/modify the structure of the database. There is also a catalog and eav setup resource. The catalog setup resource is necessary when you want to write to the catalog_eav_attribute table.

 ???ASK JOSEPH

 ######Which methods exist to access the table of a resource model?

 `Mage_Core_Model_Resource_Db_Abstract::getMainTable()`
 `Mage_Core_Model_Resource_Db_Abstract::getTable($entityName)`
 `Mage_Core_Model_Resource_Db_Abstract::getValueTable($entityName, $valueType)`

 #######Which methods exist to create joins between tables on collections and on select instances?

 Select instances:
 - `Zend_Db_Select::join($tableName, $condition, $columnsToSelect)`
 - `Zend_Db_Select::joinInner($tableName, $condition, $columnsToSelect)`
 - `Zend_Db_Select::joinLeft($name, $condition, $columns)`
 - `Zend_Db_Select::joinRight($name, $condition, $columns)`
 - `Zend_Db_Select::joinFull($name, $condition, $columns)`
 - `Zend_Db_Select::joinCross($name, $columns)`
 - `Zend_Db_Select::joinNatural($name, $columns)`

 Collections:
 - `Mage_Core_Model_Resource_Db_Collection_Abstract::join($table, $condition, $columns)`

 ######How does Magento support different RDBMSs?

 Magento supports different RDBMSs through the use of database adapters. Database adapters implement the `Varien_Db_Adapter_Interface` interface, and can be specified in local.xml:

 ```
 <config>
  <global>
   <resources>
    <db>
        <table_prefix><![CDATA[]]></table_prefix>
    </db>
    <default_setup>
        <connection>
            <host><![CDATA[localhost]]></host>
            <username><![CDATA[root]]></username>
            <password><![CDATA[tYler1dESigner]]></password>
            <dbname><![CDATA[pleasant_hill]]></dbname>
            <initStatements><![CDATA[SET NAMES utf8]]></initStatements>
            <model><![CDATA[mysql4]]></model>
            <type><![CDATA[pdo_mysql]]></type>Specify the type here
            <pdoType><![CDATA[]]></pdoType>
            <active>1</active>
        </connection>
    </default_setup>
   </resources>
   <resource>
    <connection>
    DEFINE CONNECTION TYPES HERE
     <types>
      <pdo_mysql>
       <adapter>Magento_Db_Adapter_Pdo_Mysql</adapter>
       <class>Mage_Core_Model_Resource_Type_Db_Pdo_Mysql</class>
       <compatibleMode>1</compatibleMode>
      </pdo_mysql>
     </types>
    </connection>
   </resource>
  </global>
 </config>
 ```

 ######How do table name lookups work, and what is the purpose of making table names configurable?

 Table name lookups take place in the `Mage_Core_Model_Resource::getTableName($modelEntity)` method. The model entity string is split on the `/` character, and then the entity configuration is pulled from the configuration tree.

 Table names are configurable because this way provides a central repository and access point for table names. It is much more manageable in the instance that a table name needs to be changed in the future.

 ######Which events are fired automatically during CRUD operations?

 Before save:
 - `Mage::dispatchEvent('model_save_before', array('object'=>$this));`
 - `Mage::dispatchEvent($this->_eventPrefix.'_save_before', ['data_object' => $this, $this->_eventObject => $this]);`

 After Save:
 - `Mage::dispatchEvent('model_save_after', array('object'=>$this));`
 - `Mage::dispatchEvent($this->_eventPrefix.'_save_after', ['data_object' => $this, $this->_eventObject => $this]);`

 Before delete:
 - `Mage::dispatchEvent('model_delete_before', array('object'=>$this));`
 - `Mage::dispatchEvent($this->_eventPrefix.'_delete_before', ['data_object' => $this, $this->_eventObject => $this]);`

 After delete:
 - `Mage::dispatchEvent('model_delete_after', array('object'=>$this));`
 - `Mage::dispatchEvent($this->_eventPrefix.'_delete_after', ['data_object' => $this, $this->_eventObject => $this]);`

 After delete commit:
 - `Mage::dispatchEvent('model_delete_commit_after', array('object'=>$this));`
 - `Mage::dispatchEvent($this->_eventPrefix.'_delete_commit_after', ['data_object' => $this, $this->_eventObject => $this);`

 Before load:
 - `Mage::dispatchEvent('model_load_before', ['object' => $this, 'field' => field to load on, 'value' => value]);`
 - `Mage::dispatchEvent($this->_eventPrefix.'_load_before', ['data_object' => $this, $this->_eventObject => $this, 'object' => $this, 'field' => field to load on, 'value' => value]);`

 After load:
 - `Mage::dispatchEvent('model_load_after', array('object'=>$this));`
 - `Mage::dispatchEvent($this->_eventPrefix.'_load_after', ['data_object' => $this, $this->_eventObject => $this]);`

 ######How does Magento figure out if a save() call needs to create an INSERT or an UPDATE query?

 If the model instance has an id, Magento checks whether the id exists in the database. If it does, Magento uses an update query. Otherwise, it uses an insert query.

 ######How many ways exist to specify filters on a flat table collection?

 3:
 - `Varien_Data_Collection_Db::addFieldToFilter('field', '[condition]')`
 - `Varien_Data_Collection::addFilter('field', 'value', 'type')`
 - `Mage_Core_Model_Resource_Db_Collection_Abstract::getSelect()->where(['field' => 'value'])`

 #######Which methods exist to influence the ordering of the result set for flat table collections? How do the methods differ?

 - `Varien_Data_Collection_Db::setOrder||addOrder($field, 'DESC'|'ASC')` - adds order to the end, giving it last priority
 - `Varien_Data_Collection_Db::unshiftOrder($field, 'DESC'|'ASC')` - adds order to the beginning, giving it first priority

 ######Why and how does Magento differentiate between setup, read, and write database resources?

This is because Magento supports master/slave database configuration setups. The different database resources can be configured in app/etc/local.xml.

####Install/upgrade scripts
######Describe the install/upgrade workflow

The install/upgrade workflow is rather simple. The scripts are typically titled with `mysql4-install-version1-version2` or `mysql4-upgrade-version1-version2`. You also need to update the version specified in config.xml, and then your script should run.

######Write install and upgrade scripts using set-up resources

There are a number of set-up resources that are typically used: `Mage_Eav_Model_Entity_Setup` and `Mage_Core_Model_Resource_Setup`. An install script is included into one of these classes, and typically does the following four things:

1. Aliases $this to $installer
2. Calls the $installer->startSetup() method
3. Runs installation code
4. Calls the $installer->endSetup() method

######Identify how to use the DDL class in setup scripts

The DDL class can be used to provide database-agnostic setup scripts. You use it by calling `$installer->getConnection()->newTable()`

######Under which circumstances are setup scripts executed?

Setup scripts are executed when the version specified in config.xml is higher than the version specified in `core_resource`.

######What is the difference between the different classes used to execute setup scripts?

The main difference is that some of them have additional or different functionality defined on top of the base class.

######Which is the base setup class for flat table entities, and which one the base for EAV entities?

The base setup class for flat table entities is `Mage_Core_Model_Resource_Setup` and the base setup class for EAV entities is `Mage_Eav_Model_Entity_Setup`.

######Which methods are generally available in setup scripts to manipulate database tables and indexes?

Here are a number of generally available ones for setup scripts dealing with flat tables:

- `getTable($tableName)` and `getTable($tableName, $realTableName)`
- `getTableRow($table, $idField, $id)`
- `deleteTableRow($table, $idField, $id)`
- `updateTableRow($table, $idField, $id, $field, $value)`
- `tableExists($table)`
- `run($sql)`
- `getIdxName($tableName, $fields, $indexType = '')`
- `getFkName($priTableName, $priColumnName, $refTableName, $refColumnName)`

######What is the difference between addAttribute() and updateAttribute() in EAV setup scripts?

addAttribute both updates existing attributes and creates new ones, while updateAttribute only updates existing attributes.

######How can you implement a rollback in Magento?

Magento does not currently support rollback scripts; therefore, you can write one manually using existing Magento functionality: `Mage_Core_Model_Resource_Setup::deleteTableRow()`, `Mage_Core_Model_Resource_Setup::removeAttribute()`. You will have to run it manually, or increment your version numbers.

#5. Entity-Attribute-Value (EAV) Model
##EAV model concepts
####Define basic EAV concepts and class hierarchy

The basic EAV concept is storing data (values) for entities (models) in multiple tables, defined as a specific attribute. EAV models, like their flat-table counterparts, inherit from `Mage_Core_Model_Abstract`. The difference is in the resource model: while flat-table models inherit from `Mage_Core_Model_Resource_Db_Abstract`, EAV models inherit from the `Mage_Eav_Model_Entity_Abstract` class.

####Describe the database schema for EAV entities

EAV entity types are defined in the `eav_entity_type` table, and then mapped to entities. Attributes are defined in the `eav_attribute` table, and various data related to attributes is scattered through the various `eav_attribute_` tables.

####Describe the EAV entity structure and its difference from the standard core resource model

EAV entities typically have an `entity` table, such as `catalog_product_entity`. While some data is usually stored alongside the entity in the `entity` table, the bulk of the data is stored in type-specific tables, such as `catalog_product_entity_decimal`, along with an `entity_id` that maps the data back to its parent entity and an `attribute_id` that matches the value to an attribute.

This is different from a typical flat table model, which stores all data into one table.

Bottom line: EAV is row-based, while flat models are column based.

####Describe the EAV load-and-save process and its differences from the regular load-and-save process

Load:
First, the base row (entity) is loaded. Then the various attributes associated with the entity are loaded, and finally their values are retrieved from their respective database tables. This is different from the regular load process where all the data from a model is retrieved with one SQL call.

Save:
The attributes associated with the entity are loaded, then the attribute values are processed. The base row is saved, and the attribute values are saved into their tables. The regular load process just inserts the data, and doesn't have to deal with the multiple tables.

######Which classes in Mage_Eav are used as resource models and which are used as regular models?

All the `Mage_Eav_Model_Resource_` classes are resource models, as well as the `Mage_Eav_Model_Entity_Abstract` and `Mage_Eav_Model_Entity_Setup` classes. The other `Mage_Eav_`-prefixed models are regular models.

######How do flat and EAV resource models differ?

Flat and EAV resource models differ in two ways: default functionality and inheritance chains. Flat resource models typically inherit from `Mage_Core_Model_Resource_Db_Abstract`, while EAV resource models typically inherit from `Mage_Eav_Model_Entity_Abstract`. EAV resource models also have functionality that allows them to save attributes, as well as their flat rows.

######What are the advantages and disadvantages of EAV over flat table resource models?

The main advantage of EAV over flat table resource models is it's flexibility: you can add nearly any piece of data you want to an entity without changing your database structure.

The main disadvantage of using EAV over flat table resource models is that EAV is slow, because of the added SQL necessary to retrieve data from various tables.

######Which entities in a native Magento installation use EAV resource models and why?

`customer/customer`, `customer/address`, `catalog/category`, `catalog/product`, `sales/order`, `sales/order_invoice`, `sales/order_creditmemo`, `sales/order_shipment`, and `eav/attribute_option` all use EAV resource models, for their flexibility.

######How are store and website scope attribute values implemented in the Magento EAV system?

The store code is stored in the `store_id` column in the database tables, and therefore values are associated wiht a specific store.

######How does the model distinguish between insert and update operations?

For the base rows, it checks to see if the `entity_id` exists. For the attributes, it seperates the different attributes into three categories: delete, update, and insert. It distinguishes between update and insert opperations: update: the attribute value is not empty and if the value is different from the current value, then it is an update. If the attribute was not originally defined, then it is an insert.

######How do load and save processes for EAV entities differ from those for flat table entities? What parts are identical?

See the above answer. The identical part is that they both have to load/save from a base row; EAV also must load/save attributes.

##Attributes management
####Identify the purpose of attribute frontend, source, and backend models

- Frontend: `Mage_Eav_Model_Entity_Attribute_Frontend_Abstract` - used for displaying the attribute and generating HTML for it
- Backend: `Mage_Eav_Model_Entity_Attribute_Backend_Abstract` - used for validating, converting and saving the attribute data
- Attribute: `Mage_Eav_Model_Entity_Attribute_Abstract` - used to store the attribute data
- Source: `Mage_Eav_Model_Entity_Attribute_Source_Abstract` - used to retrieve attribute options and models

####Describe how to implement the interface of attribute frontend, source, and backend models:
######How do attribute models, attribute source models, attribute backend models and attribute frontend models relate to each other?

See the above answer: attribute models house the data, attribute backend models save the data, attribute source models specify the attribute options, and attribute frontend models display the data.

######Which methods have to be implemented in a custom source model (or frontend model or backend model)?

In a custom source model, you have to implement the `getAllOptions()` method, which returns an array of option values and labels.

In a custom backend model, you don't have to implement any specific methods, so long as you inherit `Mage_Eav_Model_Entity_Attribute_Backend_Abstract`. However, you may want to override/extend some of the existing methods, such as `validate()` or `before/afterSave()`.

In a custom frontend model, you don't have to implement any specific methods.

######Can adminhtml system configuration source models also be used for EAV attributes?

Not on their own, however, you could wrap the class with an adapter that implements the `getAllOptions()` method by calling the `toArray()` method.

######What is the default frontend model (and source and backend model) for EAV attributes?

Frontend: `Mage_Eav_Entity_Attribute_Frontend_Default`
Backend: `Mage_Eav_Entity_Attribute_Backend_Default`
Source: `Mage_Eav_Model_Entity_Attribute_Source_Config`

######Does every attribute use a source model?

No. Not every attribute uses a source model, but only the ones with a `select` or `multiselect` frontend input, or where the `source_model` varien data key is defined.

######Which classes and methods are related to updating the EAV attribute values in the flat catalog tables? What factors allow for attributes to be added to flat catalog tables?

The classes and methods related to updating the EAV attribute values in the flat catalog tables are:

`Mage_Catalog_Model_Resource_Product_Flat_Indexer`, `::updateAttributes($attribute, $storeId)`, `::updateEavAttributes($storeId())`, and `Mage_Catalog_Model_Resource_Category_Flat::synchronize()`.

The attribute's `used_in_product_listing` value needs to be true.

######How are store-scoped entity attribute values stored when catalog flat storage is enabled for that entity type?

The store id is appended to the end of the table name, so for products: `catalog_product_flat_1`. For categories: `catalog_category_flat_store_1`.

######Which frontend, source, and backend models are available in a stock Magento installation?

Source models:

- `Mage_Eav_Model_Entity_Attribute_Source_Boolean` - a basic boolean source model (yes/no)
- `Mage_Eav_Model_Entity_Attribute_Source_Config` - a source model with options pulled fro a config path. Must be extended to use.
- `Mage_Eav_Model_Entity_Attribute_Source_Store` - source model for store ids
- `Mage_Eav_Model_Entity_Attribute_Source_Table` - a DB-based source model, pulling specified source options from the `eav_attribute_option` table, and value options from the `eav_attribute_option_value` table.

Frontend models:

- `Mage_Eav_Model_Entity_Attribute_Frontend_Datetime` - a date frontend model
- `Mage_Eav_Model_Entity_Attribute_Fronted_Default` - a simple extension of the abstract model

Backend models:

- `Mage_Eav_Model_Entity_Attribute_Backend_Array` - a backend model for attributes with multiple values
- `Mage_Eav_Model_Entity_Attribute_Backend_Datetime` - a backend model for date values
- `Mage_Eav_Model_Entity_Attribute_Backend_Default` - a simple extension of the abstract model
- `Mage_Eav_Model_Entity_Attribute_Backend_Increment` - simple increment support for the id
- `Mage_Eav_Model_Entity_Attribute_Backend_Serialized` - support for serialized data
- `Mage_Eav_Model_Entity_Attribute_Backend_Store` - for saving an associated store id with an attribute
- `Mage_Eav_Model_Entity_Attribute_Backend_Time_Created` - support for a `created_at` timestamp
- `Mage_Eav_Model_Entity_Attribute_Backend_Time_Updated` - support for a `updated_at` timestamp

######How do multi-lingual options for attributes work in Magento?

You can have attributes on a store-by-store basis, so one for each store with a different language. TALK TO JOSEPH!

######How do you get a list of all options for an attribute for a specified store view in addition to the admin scope?

`Mage_Eav_Model_Resource_Entity_Attribute_Option_Collection::setStoreFilter($storeId, true)`

####Describe how to create and customize attributes

Attributes can be created with `Mage_Eav_Model_Entity_Setup::addAttribute($entityTypeId, $code, array(attributes))`. They can be customized with `addAttribute()` or `udpateAttribute($entityTypeId, $id, $field, $value)`.

######Which setup methods are available to work with EAV entities?

`Mage_Eav_Model_Entity_Setup::addENtityType($code, $params)`, `::updateEntityType($code, $field, $value)`, `::removeEntityType($id)`, `::getEntityTypeId($entityTypeId)`, `::getEntityType($id, $field)`

######How can an EAV setup class be instantiated in a setup script if not specified in the XML <class> configuration for a setup resource?

By defining `$this` or `$installer` to `new Mage_Eav_Model_Entity_Setup()`. ASK JOSEPH!!!

######What is the difference between addAttribute() and updateAttribute()?

`addAttribute` both updates existing attributes and creates new ones, while `updateAttribute` only updates existing attributes.

######What are the advantages of using a custom setup class for manipulating EAV attributes in a custom module?

The advantages are greater flexibility and more specific queries, more vaildation rules, and the ability to use your own table. Also, you can have your own custom setup function for a specific attribute type.

#6. Adminhtml
##Common structure/architecture
####Describe the similarities and differences between adminhtml and frontend interface and routing
######Which areas in configuration are only loaded for the admin area?

The `adminhtml`, `menu`, `acl` and `admin` nodes are only loaded for the admin area.

######What is the difference between admin and frontend controllers?

Their inheritance chains are different, because admin controllers extend `Mage_Adminhtml_Controller_Action`, while frontend controllers extend `Mage_Core_Controller_Varien_Action`.

######When does Magento figure out which area to use on the current page?

This is determined in the router's `match()` method, when the frontname is found to be `admin`.

######How you can make your controller work under the `/admin` route?

To make your controller work under the `/admin` route, you need to add it to the `<admin>` node in `config.xml`:

```
<admin>
    <routers>
        <adminhtml>
            <args>
                <modules>
                    <Namespace_Modulename before="Mage_Adminhtml">Namespace_Modulename_Adminhtml</Namespace_Modulename>
                </modules>
            </args>
        </adminhtml>
    </routers>
</admin>
```

####Describe the components and types of cache clearing using the adminhtml interface
######At which moment does Magento check if a user is logged in or not?

When the `controller_action_predispatch` observer is fired, the `Mage_Admin_Model_Observer::actionPreDispatchAdmin()` method is listening. It checks if the user is logged in, and, if not, forwards them to the login screen.

######Which class do most Magento adminhtml blocks extend?

`Mage_Adminhtml_Block_Template` or `Mage_Core_Block_Template`

######What are the roles of adminhtml config?

The adminhtml config (`adminhtml.xml`) is used to configure menu items and permissions for the admin panel.

######What are the differences between the different cache types on the admin cache cleaning page?

Magento has eight different cache types:
- configuration - caches system and module config data.
- Layouts - caches layout files
- Block HTML output - caches the output from individual blocks
- Translations - stores translation files
- Collections - stores collections data files
- EAV types and attributes - caches entity type declarations
- Web services configuration - caches api.xml
- Web services configuration - caches api2.xml

In addition, there are these additional three design related caches:

- Catalog images
- Swatch images
- CSS/JS from the theme combined to one file

######What is the difference between “flush storage” and “flush Magento cache”?

"Flush cache storage" always flushes all 8 main cache types, while "flush Magento cache" also flushes all types. Twice. Not sure I understand this.

######How you can clear the cache without using the UI?

Calling `Mage::app()->cleanCache([arrayOfTags])`, or `Mage::app()->getCacheInstance()->flush()`

##Forms in Magento
####Define form structure, form templates, grids in Magento, and grid containers and elements:
######Which block does a standard Magento form extend?

`Mage_Adminhtml_Block_Widget_Form`

######What is the default template for a Magento form?

`widget/form.phtml`

######Describe the role of a form container and its template.

A form container acts as a parent block to the form block, setting buttons on the page, including the child block, etc. The template includes scripts, outputs the form html, and displays content, in general.

######Describe the concept of Form elements, and list system elements implemented in Magento

Form elements are classes that roughly correspond to an HTML form field by the same name. Here is a list of system elements implemented in Magento:

- `Varien_Data_Form_Element_Button` - system implementation of a form button
- `Varien_Data_Form_Element_Checkbox` - system implementation of a form checkbox
- `Varien_Data_Form_Element_Checkboxs` - system implementation of a form select
- `Varien_Data_Form_Element_Collection` - a collection of form elements
- `Varien_Data_Form_Element_Column` - a system implementation of a form column
- `Varien_Data_Form_Element_Date` - a date-selector form element
- `Varien_Data_Form_Element_Datetime` - a date-selector form element that extends `Varien_Data_Form_Element_Date`
- `Varien_Data_Form_Element_Editor` - an editor form element with WYSIWYG capabilities
- `Varien_Data_Form_Element_Fieldset` - a form fieldset implementation
- `Varien_Data_Form_Element_File` - a file element
- `Varien_Data_Form_Element_Gallery` - a category form input image element
- `Varien_Data_Form_Element_Hidden` - a hidden form element
- `Varien_Data_Form_Element_Image` - a category form input image element
- `Varien_Data_Form_Element_Imagefile` - a form image file element
- `Varien_Data_Form_Element_Label` - a form label element
- `Varien_Data_Form_Element_Link` - a link element
- `Varien_Data_Form_Element_Multiline` - multiline text element
- `Varien_Data_Form_Element_Multiselect` - a select input with multiple values
- `Varien_Data_Form_Element_Note` - a form note element
- `Varien_Data_Form_Element_Obscure` - a text element to obscure text (like a password input)
- `Varien_Data_Form_Element_Password` - a password input
- `Varien_Data_Form_Element_Radio` - a radio-button input
- `Varien_Data_Form_Element_Radios` - a radio-button collection
- `Varien_Data_Form_Element_Reset` - a form reset element
- `Varien_Data_Form_Element_Select` - a form select element
- `Varien_Data_Form_Element_Submit` - a form submit element
- `Varien_Data_Form_Element_Text` - a form text element
- `Varien_Data_Form_Element_Textarea` - a form textarea element
- `Varien_Data_Form_Element_Time` - a form time element

######Describe the concept of fieldsets.

A fieldset is a group of related form elements. Magento implements the fieldset with the `Varien_Data_Form_Element_Fieldset` class. You can add a field with the `::addField` method, and retreive the HTML with the `::getElementHtml()` method.

######How can you render an element with a custom template?

You will need to override the element's `getElementHtml()` method, and return the contents of the template you want to render.

##Grids in Magento
####Create a simple form and grid for a custom entity
####Describe how to implement advanced Adminhtml Grids and Forms, including editable cells, mass actions, totals, reports, custom filters and renderers, multiple grids on one page, combining grids with forms, and adding custom JavaScript to an admin form

######Which block class do Magento grid classes typically extend?

`Mage_Adminhtml_Block_Widget_Grid`

######What is the default template for Magento grid instances?

`widget/grid.phtml`

######How can grid filters be customized?

Grid filters can be customized by overriding the filters `getCondition()` method. Grid filters are set with the `filter` attribute for a column, and the `filter_index` key can specify what to filter against in a where (`WHERE filter_index = value`)

######How does Magento actually perform sorting/paging/filtering operations?

Filtering operations are performed by taking the filter query in the `getCondition` and calling `Varien_Data_Collection_Db::addFieldToFilter()` with it.

Sorting operations are performed by calling the `Varien_Data_Collection_Db::setOrder()` method.

Paging operations are also performed in the collection, with the `getSelect()->renderPage()`.

######What protected methods are specific to adminhtml grids, and how are they used?

There are a number of protected methods specific to adminhtml grids:

- `Mage_Adminhtml_Block_Widget_Grid->_prepareLayout()` - creates the child button blocks
- `Mage_Adminhtml_Block_Widget_Grid->_setFilterValues()` - adds column filters to the collection
- `Mage_Adminhtml_Block_Widget_Grid->_addColumnFilterToCollection()` - actually does the work of adding the column filters to the collection (via the filter calback, or add field to filter)
- `Mage_Adminhtml_Block_Widget_Grid->_setCollectionOrder()` - orders the collection
- `Mage_Adminhtml_Block_Widget_Grid->_prepareCollection()` - one of the three most important. Prepares and loads the collection object (renders the filters, orders)
- `Mage_Adminhtml_Block_Widget_Grid->_decodeFilter()` - decodes the URL encoded filter value
- `Mage_Adminhtml_Block_Widget_Grid->_preparePage()` - sets information related to Paging
- `Mage_Adminhtml_Block_Widget_Grid->_prepareColumns()` - sets the columns, also one of the most important three
- `Mage_Adminhtml_Block_Widget_Grid->_prepareMassactionBlock(), _prepareMassaction(), _prepareMassactionColumn` - `_prepareMassactionBlock()` is one of the big three, and these are all related to setting up the mass actions.
- `Mage_Adminhtml_Block_Widget_Grid->_prepareGrid()` - calls the big three
- `Mage_Adminhtml_Block_Widget_Grid->_afterLoadCollection()` - can be used to do work on the collection items

There are also a number of additional methods related to exports.

######What is the standard column class in a grid, and what is its role?

The standard column class in a grid is `Mage_Adminhtml_Block_Widget_Grid_Column`. It houses the data about the collection.

######What are column renderers used for in Magento?

Column renderers are used to manipulate and display the values in each column. They are set in the `Mage_Adminhtml_Block_Widget_Grid::addColumn()` method, with the `renderer` key.

######How can JavaScript that is used for a Magento grid be customized?

You can define a function in your grid called `getAdditionalJavascript()` which can return javascript.

######What is the role of the grid container class and its template?

The grid container class manages the buttons and labels around the grid and acts as the grid's parent block. The template renders the buttons and labels and the grid.

######What is the programmatic structure of mass actions?

Mass actions are blocks that extend `Mage_Adminhtml_Block_Widget_Grid_Massaction_Abstract`. You can add a new mass action by calling it's `addItem()` method, with an itemId, and an array describing the item.

Here is an example:

```
$item = [
  'label' => 'My massaction',
  'url' => $this->getUrl('*/*/myAction'),
];
```

The grid is submitted with Javascript, and the Javascript can be customized by overriding the `Mage_Adminhtml_Block_Widget_Grid_Massaction_Abstract::addItem()` method.

##System configuration
####Define the basic terms, elements, and structure of system configuration XML:

System configuration XML is used to

######How can elements in system configuration be rendered with a custom template?

By setting the front-end model to a custom block, that extends Mage_Adminhtml_Block_System_Config_Form_Field, and returns the template in the `_getElementHtml()` method.

######How does the structure of system.xml relate to the rendered elements in the System Configuration view?

The structure of `system.xml` relates by dictating what elements go where. It also specifies the frontend model/type, as well as the source model, sorting order, and what configuration scopes the element is to appear in.

######How can the CSS class of system configuration elements be changed?

By setting the `<frontend_class>` node to the desired value.

######What is the syntax for specifying the options in dropdowns and multiselects?

First, you set your `<frontend_type>` to `select` or `multiselect`. Then you set your `<source_model>` to the desired model that will return your options. That class should have a method called `toArray()` that returns your options in a value/label array. It is also helpful to have a method called `toOptionArray()` that returns an array of arrays, with each nested array as an option with value/label keys and values set for those keys.

######Which classes are used to parse and render system configuration XML?

The `Mage_Core_Model_Config_System` class loads the system configuration XML, and parses it. `Mage_Adminhtml_Block_System_Config_Form` parses out the fields into a `Varien_Data_Form`. The `Mage_Adminhtml_Block_System_Config_Tabs` class parses out the tabs xml into tabs, and renders them in the `system/config/tabs.phtml` template. The `Mage_Adminhtml_Model_Config_Data` class loads the config out of the database.

######What is the syntax to specify a custom renderer for a field in system configuration?

`<frontend_model>Your_Custom_renderer</frontend_model>`. The custom renderer should extend `Mage_Adminhtml_Block_System_Config_Form_Field` and override the `_getElementHtml()` method.

######How does Magento store data for system configuration?

Magento stores data for system configuration in the `core_config_data` table, which is represented by the `Mage_Adminhtml_Model_Config_Data`.

######What is the difference between Mage::getStoreConfig(...) and Mage::getConfig()->getNode(...)?

`Mage::getStoreConfig()` will by default, look for the path in the current store scope code, and looks for the value in the database. `Mage::getConfig()->getNode()` will by default look for the speficied path, but also can accept a scope code and a scope string `Mage::getConfig()->getNode($path, $scope, $scopeCode)`, which becomes `$scope/$scopeCode/path`. It looks for the value in configuration files.

####Describe system configuration scopes:
######How do different scopes (global, website, store) work in Magento system configuration?

In the `core_config_data` table, there are two columns that store this data: `scope` and `scope_id`.

`scope` will say either `default`, `stores`, or `websites`.

`scope_id` will be `0` for `default`, the store id for `stores`, and the website id for `websites`.

In the `system.xml` files, you can set which scopes you want the config option to appear in:

`<show_in_website>1</show_in_website>`
`<show_in_store>1</show_in_store>`
`<show_in_default>1</show_in_default>`

##Access Control Lists (ACL) and permissions in Magento
####Define/identify basic terms and elements of ACL

ACL is an admin panel permissions implementation for Magento. ACL allows the administrator to set up different roles for certain groups of users, and then to only allow users with those roles to access certain resources.

ACL options are configured in a modules `adminhtml.xml` file.

####Use ACL to:
######Set up a menu item

In a module's `adminhtml.xml` file:

```
<config>
  <acl>
    <resources>
      <admin>
        <children>
          <menu_identifier>
            <children>
              <menu_item_identifier>
                <title>Your Title</title>
                <sort_order>1</sort_order>
              </menu_item_identifier>
            </children>
          </menu_identifier>
      </admin>
    </resources>
  </acl>
</config>
```

######Create appropriate permissions for users

ACL can be used in two ways to create appropriate permissions for users: through the admin panel and through the code.

Admin panel:

1. Select the `System->Permissions->Roles` menu item and click on it.
2. Click `Add new role`
3. Create a name, and enter your password.
4. Select the role resources that the group should have access to, or set it to all.
5. Click save role.
6. Click `System->Permissions->Users`
7. Click the user you want to add the new role.
8. Click user role, and assign the user to the new group. Save the user.

Code:

Roles are set in the `admin_role` table, rules are in the `admin_rule` table, and users are in the `admin_user` table. To create a new role, and rule, and to assign it to a user, use this code:

```
$user = Mage::getModel('admin/user')->load(1);
$role = Mage::getModel('admin/roles');
$rule = Mage::getModel('admin/rules');

$role->setData([
    'role_name' => 'Your role name',
    'role_type' => Mage_Admin_Model_Acl::ROLE_TYPE_GROUP/USER
])->save();

$rule->setData([
    'resource_id' => 'admin/cms/poll',
    'role_id' => $role->getId(),
    'permission' => Mage_Admin_Model_Rules::RULE_PERMISSION_DENIED
])->save();

$user->setRoleId($role->getId());

Mage::getResourceModel('admin/user')->add($user);
```

######Check for permissions in permissions management tree structures

To check for permissions, you can override the `Mage_Adminhtml_Controller_Action::_isAllowed` method.

```
protected function _isAllowed()
{
  return Mage::getSingleton('admin/session')->isAllowed('path/to/resource');
}
```

If you need to check outside of a controller, you can use `Mage::getSingleton('admin/session')->isAllowed('path/to/resource');`.

If you need to check in your own code, without a user session, you can use:

```
$acl = Mage::getResourceModel('admin/acl')->loadAcl();
$user = Mage::getModel('admin/user')->load($userId);
$privilege =

$acl->isAllowed($user->getAclRole(), $resource);
```

######For what purpose is the `_isAllowed()` method used and which class types implement it?

The `_isAllowed()` method is used to check against the ACL for a specific path for a user. It is implemented in controllers that extend that `Mage_Adminhtml_Controller_Action`.

######What is the XML syntax for adding new menu element?

To add a new ACL menu item, use this syntax:

```
<config>
  <menu>
    <top_level_menu>
      <children>
        <menu_item>
          <title>Title</title>
          <sort_order>Sort Order</sort_order>
          <action>*/frontname/controllerName</action>
          <children>
            <!--see above-->
          </children>
        </menu_item>
      </children>
    </top_level_menu>
  </menu>
  <acl>
    <resources>
      <admin>
        <children>
          <top_level_menu>
            <children>
              <menu_item>
                <title>Menu Item</title>
                <sort_order>1</sort_order>
              </menu_item>
            </children>
          </top_level_menu>
        </children>
      </admin>
    </resources>
  </acl>
</config>
```

######What is adminhtml.xml used for? Which class parses it, and which class applies it?

Adminhtml.xml has two purposes: add menu items, and add ACL items. It is parsed by `Mage_Admin_Model_Config`, and is applied for menu items by `Mage_Adminhtml_Block_Page_Menu`, and is applied by the `Mage_Admin_Model_Session->isAllowed()` method, through the `Mage_Adminhtml_Controller_Action::_isAllowed()` method.

######Where is the code located that processes the ACL XML and where is the code that applies it?

The code that loads it is located in the `Mage_Admin_Model_Config` class, in the `__construct` and `loadAclResources` methods. The code that applies it is in the `Zend_Acl->isAllowed()` method.

######What is the relationship between Magento and Zend_Acl?

Magento's ACL system is built on top of the `Zend_Acl` module. `Mage_Admin_Model_Acl` extends `Zend_Acl`.

######How is ACL information stored in the database?

ACL information is stored in the database in the `admin_rule` table, in a structure like this:

|rule_id|role_id|resource_id|privileges|role_type|assert_id|role_type|permission|
|---|---|---|---|---|---|---|---|
|1|1|admin/cms/item|not used currently|also appears not to be used|G/U|allow/deny|


##Working with extensions in Magento
####Describe how to enable and configure extensions

Extensions can be enabled/disabled by going to the admin panel `System -> Configuration` menu, and clicking on the Advanced item under the admin menu. Enable/disable the modules you want to.

####Define Magento extensions and describe the different types of extension available (Community, Core, Commercial)

Magento extensions are bundles of source code that add or modify functionality to a default Magento installlation. A commercial extension is a paid community extension. A community extension is a free, often open-source extension, developed by community members or partners. A core extension ships with Magento and is developed by the core team.

######In which folders are Magento extensions files located?

Core extensions are located in the core code pool, community extensions in the community code pool, and commercial extensions are often located in the local code pool.

######Which files are necessary to make custom modules work?

They need the same files as any other module: a etc directory with a config.xml file, and a Namespace_Modulename.xml file in the app/etc/module/ dir.

######How can module dependencies be manipulated?

Module dependencies can be manipulated through the `depends` node in the Namespace_modulename.xml file.

```
<config>
  <modules>
    <Module_Identifier>
      <depends>
        <Dependant_Module/>
      </depends>
    </Module_Identifier>
  </modules>
</config>
```

######What is the role of the downloader?

The role of the downloader is to download and install community extensions from Magento Connet.

######How can modules be installed through Magento Connect?

To install a module through Magento Connect, obtain the key through the connect store. Select the `System -> Magento Connect -> Magento Connect Manager` menu item. Sign in with your admin credentials, and paste the extension key. Continue with the install, and then click refresh. You can also install downloaded extensions there, as well.

#7. Catalog
##Product Types
####Identify and describe standard product types (simple, configurable, bundled, etc.).

Magento ships with 6 product types:

- simple - a simple, physical, shipped product. Has a unique SKU, and inventory is handled at the simple product level. (`Mage_Catalog_Model_Product_Type_Simple`)
- bundle - gives the customer the option to bundle products together. (`Mage_Bundle_Model_Product_Type`)
- virtual - a service product. (`Mage_Catalog_Model_Product_Type_Virtual`)
- configurable - complex product. A product that the user can configure, to set details. Made up of other simple products. (`Mage_Catalog_Model_Product_Type_Configurable`)
- grouped - a group of simple products, sold as one. (`Mage_Catalog_Model_Product_Type_Grouped`)
- downloadable - sends the customer a personalized link with access to their purchase. (`Mage_Downloadable_Model_Product_Type`)

####Create custom product types from scratch or modify existing product types.

New product types can be created by extending `Mage_Catalog_Model_Product_Type_Abstract`, or the model you want to model it after. To modify an existing product type, extend it and then rewrite it.

To create a new product type, you must declare it in the module's `config.xml` file:

```
<config>
  <global>
    <catalog>
      <product>
        <type>
          <TYPENAME>
            <label>New product</label>
            <model>Namespace_Modulename/Product_Type</model>
            <!--other configuration options, such as is_qty, composite, can_use_qty_decimals-->
          </TYPENAME>
        </type>
      </product>
    </catalog>
  </global>
</config>
```

####Identify how custom product types interact with indexing, SQL, and underlying data structures

A custom product type can interact with the database and the underlying product through it's `beforeSave()` method, which is called before a product is saved, or it's `save()` method, which is called after a product is saved. It can also specify if a product is salable, through it's `getIsSalable` method. It is involved in the revieval of SKU's with the getSku method.

The type's `processBuyRequest()` method will be called when the product's method is called.

If a product type's `<price_indexer>` node is set, the specified model will be used for indexing.

######Which product types exist in Magento?

Magento ships with 6 product types by default:

`Mage_Catalog_Model_Product_Type_Grouped`, `Mage_Catalog_Model_Product_Type_Simple`, `Mage_Catalog_Model_Product_Type_Virtual`, `Mage_Catalog_Model_Product_Type_Configurable`, `Mage_Bundle_Model_Product_Type`, `Mage_Downloadable_Model_Product_Type`

######Which product types are implemented as part of the `Mage_Catalog` module, and which are not?

Grouped, simple, virtual, and configurable product types are all implemented as part of the `Mage_Catalog` module. Bundle is implemented in `Mage_Bundle`, and the downloadable product type is implemented in the downloadable module.

######What steps need to be taken in order to implement a custom product type?

See http://www.divisionlab.com/solvingmagento/creating-a-custom-product-type-in-magento/

Basically, declare the type in `config.xml`:

```
<config>
  <global>
    <catalog>
      <product>
        <type>
          <TYPENAME>
            <label>New product</label>
            <model>Namespace_Modulename/Product_Type</model>
            <!--other configuration options, such as is_qty, composite, can_use_qty_decimals, price_model-->
          </TYPENAME>
        </type>
      </product>
    </catalog>
  </global>
</config>
```

and implement your custom type, extending whatever classes and functions you find necessary to extend.

######How do the different product types handle calculation?

- Simple products use the basic price model, which just returns the price associated with the product itself (Mage_Catalog_Model_Product_Type_Price)
- Grouped products use their own price model, which returns the products final price based on the child products. (`Mage_Catalog_Model_Product_Type_Price_Grouped`)
- Virtual products also use the basic price model
- Configurable products use their own price model, which gets it's final price by adding the total price for the configrable items to the products base price and also adds to total of the associated options prices. (`Mage_Catalog_Model_Product_Type_Price_Configurable`)
- Bundle products use their own price model, `Mage_Bundle_Model_Product_Price`. This price model has the ability to calculate a minimum or maximum price for the bundle, and calculates the final price based on the total bundle items price and the options price.
- Downloadable products have their own price model, `Mage_Downloadable_Model_Product_Price`, which calculates the final price based on any additional links purchased as part of the product.

Price models are set in the `<price_model>` node when configuring a product type.

######Which indexing processes does the product type influence?

Product price, through the `<price_indexer>` node.

######Which product types implement a parent-child relationship between product entities?

Configurable, bundle, grouped. `<composite>` node is `1`, and the `<allow_product_types>` node is set to specific types of products.

######Which database tables are shared between product types, and which ones are specific to one product type?

For multipe product types: `catalog_product_relation`.
For one product type:
- (configurable) - catalog_product_super_link
- (bundle) - The `catalog_product_bundle_*` tables, and the `catalog_product_index_price_bundle_*` tables
- (downloadable) - The `downloadable_*` tables

##Price Generation
####Identify basic concepts of price generation in Magento

Prices are generated by the price model associated with a product type. By default, simple products use the `Mage_Catalog_Model_Product_Type_Price` model, but when additional functionality is needed, Magento extends this class and adds the functionality on top of it. Downloadable, configurable, bundle, and grouped products use their own price models.

The `Mage_Catalog_Model_Product_Type_Price::getBasePrice()` and `getFinalPrice()` methods are very important to price generation. The `getBasePrice` method calculates the group price, tier price, and special price for the product, and then returns the lowest value of the three. A special price is a promotional price, a tier price is discounts based on the quantity of products purchased, and group pricing is pricing for a specific group of customers. The `getFinalPrice` method applies product options to the price and returns this as the final price.

####Modify and adjust price generation for products (for example, during integration of third-party software)

The easiest way to modify and adjust the price generation mechanisms for products is through the `catalog_product_get_final_price` event. It has the product and quantity specified with it, and allows for a great (although not complete) degree of flexibility over the price generation process.

The other thing you can do is set the final price on the product. If it is set, Magento will not recalculate it

######Under what circumstances are product prices read from the index tables?

Product prices are read from the index tables when a products are in a collection.

######From which modules do classes participate in price calculation?

Price calculation is performed in the price model for a product type, so the `Mage_Catalog`, `Mage_Downloadable` and `Mage_Bundle` modules are involved in price calculations.

######Which ways exist to specify custom prices during runtime?

By observing the `catalog_product_get_final_price` event, or by setting the final price on the product directly.

######How do custom product options influence price calculation?

Options prices are added to the final price in the `getFinalPrice` method.

######How are product tier prices implemented and displayed?

Product tier prices are stored are stored in the `catalog_product_entity_tier_price` table. They are processed in the `Mage_Catalog_Model_Product_Type_Price::_applyTierPrice()` and `getTierPrice()` methods. They are displayed in the `template/catalog/product/view/tierprices.phtml` file.

######What is the priority of the different prices that can be specified for products (price, special price, group price, tier price, etc.)?

Whatever pricing is lowest is given to the customer.

##Category Structure
####Describe the Category Hierarchy Tree Structure implementation (the internal structure inside the database), including:

The Catalog Hierarchy Tree Structure implementation takes place in the `catalog_category_entity` table. Categorys have parents and children.

######The meaning of `parent_id 0`

`parent_id 0` means that the category is the root category for a store.  

######The construction of paths

Paths are constructed by concatenating the parent category entity ids.

######The attributes required to display a new category in the store

1. `url_key`
2. `name`
3. `all_children`
4. `is_anchor`
5. `position`
6. `is_active`

######How is the category hierarchy reflected in the database? Does it differ when multiple root categories are present?

The catalog hierarchy is reflected in the database with the `parent_id` and `path` fields. There can only be one root category in a magento instance (0), however, when the child category has grandparent categories, another slash is added to the path followed by the category id and then another slash.

######How is a catalog tree read from the database tables, with and without flat catalog tables enabled?

It's a bit ambiguous, but if the flat catalog tables are enabled and the app is running on the frontend, the `Mage_Catalog_Model_Resource_Category_Flat` resource model is used in the `Mage_Catalog_Model_Category` model. Otherwise, the `Mage_Catalog_Model_Resource_Category` model is used.

The catalog tree is always read with `Mage_Catalog_Model_Resource_Category_Tree`, whether or not flat catalog tables are enabled.

######How does working with categories differ if the flat catalog is enabled on a model level?

Attributes are read differently: if flat is not enabled, than the `Mage_Catalog_Model_Resource_Category_Flat` is used to read attributes. Otherwise, the `Mage_Catalog_Model_Config` is used to read attributes.

The resource model for the category will be `Mage_Catalog_Model_Resource_Category_Flat`, and the store's `catalog_category_flat_` table will be used.

You can't move categories.

######How is the category parent id path set on new categories?

The parent category ID is loaded in when the category is submitted. Then, it's path is retreived, and the id of the new category is concatenated on to it and then the path is set on the category.

######Which methods exist to read category children and how do they differ?

- `Mage_Catalog_Model_Category->getAllChildren(true)`: returns an array of all children category ID's
- `Mage_Catalog_Model_Category->getChildren()`: returns a string of all children category ID's, separated by commas
- `Mage_Catalog_Model_Category->getChildrenCategories()`: returns a collection of loaded, active child categories
- `Mage_Catalog_Model_Category->getChildrenCategoriesWithInactive()`: returns a unloaded collection of all child categories
- `Mage_Catalog_Model_Resource_Category->getAllChildren()`: returns an array of all children category ID's with the catagory ID included
- `Mage_Catalog_Model_Resource_Category->getChildren($category)`: returns an array of all children category ID's
- `Mage_Catalog_Model_Resource_Category->getChildrenCategories($category)`: returns a loaded collection of active child categories
- `Mage_Catalog_Model_Resource_Category->getChildrenCategoriesWithInactive($category)`: returns an unloaded collection of all child categories
- `Mage_Catalog_Model_Resource_Category->getChildrenIds($category)`: returns an array of child ID's, inactive and active
- `Mage_Catalog_Model_Resource_Category_Flat->getChildrenAmount($category)`: returns a count of all children categories
- `Mage_Catalog_Model_Resource_Category_Flat->getChildrenCategories($category)`: returns an array of active child categories
- `Mage_Catalog_Model_Resource_Category_Flat->getChildrenCategoriesWithInactive($category)`: returns an array of all child categories
- `Mage_Catalog_Model_Resource_Category_Flat->getChildren($category)`: returns an array of all child ids of a category
- `Mage_Catalog_Model_Resource_Category_Flat->getAllChildren($category)`: returns an array of all child ids of a category, with the parent id included

##Catalog Price Rules
####Identify how catalog price rules are implemented in Magento

Magento implements catalog price rules in the `Mage_CatalogRule` module. Catalog price rules are based on a certain date or range of time, a discount action and amount, customer groups and website ids.

######How are the catalog price rules related to the product prices?

Catalog price rules created a lowered or raised price for certain customer groups during a specific period of time. Events, specifically the `catalog_product_get_final_price` event, are used to retrieve the rules prices for a product from the `catalogrule_product_price` table.

######How are the catalog price rules stored in the database?

The catalog price rules are stored in the `catalogrule*` tables. The rule records themselves are stored in the `catalogrule` table, while the product rule information is stored in the `catalogrule_product` table. The product pricing information with the rules calculated in is stored in the `catalogrule_product_price` table. The `catalogrule_product_price` table is recalculated at least daily, by the `catalogrule_apply_all` cronjob.

##Other Skills
####Choose optimal catalog structure (EAV vs. Flat) for a given implementation

Flat will be more performant, but EAV will be better suited to a large number of differences between products. Your best bet is to add a lot of attributes to the flat table, and to mimimize the use of EAV.

####Implement, troubleshoot, and modify Magento tax rules

Magento tax rules allow you to configure taxes by combining customer and product tax classes, with zones and rates, and tax rules. They are implemented in the `Mage_Tax` module.

TALK to Joseph

####Modify, extend, and troubleshoot the Magento layered (“filter”) navigation

The layered navigation is configured in the `Mage/Catalog/Block/Layer/*` blocks. It pulls out attributes that are marked `is_filterable`, and loads their filter blocks. Layered navigation is loaded with the `catalog_category_layered` layout handle.

####Troubleshoot and customize Magento indexes

Magento indexes are used to enhance the speed of the application. To troubleshoot them, you can reindex and check the query; to customize, you need to extend `Mage_Index_Model_Indexer_Abstract` and implement the `getName`, `_processEvent`, and `_registerEvent` methods.

####Describe custom product options in Magento

Custom product options are used to specify things like add-ons to products. They can be used to add to the price, and are figured into the pricing calculations.

######When and how are the catalog flat tables updated when a product is modified?

The catalog flat table entry for the product is removed with `Mage_Catalog_Model_Resource_Product_Flat_Indexer::removeProduct`, recreated with `Mage_Catalog_Model_Resource_Product_Flat_Indexer::saveProduct` and then the related products (children, right?) are updated with `Mage_Catalog_Model_Resource_Product_Flat_Indexer::updateRelationProducts`.

######Which factors are used by the Mage_Tax module to apply the correct tax rate (or rates) to a product price?

- customer group
- location
- tax calculation rate
- product tax class
- tax calculation rule

######How can attributes with custom source models be integrated into layered navigation filtering?

Create your filter, with the custom HTML for it's rendering. Then, you need to override the `Mage_Catalog_Model_Resource_Eav_Attribute->isIndexable()` method to allow your own filter to be indexed.

######Which classes are responsible for rendering the layered navigation?

`Mage_Catalog_Block_Layer_View`, `Mage_Catalog_Block_Layer_State`

######Which indexes are used for the layered navigation?

The layered navigation reads from the `Mage_Catalog_Model_Product_Indexer_Eav` (`catalog_product_attribute`) and `Mage_Catalog_Model_Product_Indexer_Price` (`catalog_product_price`) indexes

######Which steps are needed to integrate a custom indexer into the framework offered by the Mage_Index module?

You can add the indexer into the framework by adding it to config.xml:

```
<config>
  <global>
    <index>
      <indexer>
        <indexer_identifier>
          <model>Should extend Mage_Index_Model_Indexer_Abstract</model>
        </indexer_identifier>
      </indexer>
    </index>
  </global>
</config>
```

The indexer class must implement the `Mage_Index_Model_Indexer_Abstract::getName()`, `_registerEvent()`, and `_processEvent()` methods. registerEvent specifies what to reindex; processEvent does the indexing.

######How are custom product options stored on quote and order items?

Quote: saved in `sales_flat_quote_item_option`.
Order: saved in `sales_flat_order_item` table, `product_options` field

######How can you specify custom product options on-the-fly on quote items?

By listening to the `sales_quote_item_set_product` event, and then `getQuoteItem()` on the observer. Then, call `addOption` on the quote item, passing in the option as a parameter.

######How are custom product options copied from quote to order items?

They are set as the product options on the order.

######How are custom product options processed when a product is added to the cart?

They are processed with the product type's `_prepareOptions()` method. The option type is set on the option, the associated product is set, the buy request is set, and the process mode is set. The user values are validated. The `catalog_product_type_prepare_$processMode_options` event is fired.

#8. Checkout

##Checkout components
####Describe how to modify and effectively customize the quote object, the quote item object, and the address object:
######What is the quote model used for in Magento?

The quote model (`Mage_Sales_Model_Quote`) is used to store data about an order before it is placed. It stores addresses, items to be ordered, shipping and payment methods, price totals, and customer information.

######What is the shopping cart model used for in Magento?

The shopping cart model is used to manipulate and store items in the quote.

######How does Magento store information about the shopping cart?

In the quote model/`sales_flat_quote*` tables.

######How are different product types processed when added to the cart?

In `Mage_Sales_Model_Quote::addProductAdvanced()`, the product's type model is instantiated, and it's `prepareForCartAdvanced()` method is called.

Custom product options are processed.

If the product has a super-product associated with it (grouped and configurable products), the super product's configuration is added to the buy request.

######What is the difference between shipping and billing address objects in Magento? How is each used in the quote object?

The shipping and billing address objects both are instances of `Mage_Sales_Model_Quote_Address`. They are marked as different by their type.

The shipping address is used to calculate the total for the order and available shipping methods for physical orders, and the billing address is used to calulate for non-physical orders.

######What is the difference in processing quote items for onepage and multishipping checkout in Magento?

Multishipping - there are multiple orders to each shipping address. `quote_address_item` relates multishipping items to addresses.
Onepage - there is one order created.

Ask Joseph??

###### How does Magento process additional information about products being added to the shopping cart (custom options, components of configurable products, etc.)?

`Mage_Sales_Model_Quote->_addCatalogProduct` adds the options and the parent item is set in `Mage_Sales_Model_Quote->addProductAdvanced`.

Ask Joseph??

######How do products in the shopping cart affect the checkout process?

They can affect the totals, with promotions. Also, some product types do not need shipping information, so shipping steps would not be shown.

######How can the billing and shipping addresses affect the checkout process?

They can affect the totals, because they are used to set the tax, shipping methods, etc.

######When exactly does inventory decrementing occur?

Whey an order is being submitted, before saving it to the database.

######When exactly does card authorization and capturing occur?

When the quote is submitted, the quote is converted to an order for the address. As the order is saved, the `Mage_Sales_Model_Order->place()` method is called. This method calls other methods that eventually call the `Mage_Sales_Model_Order_Payment->place()` method, which captures and authorizes the payment method.

####Explain the database schema for total models:
######What are total models responsible for in Magento?

Total models are responsible for tracking the cost of an order or quote.

######How you can customize total models?

To customize total models:

To add a new total model, you have to create a new model extending from `Mage_Sales_Model_Quote_Address_Total_Abstract`. You will likely want to override/customize the `collect` method, and perhaps the `fetch` method.

Then, to register it with Magento, you add it to config.xml:

```
<config>
  <global>
    <sales>
      <quote>
        <totals>
          <total_identifier>
            <class>Module/Total_Model</class>
            <before>comma,seperated,list of other total modules to run this before</before>
            <after>same as above</after>
          </total_identifier>
        </totals>
      </quote>
    </sales>
  </global>
</config>
```

You also can override total models, and could change the order by making your module depend on the other module and then changing the value of the node.

######How can the individual total models be identified for a given checkout process?

Each total model has a code or total identifier associated with it.

######How can the priority of total model execution be customized?

Through the comma-separated list of identifiers in the `<before>` and `<after>` nodes.

######To which objects do total models have access in Magento, and which objects do they usually manipulate?

Total models have access to the shipping address they calculate the totals for, and through that `Mage_Sales_Model_Quote_Address` object, they have access to a number of other objects: the quote, the order, the resource model, the associated items, etc.

They typically manipulate the address with information retreived from the quote.

######Which class runs total models processing?

`Mage_Sales_Model_Quote_Address` calls the methods, but `Mage_Sales_Model_Quote_Address_Total_Collector` processes the total models.

######What is the flow of total model execution?

By default, the total models are executed in this order:

MSRP (`Mage_Sales_Model_Quote_Address_Total_Msrp`)
Nominal (`Mage_Sales_Model_Quote_Address_Total_Nominal`)
Subtotal (`Mage_Sales_Model_Quote_Address_Total_Subtotal`)
Freeshipping (`Mage_SalesRule_Model_Quote_Freeshipping`)
Shipping (`Mage_Sales_Model_Quote_Address_Total_Shipping`)
Tax shipping (`Mage_Tax_Model_Sales_Total_Quote_Shipping`)
Tax subtotal (`Mage_Tax_Model_Sales_Total_Quote_Subtotal`)
Discount (`Mage_SalesRule_Model_Quote_Discount`)
Weee (`Mage_Weee_Model_Total_Quote_Weee`)
Tax (`Mage_Tax_Model_Sales_Total_Quote_Tax`)
Grand total (`Mage_Sales_Model_Quote_Address_Total_Grand`)

######At which moment(s) are total models executed?

Total models are collected by the `Mage_Sales_Model_Quote_Address_Total_Collector::collectTotals()` method, and are run at the following checkout steps (onepage): initialization, billing information, shipping information, shippping method, payment information, and place order. Also, when an item is added to the cart.

##Shopping Cart price rules
####Describe how shopping cart price rules work and how they can be customized:

They are based on a set of conditions that specify when the rule applies. They are created in the admin panel.
######Which module is responsible for shopping cart price rules?

The `Mage_SalesRule` module is responsible for shopping cart price rules.

######What is the difference between shopping cart and catalog price rules?

Catalog price rules are specific to products, while shopping cart price rules are applied to the cart as a whole and are typically based on the cart totals.

######What are the limitations of Magento shopping cart rules?

Some limitations include:

Only one voucher can be applied at once, you cannot base a rule off another rule as rules work independently of each other.

##Shipping and payment methods in Magento
####Describe the programmatic structure of shipping methods, how to customize existing methods, and how to implement new methods

Shipping methods are defined in config.xml. Their models extend `Mage_Shipping_Model_Carrier_Abstract`, and must provide an implementation of the `collectRates()` method.

```
<config>
  <default>
    <carriers>
      <carrier_id>
        <active>0||1</active>
        <sallowspecific>0</sallowspecific>
        <model></model>
        <name></name>
        <title></title>
        ...
      </carrier_id>
    </carriers>
  </default>
</config>
```

Existing methods can be customized by changing the values in config, or overriding the classes.

####Describe the shipping rates calculation process:
######How does Magento calculate shipping rates?

The address model is involved in calculating shipping rates: `Mage_Sales_Model_Quote_Address::requestShippingRates()` sends the quote address data as an instance of `Mage_Shipping_Model_Rate_Request` to `Mage_Shipping_Model_Shipping::requestRates()`, which is used collect the different carrier rates from the store's carriers.

######What is the hierarchy of shipping information in Magento?

Carriers (Fedex, UPS) provide rates (next day, flat rate, priority) to customers (you and I).

######How does the `TableRate` shipping method work?

The `Tablerate` shipping method provides a way to specify shipping costs based on a number of conditions compared with the destination: price, weight, or number of items. A csv file with conditions is imported into the database, and then the applicable shipping methods display on the frontend.

######How do US shipping methods (FedEX, UPS, USPS) work?

US shipping methods work by making a request to the carriers API to get the shipping rates. These are in a seperate module, `Mage_Usa`, and extend the `Mage_Usa_Model_Shipping_Carrier_Abstract` model. Fedex and UPS use a SOAP API to communicate, while USPS uses a specialized HTTP API format.

####Describe the programmatic structure of payment methods and how to implement new methods:
######How can payment method behavior be customized (for example: whether to charge or authorize a credit card; controlling URL redirects; etc.)?

Payment method behavior can be controlled by overriding/customizing the basic payment method models. To register a new payment method, you do so in `config.xml`:

```
<config>
  <default>
    <payment>
      <paymentcode>
        <active>1</active>
        <model>module/modelname</model>
        <order_status>pending, complete, etc.</order_status>
        <title>Payment Code Title</title>
        <allowspecific>0</allow_specific>
        <group>offline</group>
      </paymentcode>
    </payment>
  </default>
</config>
```

These variables are used to configure behavior like charging vs. authorization, etc., and are set on the payment model:

```
protected $_isGateway                   = false;
protected $_canOrder                    = false;
protected $_canAuthorize                = false;
protected $_canCapture                  = false;
protected $_canCapturePartial           = false;
protected $_canCaptureOnce              = false;
protected $_canRefund                   = false;
protected $_canRefundInvoicePartial     = false;
protected $_canVoid                     = false;
protected $_canUseInternal              = true;
protected $_canUseCheckout              = true;
protected $_canUseForMultishipping      = true;
protected $_isInitializeNeeded          = false;
protected $_canFetchTransactionInfo     = false;
protected $_canReviewPayment            = false;
protected $_canCreateBillingAgreement   = false;
protected $_canManageRecurringProfiles  = true;
```

To specify the url to redirect to, set the payment method's `order_place_redirect_url` data.

######Which class is the basic class in the payment method hierarchy?

`Mage_Payment_Model_Method_Abstract`

######How can the stored data of payment methods be customized (credit card numbers, for example)?

The stored data of payment methods is held in an instance of `Mage_Payment_Model_Info`. This method has a number of methods (`has/get/uns/setAdditionalInformation`) that allow for the manipulation of the data stored inside, in addition to the standard `get/set` on `Varien_Object`. If you want to modify the data before it is set, observe the `sales_quote_payment_import_data_before` event.

######What is the difference between payment method and payment classes (such as order_payment, quote_payment, etc.)?

TALK TO JOSEPH!?!?!

######What is the typical structure of the payment method module?

A module that provides a payment method typically has a few important components:

1. `config.xml` - registers the modules information, the payment information
2. `system.xml` - adds the system configuration for the payment method
3. The payment model - extends `Mage_Payment_Model_Method_Abstract`, implements the authorize, capture methods, etc.
4. A form block (for the checkout) - assigned in the model's `_formBlockType` property
5. An info block (to show progress in the checkout) - assigned in the model's `_infoBlockType` property

######How do payment modules support billing agreements?

Billing agreements implement the `Mage_Payment_Model_Billing_Agreement_MethodInterface`. Billing agreements are integrated out of the box, you need to set the billing agreement by calling `getPayment()->setBillingAgreementData($data)` on the order.

##Magento multishipping implementation
####Describe how to extend the Magento multishipping implementation

To extend the Magento multishipping implementation, you will need to extend the `Mage_Checkout_Model_Type_Multishipping` class. Multishipping also uses a custom controller, `Mage_Checkout_MultishippingController`.

####Identify limitations of the multishipping implementation:

Multishipping can often be harder to use and understand, require more database storage, and be a bit slower than onepage checkouts.

######How does the storage of quotes for multishipping and onepage checkouts differ?

Quotes for multishipping orders have multiple addresses. The difference lies in the storage of items: the items are stored in the database, related to addresses, in the `sales_flat_quote_address_item` table.

######Which quotes in a multishipping checkout flow will be virtual?

Quotes that only have virtual items.

######What is the difference in the multishipping processing for a quote with virtual products in it?

When an order has virtual items, the billing address will be used to process the order. Quotes with virtual items create an additional order from the billing address containing those items.

######How can different product types be split among multiple addresses when using multishipping in Magento?

Grouped products can be split among multiple addresses when using multishipping because they are added to the cart as separate items. Bundled products, however, are added as one item and, thus, cannot be split among multiple addresses.

######How many times are total models executed on a multishipping checkout in Magento?

Total models are run a lot when using multishipping in Magento: 2x when setting the billing address, 1x when setting the payment method, 1x when preparing the order based on the address, 1x when saving the quote, and (optionally) 1x when updating the shipping address. 6 times in addition to the normal total calculation per address, which can be more because of multiple shipping addresses.

######Which model is responsible for multishipping checkout in Magento?

`Mage_Checkout_Model_Type_Multishipping`

#9- Sales and Customers
##Sales
####Describe order creation in the admin

Orders are created in the magento admin by filling out a form with a lot of data pertaining to the order. The order creation model (`Mage_Adminhtml_Model_Sales_Order_Create`) is provided with the data which is then used to create the actual order.

####Describe the differences in order creation between the frontend and the admin:

The admin panel uses an order creation model to create orders, whereas the frontend just creates the order models directly. Also, unless there is an error, the quote object is stored in the seesion for the admin, not the database like the frontend. All products can be added to the order in the admin, as opposed to frontend orders which can only add products they have access to.

######Which classes are involved in order creation in the admin? What are their roles (especially the role of adminhtml classes)?

- `Mage_Adminhtml_Sales_Order_CreateController` - controller for the order creation process in the admin panel
- `Mage_Adminhtml_Model_Sales_Order_Create` - order creation model that takes the data received from the admin user and creates the appropriate order. Also manages the customer data presented to the admin user

######How does Magento calculate price when an order is created from the admin?

When products are added to the order with the special grid, the `Mage_Adminhtml_Model_Sales_Order_Create->addProduct()` method is called. It adds the product to the quote, and then sets the `_needCollect` flag on that order to be true. The totals are then collected, and then sent back to the admin.

######Which steps are necessary in order to create an order from the admin?

1. Click on the `Orders` item on the `Sales` menu.
2. In the upper-right corner, click on `Create New Order`.
3. From the next screen, choose or create a customer.
4. Then, for multi-store Magento instances, select the store.
5. Fill out the order creation information, and add products.
6. In the lower-right corner, click `Submit Order`.

######What happens when existing orders are edited in the admin?

Reordering and editing actually use the same internal method. When an existing order is edited, it is not truly edited but copied. The old order is canceled and the new order is set as a child of the old order.

######What is the difference between order status and order state?

Order status is used by and displayed to merchants to track order flow. Order state is used by Magento to keep track of where the system is in terms of order processing. One state can have multiple statuses.

####Card operations (capturing and authorization):
######Which classes and methods are responsible for credit card operations (for example authorization or capturing)?

Capturing: `Mage_Sales_Model_Order_Invoice->capture()`, `Mage_Sales_Model_Order_Payment->place()` (handles both capturing and authorization), `Mage_Payment_Model_Method_Abstract->capture()`
Authorization: `Mage_Sales_Model_Order_Payment->authorize()`, `Mage_Sales_Model_Order_Payment->place()`

######What is the difference between “pay” and “capture” operations?

Pay: sets the invoice state as paid, works for both online and offline.
Capture: when actual payment is captured online. Pay is called in capture.

######What are the roles of the `order`, `order_payment`, `invoice`, and `payment` methods in the process of charging a card?

TALK TO JOSEPH!

######What are the roles of the total models in the context of the invoice object?

They are used to display the total amounts to invoice and to capture.

######How does Magento store information about invoices?

In the `sales_flat_invoice*` tables. The `sales_flat_invoice` table is the main invoice entity, invoice items are stored in the `sales_flat_invoice_item` table, invoice comments are stored in the `sales_flat_invoice_comment` table, and ASK JOSEPH WHAT IS IN THE `sales_flat_invoice_grid` table (lighter representation to deal with data?)

####Describe the order shipment structure and process:

The order can be marked as shipped in the admin panel. Selected items or a whole order can be shipped at once. Order shipments are represented by the `Mage_Sales_Model_Order_Shipment` class.

######How shipment templates be customized?

The various shipment email templates are set up in system config, and can be changed in the Sales Emails -> Shipment config section.

######How can different items from a single order be shipped to multiple addresses? Is it possible at all?

It is possible by using the Magento Multishipping checkout implementation.

######How does Magento store shipping and tracking information?

Shipping and tracking information is stored in the DB, similar to invoicing information:

- tracking info is stored in the `sales_flat_shipment_track` table, and managed with `Mage_Sales_Model_Order_Shipment_Track`
- shipping info is stored in a number of tables: `sales_flat_shipment` is the main one, comments are in `sales_flat_shipment_comment`, and shipment items are in `sales_flat_shipment_item`.

####Describe the architecture and processing of refunds:

In Magento, refunds are known as credit memos. The `Mage_Sales_Model_Order_Creditmemo` class is the base refund class, and it interacts with the order payment model, the credit memo totals, and the payment method to issue refunds for items. Refunds can be issued for only one item, or an entire order.

######Which classes are involved, and which tables are used to store refund information?

- `Mage_Sales_Model_Order_Creditmemo->refund()`, `Mage_Sales_Model_Order_Payment->refund()`, `Mage_Payment_Model_Method_*->refund()`, `Mage_Adminhtml_Sales_Order_CreditmemoController`

- `sales_flat_creditmemo*`. `sales_flat_creditmemo` is the main entity, `sales_flat_creditmemo_comment` is for comments, `sales_flat_creditmemo_item` for individual items.

######How does Magento process taxes when refunding an order?

Magento processes taxes by recalculating the taxes based on the tax charged on the item and the tax on shipping. If the shipping costs have changed, Magento employs some very fancy math to calculate this.

######How does Magento process shipping fees when refunding an order?

The admin can specify what parts of shipping to refund. Magento ensures that not more shipping is refunded than was originally charged.

######What is the difference between online and offline refunding?

The difference between online and offline refunding is that online refunding involves credit cards or online payment methods, while offline refunding involves cash, checks, etc. Another way to put it: they both create a credit memo, but online refunding also integrates with the payment gateway to void or refund the transaction.

######What is the role of the credit memo total models in Magento?

Credit memo total models are used to calculate the credit memo totals to refund.

####Describe the implementation of the three partial order operations (partial invoice, partial shipping, and partial refund):

Partial order operations allow a shipment, invoice, or credit memo to be created for only selected items from an order. Partial operations set the order state to processing until all the items have been processed.

######How do partial order operations affect order state?

Partial operations set the order state to processing.

######How is data for partial operations stored?

Data for partial operations is stored in the DB (`sales_flat_invoice_item`, `sales_flat_shipment_item`, and `sales_flat_creditmemo_item`).

####Describe cancel operations:

Magento offers the ability to cancel most order entities. Cancellation is available through the admin panel. Orders can be mass-canceled.

######What cancel operations are available for the various order entities in Magento (order, order item, shipment, invoice, credit memo)? Do all of them support cancellation?

Orders can be canceled, order items can be canceled (or partially canceled, as in canceling a specific quantity, probably closer to an edit), shipments cannot be canceled, invoices can be canceled and voided (voiding an invoice also voids the authorization on the payment), and credit memos can be canceled.

######How are taxes processed during cancel operations?

- Orders: when an order is canceled, all tax charges are canceled.
- Order items: when an order item is canceled, the base tax amount is multiplied by the quantity canceled, and then divided by the quantity originally ordered. That tax amount is marked as canceled.
- Invoices: the order's invoiced tax totals are adjusted based on the invoice amount to cancel.
- Credit memos: the order's refunded tax totals are adjusted based on the creditmemo amount to cancel.

####Describe the architecture of the customer module

The customer module is a core Magento module that uses an EAV-based database system to store, manipulate and process customer information.

####Describe the role of customer addresses

Customer addresses can be saved to the database, and related to the customer. This allows for faster checkout times. Customer addresses are EAV, so attributes can be easily added to them. Taxes are also calculated on the shipping address.

####Describe how to add, modify, and display customer attributes:

Customer models use the EAV-based database structure.

######What is the structure of tables in which customer information is stored?

The customer information is stored in an EAV-based database schema. The `customer_entity` table is used to store the core customer data, and then the EAV tables (`customer_entity_`) are used to store related data by type (`text`, `varchar`, `datetime`, `decimal`, `int`). Customer addresses are also EAV-based, and are stored in the `customer_address_entity*` tables.

######What is the customer resource model?

`Mage_Customer_Model_Resource_Customer`

######How is customer information validated?

Customer information stored in attributes is validated with the `validate_rules` associated with an attribute. The validate rules are a serialized array of rules.

######How can customer-related email templates be manipulated?

Customer-related email templates can be manipulated in a number of different ways:

1. through the `Transactional emails` page in the admin
2. Assigning different templates in the admin system config -> customers section.
3. Setting default options for the transactional emails through system config aftering ensuring the the `Mage_Customer` model will load first.
4. New templates can be added in the `global->template->email` node in config.

######What is the difference between shipping and billing addresses for a customer?

No difference. From the customer's perspective, all addresses are added to the address book.

######How does the number of shipping and billing address entities affect the frontend interface for customers?

Saved addresses will be displayed as options in a select box during checkout; there is an addresses list in the customer account page and the default address can be set there, and other addresses managed.

######How does customer information affect prices and taxes?

Specific prices (with Catalog and shopping price rules) can be set based on the customer's group. Taxes can also be set based on the customer group.

######How can attributes be added to a customer address? How are custom address attributes you added converted to an order address?

Attributes can be added to customer addresses through setup scripts. You will use `Mage_Customer_Model_Entity_Setup` as your parent class. `customer_address` will be the entity you add it to.

######Can a customer be added to two customer groups at the same time?

No. A customer can, by default, be in only one group at a time.

#10. Advanced features
##Widgets
####Create frontend widgets and describe widget architecture:

Widgets are little reusable and easily configurable blocks for use on the Magento frontend. Widgets needs to be identified in the `system/widget.xml` file, and need an associated block to display. Widget parameters can be setup in the widget file, and are made available in the block as object data.

######What classes are typically involved in Magento widget architecture?

The widget system architecture is driven by the `Mage_Widge_Model_Widget` class. Frontend widget blocks implement the `Mage_Widget_Block_Interface` interface. Frontend widget blocks typically extend `Mage_Core_Block_Template`.

######How are admin-configurable parameters and their options specified?

These are specified in the `config/widget.xml` file. The file structure looks something like this:

```
<?xml version="1.0" ?>
<widgets>
	<Namespace_Modulename type="Namespace_Modulename/Widget_Block" module="Namespace_Modulename">
		<name>Widget name</name>
		<description type="desc">Widget description</description>
		<parameters>
      <parameter_identifier>
          <visible>1</visible>
          <label>label</label>
          <type>type, similar to system config</type>
      </parameter_identifier>
		</parameters>
	</Namespace_Modulename>
</widgets>
```

##API
####Use the Magento API to implement third party integrations

The Magento API can be used to integrate with third party applications. Magento's SOAP API allows for an easy-to-use interface with the API.

####Extend the existing Magento API to allow for deeper integrations into third party products

You can write your own API's to integrate the Magento API with custom modules, or to add new functionality. You need an API declaration file (`etc/api.xml`). Then, it's a simple process to write a model that has the necessary functionality. The model will extend from `Mage_Api_Model_Resource_Abstract`.

####Describe the different Web Service APIs available within the Magento Core

The Magento core offers a number of different APIs: SOAP v1, SOAP v2, XML-RPC, and a RESTful API.

SOAPv1 vs v2: When using Magento v1 API, we access resources via call method and provide API method name and parameters as parameters of call method. When using Magento v2 API, we access resources via real method name and we provide parameters as defined in WSDL for each specific method.

The XML_RPC API is very similar to the v1 API (they use the same interaction syntax and config files).

The RESTful API is newer, uses HTTP verbs and three-legged authentication protocol, and is a bit harder to extend.

####Describe the advantages and disadvantages of the available Web Service APIs in Magento

(Dis-)Advantages of SOAP:

well defined web-service
has pre-built standards (SOAPv1, SOAPv2, SOAPv2 WS-I)
works well in enterprise environments (due to standards)
some tools can be automated by using the WSDL
heavyweight compared to REST

(Dis-)Advantages of REST:

easier to use
more flexible
smaller learning curve
effictient/lightweight compared to SOAP
no defined web-service structure (no WSDL)

----------------Tips to remember--------------
- Event configuration: AEEOIC/M/T (Area, Events, Eventname, Observers, Id for module, class/method/type)
- Override: GTIRCc (Global, Type, ModuleId, Rewrite, Class to rewrite/to rewrite with)
- Cron: CJIscRm (Crontab, Jobs, Name of job, schedule, cron_expr, Run, model)
- Translation files: ATMIFDfile (Area, translate, modules, identifier, files, default, filename)
- Database connections: ARIC (Area, resources, identifier, connection)
- - connection types: `core_read`, `core_setup`, `core_write`, `default_read`, `default_setup`, `default_write`
