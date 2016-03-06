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

The role of the front controller (`Mage_Core_Controller_Varien_Front`) is to initialize the routers
