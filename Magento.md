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

Magento stores templates and layout files in the `app/design` directory. There are two difference directories in the `app/design` folder: `adminhtml` and `frontend`. Inside each of those, there are directories for each theme. Inside the theme folder, there is a namespace folder, and then a `layout` and `template` folder. Layout files are stored in the `layout` folder, and template files are stored in the `template` folder.

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

######Identify the function and proper use of automatically available events, including *_load_after, etc.

These events can be used to "hook" into different actions in the system and modify results without editing core code. The *_load_after event is fired every time a model is loaded, and it is prefixed with the `_eventPrefix` property of the model. This can be set in each model. It provides a way to modify data easily.

#######Setup a cron job

Cron jobs can be set up in Magento inside a modules config.xml file.

```
<config>
  <crontab>
    <jobs>
      <job_name>
        <schedule>
          <cron_expr>Takes cron expressions (ex., 0 9 10 * * 2017 - this would run every day at 10:09am in 2017)</cron_expr>
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

######How does the framework discover active modules and their
configuration?

*See `Mage_Core_Model_Config`.

Active modules and their configuration are discovered by going into the app/etc/modules/ directory and grabbing every XML file in the directory. The files are then looped through and checked to ensure that the modules are active. The framework then loads the module config files from each modules etc/ directory (Mage_Core_Model_Config->loadModulesConfiguration()) and adds them to the mammoth config object.

######What are the common methods with which the framework accesses its
configuration values and areas?

`Mage::getStoreConfig()`, `Mage_Core_Model_Config (through Mage::app()->getConfig()->getNode() or Mage::getConfig->getNode()` and `Mage::getStoreConfigFlag()`

######How are per-store configuration values established in the XML DOM?

Per-store configuration values are read from the database (takes place in Mage_Core_Model_Config->_loadDb) or from the module config files (in the Mage_Core_Model_Config->loadModulesConfiguration function). They are stored in Mage_Core_Model_Config->_xml->stores property, and can be accessed with Mage::getConfig()->getNode() or Mage::getStoreConfig().

######By what process do the factory methods and autoloader enable class
instantiation?

The factory methods, depending on which is being called, either look in the registry for an existing object and then instantiate if not found (singleton and helper), or just instantiate a new object (model and block), by translating the name into a string like this: Namespace_Modulename_Model_Class. The framework first looks for the class by checking if it's path is registered in the config tree. The autoloader then replaces the underscores with directory separators and includes the file path. If the class is not found, an error is thrown. The factory method then instantiates the class, and returns it.

######Which class types have configured prefixes, and how does this relate to
class overrides?

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

######What is the structure of event observers, and how are properties
accessed therein?

Event observers are instances of Varien_Event_Observer. Varien_Event_Observer extends from Varien_Object, so properties in an observer can be retrieved/assigned with the “magic getters and setters.” Varien_Event_Observer also has a number of predefined getters and setters, such as get/setCallback, get/setName, and others, so they can also be used.

######What configuration parameters are available for cron jobs?

The following configuration parameters are available for cron jobs: `<schedule><cron_expr></cron_expr></schedule>` and `<run><model></model></run>`, which specifies which model/method (separated by a double colon) to run.

####Internationalization

#######Describe how to plan for internationalization of a Magento site

Internationalization is fairly easy in Magento. You will have a unique store view for each language you are using. You can then assign the different store views to sub-directories or sub-domains in Magento. Store views can be created in the admin panel in System->Manage Stores. Once you create a store view, you will need to download the language packs from Magento and configure the store view's locale information appropriately (this is in system config).

#######Describe the use of Magento translate classes and translate files

Magento translate files can be obtained from Magento Connect and are placed in the app/locale folder. They are comma-separated strings with english strings and foreign language equivalents. The Mage_Core_Model_Translate class loads the translation files from the translate/modules node in the config tree, the theme translations in the theme's locale/[languagecode]_[countrycode]/translate.csv and the translation rows from the core_translate table. The data is then pulled out in pairs and stored in Mage_Core_Model_Translate's _data property. When a helper/block's `__()` method is called, the class looks for the string in the config tree. If found, it returns the foreign-language equivalent.

######Describe the advantages and disadvantages of using subdomains and subdirectories
in internationalization

Subdomains are better because they provide a more "finished" look and national brand. They are also easier, slightly, to link content with, and can be hosted on separate servers which allows for localizing boxes by country or region.  However, subdirectories are better because they increase search engine ratings for the main site. Subdirectories increase name recognition for the main brand. All in all, both are viable options.

######Which method is used for translating strings, and on which types of
objects is it generally available?

The `__('Your string here')` method is used for translating strings. It is generally available on blocks and helpers.

######In what way does the developer mode influence how Magento handles
translations?

When the developer mode is set, all translations not related to a module are removed. Only module level translations are allowed.

######How many options exist to add a custom translation for any given
string?

There are three options:

1. The core_translate table in the database
2. The theme's translations files (app/design/[area]/[package]/[theme]/locale/[languagecode][countrycode]/translate.csv)
3. Module-level translation files (registered in the config.xml file and stored in app/locale/[languagecode]_[countrycode]/[namespace]_[module].csv)

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

######How are translation conflicts (when two modules translate the same
string) processed by Magento?

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

######How and when does Magento load the base configuration, the module
configuration, and the database configuration?

Magento loads the base configuration as the first step in the Mage_Core_Model_App->run() method by grabbing and parsing all the XML files in the app/etc folder. The module configuration is loaded next, with the _initModules method. It (and it's called functions) grab and parse all the module declaration *.xml files in app/etc/modules. It then loads the config.xml files from the etc directories. The database configuration is then loaded with the config->loadDb method. It loads the configuration values stored in the DB into the XML tree.

######How and when are the two main types of setup script executed?

Install/update scripts are run in the Mage_Core_Model_Resource_Setup::applyAllUpdates() function after the local.xml file is loaded. The function runs any necessary updates (determining any necessary upgrades by comparing the version stored in the database to the version in the configuration file).

Data scripts are run after the local configuration is loaded and the store and request are initiated, by the Mage_Core_Model_Resource_Setup::applyAllDataUpdates(). Its method of actions is fairly similar to the updates function.

###### When does Magento decide which store view to use, and when is the
current locale set?

The store view is decided in the Mage_Core_Model_App->_initCurrentStore() function. The locale is set when system configuration is loaded in.

######Which ways exist in Magento to specify the current store view?

1. Environment variables ($_SERVER['MAGE_RUN_CODE'], set in index.php)
2. Cookies (the store cookie and the Mage_Core_Model_App->_checkCookieStore() function)
3. The HTTP GET parameter `__store` (Mage_Core_Model_App->_checkGetStore())

######When are the request and response objects initialized?

The request object is initialized in the Mage_Core_Model_App->_initRequest() function, through the Mage_Core_Model_App->getRequest() function, which instantiates the object if it does not exist yet, and returns it.

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
2. 
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
- `controller_front_send_response_after` - potentially useful to use like a destructor, for clean-up related tasks. I.E., you want to increment the number of visitors on the site, etc. TALK TO JOSEPH!!
- `controller_front_init_before` - useful to modify the request/input
- `controller_front_init_routers` - useful to add new routers

####URL rewrites
######Describe URL structure/processing in Magento

Magento URL's are made up of the followup structure: `www.baseurl.mag/frontName/controller/action/params/1`. A front name is set by a module and identifies a URL to a module.

Front names are set in config.xml:

```
<config>
  <area>
    <routers>
      <moduleName>
        <use>standard (for frontend) || admin (for backend)</use>
        <args>
          <module>Namespace_Modulename</module>
          <frontName>frontName</frontName>
        </args>
      </moduleName>
    </routers>
  </area>
</config>
```

The front name is then followed by the prefix of the controller you are trying to access (i.e., IndexController = index), and then the prefix of the action you are attempting to run (e.g., viewAction() = view). This is then followed by any parameters in the URL.

######Describe the URL rewrite process

The Magento URL rewrite process allows Magento's API-style urls to become SEO-friendly URLs. The rewrite process involves these steps:

- The `$this->_getRequestRewriteController()->rewrite` method is called in Mage_Core_Controller_Varien_Front.
- The _getRequestRewriteController method instantiates the Mage_Core_Model_Url_Rewrite_Request class.
- The Mage_Core_Model_Url_Rewrite_Request->rewrite() method is called. It applies the rewrites from the database and from the config.

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

The request is matched to the rewrite in Mage_Core_Model_Url_Rewrite_Request->_rewriteDb. This method calls the Mage_Core_Model_Url_Rewrite->loadByRequestPath method which looks for the request_path to match. If they match, the system assigns the rewritten url to the _aliases property in the Zend_Controller_Request_Http object.

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

Mage_Core_Model_Config->_loadDeclaredModules

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
####Themes in Magento
######How you can use themes to customize core functionality?
Themes should not be used to truly _customize_ core functionality; if you want to really customize core functionality you should emply a custom module. Themes can be used to customize layouts and templates, however. Whether or not those are "functionality" is debatable.

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
