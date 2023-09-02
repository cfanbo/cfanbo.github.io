---
title: Zend_Filter_Input的基本用法
author: admin
type: post
date: 2012-04-21T09:51:30+00:00
url: /archives/12788
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
这里我们假设有一个登录入口，有三个表单元素，分别为用户名（username)，密码(password)和验证码(captcha)．

**要求如下：**

 1. 所有元素去掉两边的空格
 2. 用户名要为数字和字母
 3. 验证码为数字类型（这里为了验证为空的情况下，注释掉了这块功能．启用的话，如果输入的是非数字的话，会直接提示不能为空．）

PHP 代码如下：

```
 'StringTrim',
			'username' => 'Alnum',
			//'captcha'   => 'Digits'
	);

	$validators = array(
			'username' => array(
				'allowEmpty' => false
				),
			'password' => array(
					'allowEmpty' => false
				),
			 'captcha' => array(
					'Digits',
					'messages' => array(
							0 => 'captcha must be Digits type!'
							)
				)

	);

	$options = array(
			'notEmptyMessage' => "A non-empty value is required for field '%field%'"
	);


	$data = $_POST;
	$input = new Zend_Filter_Input($filters, $validators, $data, $options);

	if ($input->isValid()) {
		$username = $input->username;
		$password = $input->password;
		$captcha = $input->captcha;

		echo 'username:' . $username . '

';
		echo 'password:' . $password . '
';
		echo 'captcha:' . $captcha;
		echo "OK\n";
	} else {
		// error messages
		$invalidFields = $input->getInvalid();		//invalidFields
		$messages = $input->getMessages();			//messages

		//print_r($invalidFields);
		print_r($messages);
	}
?>

```