---
title: 7.6. 分发器 第 7 章 Zend_Controller
author: admin
type: post
date: 2008-10-29T05:40:34+00:00
excerpt: |
 |
 7.6.1. 概述
 分发是取得请求对象，提取其中的模块名，控制器名，动作名以及可选参数，然后实例化控制器并调用其中的动作的整过过程。如果其中的模块、控制器或者动作没能找到，将使用它们的默认值。Zend_Controller_Dispatcher_Standard指定每个控制器和动作的默认值为 index，模块的默认值为default，允许开发人通过setDefaultController()、setDefaultAction()和setDefaultModule()改变默认值设定。
url: /archives/481
IM_contentdowned:
 - 1
categories:
 - 程序开发

---

## 7.6. 分发器

### 7.6.1. 概述

分发是取得请求对象，提取其中的模块名，控制器名，动作名以及可选参数，然后实例化控制器并调用其中的动作的整过过程。如果其中的模块、控制器或者动作没能找到，将使用它们的默认值。 `Zend_Controller_Dispatcher_Standard` 指定每个控制器和动作的默认值为 `index`，模块的默认值为 `default`，允许开发人通过 `setDefaultController()`、 `setDefaultAction()` 和 `setDefaultModule()` 改变默认值设定。



缺省模块

当创建模块程序，你可能也需要缺省模块的命名空间（缺省配置中，缺省模块_没有_命名空间）。从 1.5.0 开始，可以在前端控制器或你的派遣器里通过指定 `prefixDefaultModule` 为 true 来实现。

```
<?php
// In your front controller:
$front->setParam('prefixDefaultModule', true);

// In your dispatcher:
$dispatcher->setParam('prefixDefaultModule', true);
?>
```

这允许你重定义一个已存在的模块为程序的缺省模块。

分发发生在前端控制器中的一个循环（loop）中。分发之前，前端控制器通过路由请求，找到用户指定的模块、控制器、动作和可选参数。然后进入分发循环，分发请求。


每次迭代（iteration）过程开始时，在请求对象中设置一个标志指示该动作已分发。如果在动作或者前/后分发（pre/postDispatch）插件重置了该标志，分发循环将继续下去并试图分发新的请求。通过改变请求中的控制器或者动作并重置已分发标志，开发人员可以定制执行一个请求链。


控制这种分发过程的动作控制器方法是 `_forward()`；在任意的 `pre/postDispatch()` 或者动作中调用该方法，并传入动作、控制器、模块、以及可选的附加参数，就可以进入新的动作。


```
<?php
public function fooAction()
{
    // forward to another action in the current controller and module:
    $this->_forward('bar', null, null, array('baz' => 'bogus'));
}

public function barAction()
{
    // forward to an action in another controller, FooController::bazAction(),
    // in the current module:
    $this->_forward('baz', 'foo', null, array('baz' => 'bogus'));
}

public function bazAction()
{
    // forward to an action in another controller in another module,
    // Foo_BarController::bazAction():
    $this->_forward('baz', 'bar', 'foo', array('baz' => 'bogus'));
}
```

### 7.6.2. 子类化分发器

`Zend_Controller_Front` 首先调用路由器找到请求中的第一个动作，然后进入分发循环，调用分发器分发动作。


分发器需要大量数据完成任务——它需要知道如何格式化控制器和动作的名称，到哪儿找到控制器类文件，模块名是否有效，以及基于其它可用信息判定请求是否能分发的API。


`Zend_Controller_Dispatcher_Interface` 定义了下列所有分发器需要实现的方法。


```
interface Zend_Controller_Dispatcher_Interface
{
    /**
     * Format a string into a controller class name.
     *
     * @param string $unformatted
     * @return string
     */
    public function formatControllerName($unformatted);

    /**
     * Format a string into an action method name.
     *
     * @param string $unformatted
     * @return string
     */
    public function formatActionName($unformatted);

    /**
     * Determine if a request is dispatchable
     *
     * @param  Zend_Controller_Request_Abstract $request
     * @return boolean
     */
    public function isDispatchable(Zend_Controller_Request_Abstract $request);

    /**
     * Set a user parameter (via front controller, or for local use)
     *
     * @param string $name
     * @param mixed $value
     * @return Zend_Controller_Dispatcher_Interface
     */
    public function setParam($name, $value);

    /**
     * Set an array of user parameters
     *
     * @param array $params
     * @return Zend_Controller_Dispatcher_Interface
     */
    public function setParams(array $params);

    /**
     * Retrieve a single user parameter
     *
     * @param string $name
     * @return mixed
     */
    public function getParam($name);

    /**
     * Retrieve all user parameters
     *
     * @return array
     */
    public function getParams();

    /**
     * Clear the user parameter stack, or a single user parameter
     *
     * @param null|string|array single key or array of keys for params to clear
     * @return Zend_Controller_Dispatcher_Interface
     */
    public function clearParams($name = null);

    /**
     * Set the response object to use, if any
     *
     * @param Zend_Controller_Response_Abstract|null $response
     * @return void
     */
    public function setResponse(Zend_Controller_Response_Abstract $response = null);

    /**
     * Retrieve the response object, if any
     *
     * @return Zend_Controller_Response_Abstract|null
     */
    public function getResponse();

    /**
     * Add a controller directory to the controller directory stack
     *
     * @param string $path
     * @param string $args
     * @return Zend_Controller_Dispatcher_Interface
     */
    public function addControllerDirectory($path, $args = null);

    /**
     * Set the directory (or directories) where controller files are stored
     *
     * @param string|array $dir
     * @return Zend_Controller_Dispatcher_Interface
     */
    public function setControllerDirectory($path);

    /**
     * Return the currently set directory(ies) for controller file lookup
     *
     * @return array
     */
    public function getControllerDirectory();

    /**
     * Dispatch a request to a (module/)controller/action.
     *
     * @param  Zend_Controller_Request_Abstract $request
     * @param  Zend_Controller_Response_Abstract $response
     * @return Zend_Controller_Request_Abstract|boolean
     */
    public function dispatch(Zend_Controller_Request_Abstract $request, Zend_Controller_Response_Abstract $response);

    /**
     * Whether or not a given module is valid
     *
     * @param string $module
     * @return boolean
     */
    public function isValidModule($module);
}
```

不过大多数情况下，只需要简单地扩展抽象类 `Zend_Controller_Dispatcher_Abstract`，其中已经定义好了上面的大部分方法。或者扩展 `Zend_Controller_Dispatcher_Standard` 类，基于标准分发器来修改功能。


需要子类化分发器的可能原因包括：期望在动作控制器中使用不同的类和方法命名模式，或者期望使用不同的分发方式，比如分发到控制器目录下的动作文件，而不是控制器类的动作方法。